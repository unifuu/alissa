---
title: "About Redis"
date: "2025-12-18"
summary: "📝 Quick notes on Redis..."
tags: ["backend", "database", "redis"]
---

## RTT

- RTT: Round-Trip Time
- The total time it takes for a request to go from the client to the server and for the response to come back.
- `Client → Server → Client` full round trip = 1 RTT

## Pipeline

- Pipeline is mainly used to improve speed and throughput.
- The operations put into a pipeline usually do not require strict consistency with each other.
- Although Redis could execute them inside a transaction, doing so would introduce blocking and reduce performance, which goes against the original goal of using Pipeline.
- It’s that the operations are independent or eventually consistent

## Distributed Lock

- A correct Redis distributed lock requires SET NX with expiration and a Lua-based unlock to guarantee ownership.

``` go
type RedisLock struct {
	client *redis.Client
	key    string
	value  string
	ttl    time.Duration
}

func Acquire(
	ctx context.Context,
	client *redis.Client,
	key string,
	ttl time.Duration,
) (*RedisLock, bool, error) {

	value := uuid.NewString()

	ok, err := client.SetNX(ctx, key, value, ttl).Result()
	if err != nil {
		return nil, false, err
	}

	if !ok {
		return nil, false, nil
	}

	return &RedisLock{
		client: client,
		key:    key,
		value:  value,
		ttl:    ttl,
	}, true, nil
}

// Lua Script
const unlockScript = `
if redis.call("GET", KEYS[1]) == ARGV[1] then
	return redis.call("DEL", KEYS[1])
else
	return 0
end`

func (l *RedisLock) Release(ctx context.Context) (bool, error) {
	res, err := l.client.Eval(
		ctx,
		unlockScript,
		[]string{l.key},
		l.value,
	).Int()

	if err != nil {
		return false, err
	}

	return res == 1, nil
}

func GrantReward(ctx context.Context, rdb *redis.Client, playerID, missionID string) error {
	lockKey := "lock:reward:" + playerID + ":" + missionID

	lock, ok, err := Acquire(ctx, rdb, lockKey, 10*time.Second)
	if err != nil {
		return err
	}

	if !ok {
		return nil
	}

	defer lock.Release(ctx)

	err = rdb.IncrBy(ctx, "player:"+playerID+":gold", 100).Err()
	if err != nil {
		return err
	}

	return rdb.Set(ctx,
		"reward:sent:"+playerID+":"+missionID,
		1,
		0,
	).Err()
}
```