# twemproxy timeout testbench

```
docker compose up
```


Starts twemproxy pointing to 3 redis servers, 3rd of which has 3 seconds of latency. 

twemproxy's read/connection timeout is only 2 seconds, so requests to 3rd server are destimed to time out.

### Testing commands

These keys point to servers 1, 2, 3 respectively (based on default hashing function and key distribution)
```
~/ redis-cli -p 6380 GET "from_redis______1"
(nil)
~/ redis-cli -p 6380 GET "from_redis___2"
(nil)
~/ redis-cli -p 6380 GET "from_redis_3"
(error) ERR Operation timed out
```

Example of MGET timing out if even a single key times out:
```
~/ redis-cli -p 6380 MGET "from_redis______1" "from_redis___2" "from_redis_3"
(error) ERR Operation timed out
```
