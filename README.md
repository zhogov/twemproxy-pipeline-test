# twemproxy timeout testbench

```
docker compose up
```

Starts twemproxy pointing to 3 redis servers, **2nd** of which has 3 seconds of latency. 

twemproxy's read/connection timeout is only 2 seconds, so requests to **2nd** server are destimed to time out.

## Testing commands via direct connection to Redis

The following keys point to servers 1, 2, 3 respectively (based on default hashing function and key distribution)
Let's set values via direct conection to Redis servers. Note significant time that second command takes
```
$ echo 'SET "from_redis______1" "value_from_redis_1"' | nc localhost 44001
+OK
$ echo 'SET "from_redis___2" "value_from_redis_2"' | nc localhost 44002
+OK
$ echo 'SET "from_redis_3" "value_from_redis_3"' | nc localhost 44003
+OK
```

## Testing commands via twemproxy

### GET
Second server times out, as expected.
```
$ redis-cli -p 6380 GET "from_redis______1"
"value_from_redis_1"
$ redis-cli -p 6380 GET "from_redis___2"
(error) ERR Operation timed out
$ redis-cli -p 6380 GET "from_redis_3"
"value_from_redis_3"
```

### MGET
Example of MGET timing out if even a single key times out:
```
$ redis-cli -p 6380 MGET "from_redis______1" "from_redis___2" "from_redis_3"
(error) ERR Operation timed out
```

### Pipeline

[Pipelining in twemproxy is only possible using Redis serialization protocol.](https://github.com/twitter/twemproxy/issues/259)
To pipeline three commands:
```
GET "from_redis______1"
GET "from_redis___2"
GET "from_redis_3"
```
...we need to send to redis the following string:
```
$ printf '*2\r\n$3\r\nGET\r\n$17\r\nfrom_redis______1\r\n*2\r\n$3\r\nGET\r\n$14\r\nfrom_redis___2\r\n*2\r\n$3\r\nGET\r\n$12\r\nfrom_redis_3\r\n' > /tmp/redis.txt
```
That reads like this:
```
$ cat /tmp/redis.txt
*2
$3
GET
$17
from_redis______1
*2
$3
GET
$14
from_redis___2
*2
$3
GET
$12
from_redis_3
```
Now let's run the pipeline and observe that **twemproxy will wait for its timeout to expire before returning a response from 2nd server**
```
$ (cat /tmp/redis.txt; sleep 4) | nc localhost 6380
$18
value_from_redis_1
-ERR Operation timed out
$18
value_from_redis_3
```
