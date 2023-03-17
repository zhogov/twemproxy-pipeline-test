twemproxy pipelining testbench

```
docker compose up
```

Starts twemproxy pointing to 3 redis servers, 3rd of which has 3 seconds of latency.

These keys point to servers 1, 2, 3 respectively
```
~/ redis-cli -p 6380 GET "from redis 1"
(nil)
~/ redis-cli -p 6380 GET "from redis       2"
(nil)
~/ redis-cli -p 6380 GET "from redis    3"
(error) ERR Operation timed out
```

Example of MGET timing out if even single key times out:
```
~/ redis-cli -p 6380 MGET "from redis 1" "from redis 2" "from redis 3 "
(error) ERR Operation timed out
```
