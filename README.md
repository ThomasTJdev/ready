# Hero

`nimble install hero`

![Github Actions](https://github.com/guzba/hero/workflows/Github%20Actions/badge.svg)

[API reference](https://nimdocs.com/guzba/hero)

Hero is a Redis client that is built to work well in a multi-threaded server such as [Mummy](https://github.com/guzba/mummy).

Check out the [examples/](https://github.com/guzba/hero/tree/master/examples) folder for more sample code using Hero.

## Using Hero

Hero supports both individual Redis connections:

```nim
import hero

let redis = newRedisConn() # Defaults to localhost:6379
```

And Redis connection pools:

```nim
import hero

let redisPool = newRedisPool(3) # Defaults to localhost:6379

redisPool.withConnection conn:
    # `conn` is automatically recycled back into the pool after this block
    let reply = conn.roundtrip("PING")
```

Send any of Redis's vast set of commands:

```nim
import hero

let redis = newRedisConn()

let reply = redis.roundtrip("HSET", "mykey", "myfield", "myvalue")
```

Easily pipeline commands and transactions:

```nim
import hero

let redis = newRedisConn()

redis.send("MULTI")
redis.send("INCR", "mycount")
redis.send("SET", "mykey", "myvalue")
redis.send("EXEC")

## OR:

# redis.send([
#  ("MULTI", @[]),
#  ("INCR", @["mycount"]),
#  ("SET", @["mykey", "myvalue"]),
#  ("EXEC", @[])
#])

# Match the number of `recv` calls to the number of commands sent

let
  reply1 = redis.receive() # OK
  reply2 = redis.receive() # QUEUED
  reply3 = redis.receive() # QUEUED
  reply4 = redis.receive() # 1, OK
```

Use [PubSub](https://redis.io/docs/manual/pubsub/) to concurrently receive messages and send connection updates such as SUBSCRIBE or UNSUBSCRIBE.

```nim
import hero, std/os

let pubsub = newRedisConn()

proc recvProc() =
  try:
    while true:
      let reply = pubsub.receive()
      echo "Event: ", reply[0].to(string)
      echo "Channel: ", reply[1].to(string)
      echo "Raw: ", reply
  except RedisError as e:
    echo e.msg

var recvThread: Thread[void]
createThread(recvThread, recvProc)

pubsub.send("SUBSCRIBE", "mychannel")
```
