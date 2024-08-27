###### docs = https://realpython.com/python-redis/

install:
```
$ redisurl="http://download.redis.io/redis-stable.tar.gz"
$ curl -s -o redis-stable.tar.gz $redisurl
$ sudo su root
$ mkdir -p /usr/local/lib/
$ chmod a+w /usr/local/lib/
$ tar -C /usr/local/lib/ -xzf redis-stable.tar.gz
$ rm redis-stable.tar.gz
$ cd /usr/local/lib/redis-stable/
$ make && make install
```
```
check installation : $ redis-cli --version
```
```
$ ls -hFG /usr/local/bin/redis-* | sort
/usr/local/bin/redis-benchmark*
/usr/local/bin/redis-check-aof*
/usr/local/bin/redis-check-rdb*
/usr/local/bin/redis-cli*
/usr/local/bin/redis-sentinel@
/usr/local/bin/redis-server*
```
config redis: 
```
$ sudo su root
$ mkdir -p /etc/redis/
$ touch /etc/redis/6379.conf
```

open /etc/redis/6379.conf and add ->
```
# /etc/redis/6379.conf
```

- port              6379
- daemonize         yes
- save              60 1
- bind              127.0.0.1
- tcp-keepalive     300
- dbfilename        dump.rdb
- dir               ./
- rdbcompression    yes


->
enter redis:
```
$ redis-server /etc/redis/6379.conf
127.0.0.1:6379>
```
test : 
```
$ 127.0.0.1:6379> PING
PONG
```

to shutdown server use :
```
$ pgrep redis-server  # get pdid grom here and kill or
$ pkill redis-server
```

------------

start-server using: 
```
$ reids-server
```
use redis :
```
$ redis-cli
```
it also creates a db at 0

create new db :
```
$ redis-cli -n 5
```


---- 

#### COMMANDS:
```
SET- takes a key value pair and return OK
GET- takes a key and return value otherwise nil
MSET- takes multiple key value pair
MGET- takes multiple keys and returns their values if exists otherwise nil
EXISTS- returns 1 if the key exists otherwise 0
EXPIRE- expires after some time ex. EXPIRES key_name time_in_sec
FLUSHDB- remove all key-value pairs
QUIT - exit redis cli
KEYS * - get all keys
```
----
#### Datatypes in redis:
Type 	Commands
```
Sets :	SADD, SCARD, SDIFF, SDIFFSTORE, SINTER, SINTERSTORE, SISMEMBER,SMEMBERS, SMOVE, SPOP, SRANDMEMBER, SREM, SSCAN, SUNION, SUNIONSTORE
Hashes :	HDEL, HEXISTS, HGET, HGETALL, HINCRBY, HINCRBYFLOAT, HKEYS, HLEN, HMGET, HMSET, HSCAN, HSET, HSETNX, HSTRLEN, HVALS
Lists :	BLPOP, BRPOP, BRPOPLPUSH, LINDEX, LINSERT, LLEN, LPOP, LPUSH, LPUSHX, LRANGE, LREM, LSET, LTRIM, RPOP, RPOPLPUSH, RPUSH, RPUSHX
Strings :	APPEND, BITCOUNT, BITFIELD, BITOP, BITPOS, DECR, DECRBY, GET, GETBIT, GETRANGE, GETSET, INCR, INCRBY, INCRBYFLOAT, MGET, MSET, MSETNX, PSETEX, SET, SETBIT, SETEX, SETNX, SETRANGE, STRLEN
```


### Use in Python
```python
import redis
# protocol and db are optional
# db taken as 0 by default
redis_client = redis.Redis(host='localhost',port=6379,db=5,protocol=3,db=5)

key = "shubh"
val = "shubham"

redis.set(key,val) # put key value pair into redis
val = redis.get(key) # fetch from redis
print(val)   # shubham
```

### OPERATIONS of redis in python
#### String
```python
# Set a key-value pair
redis_client.set(key, value)

# Get the value associated with a key
value = redis_client.get(key)

# Set a key-value pair with an expiration time
redis_client.setex(key, time_in_seconds, value)

# Set multiple key-value pairs
redis_client.mset({key1: value1, key2: value2})

# Get multiple values by keys
values = redis_client.mget([key1, key2])

# Append to the value of a key
redis_client.append(key, more_value)

# Increment the integer value of a key
redis_client.incr(key, amount=1)

# Decrement the integer value of a key
redis_client.decr(key, amount=1)

# Get the length of the value of a key
length = redis_client.strlen(key)
```

#### HASH
```python
# Set a field in a hash
redis_client.hset(name, key, value)

# Get a field value from a hash
value = redis_client.hget(name, key)

# Set multiple fields in a hash
redis_client.hmset(name, {field1: value1, field2: value2})

# Get multiple fields from a hash
values = redis_client.hmget(name, [field1, field2])

# Get all fields and values in a hash
hash_data = redis_client.hgetall(name)

# Increment a field in a hash
redis_client.hincrby(name, key, amount=1)

# Get the number of fields in a hash
num_fields = redis_client.hlen(name)

# Check if a field exists in a hash
exists = redis_client.hexists(name, key)

# Delete one or more fields from a hash
redis_client.hdel(name, key)
```

#### LIST
```python
# Push a value onto the end of a list
redis_client.rpush(key, value)

# Push a value onto the start of a list
redis_client.lpush(key, value)

# Get a range of elements from a list
elements = redis_client.lrange(key, start, stop)

# Get the length of a list
length = redis_client.llen(key)

# Pop a value from the start of a list
value = redis_client.lpop(key)

# Pop a value from the end of a list
value = redis_client.rpop(key)

# Get an element at a specific index
element = redis_client.lindex(key, index)

# Set the value of an element at a specific index
redis_client.lset(key, index, value)

# Remove elements from a list
redis_client.lrem(key, count, value)

# Trim a list to a specified range
redis_client.ltrim(key, start, stop)
```

#### SET
```python
# Add one or more members to a set
redis_client.sadd(key, value1, value2)

# Remove one or more members from a set
redis_client.srem(key, value1, value2)

# Get all the members in a set
members = redis_client.smembers(key)

# Check if a member is in a set
is_member = redis_client.sismember(key, value)

# Get the number of members in a set
size = redis_client.scard(key)

# Pop a random member from a set
random_member = redis_client.spop(key)

# Get the intersection of multiple sets
intersection = redis_client.sinter(set_key1, set_key2)

# Get the union of multiple sets
union = redis_client.sunion(set_key1, set_key2)

# Get the difference between multiple sets
difference = redis_client.sdiff(set_key1, set_key2)
```

#### SORTED SET - ZSet
```python
# Add a member with a score to a sorted set
redis_client.zadd(key, {member1: score1, member2: score2})

# Remove one or more members from a sorted set
redis_client.zrem(key, member1, member2)

# Increment the score of a member in a sorted set
redis_client.zincrby(key, amount, member)

# Get the score of a member in a sorted set
score = redis_client.zscore(key, member)

# Get a range of members in a sorted set by index
members = redis_client.zrange(key, start, stop, withscores=True)

# Get a range of members in a sorted set by score
members = redis_client.zrangebyscore(key, min_score, max_score)

# Get the rank of a member in a sorted set
rank = redis_client.zrank(key, member)

# Get the number of members in a sorted set
size = redis_client.zcard(key)
```

#### KEY
```python
# Delete one or more keys
redis_client.delete(key1, key2)

# Check if a key exists
exists = redis_client.exists(key)

# Get all keys matching a pattern
keys = redis_client.keys(pattern)

# Rename a key
redis_client.rename(old_key, new_key)

# Set a key to expire after a certain number of seconds
redis_client.expire(key, time_in_seconds)

# Get the remaining time to live of a key
ttl = redis_client.ttl(key)

# Persist a key (remove its expiration time)
redis_client.persist(key)

# Get the type of a key
key_type = redis_client.type(key)
```

##### Transcations
```python
# Start a transaction
pipe = redis_client.pipeline()

# Queue commands into the transaction
pipe.set(key1, value1)
pipe.set(key2, value2)

# Execute the transaction
pipe.execute()
```

#### PUB/SUB
```python
# Subscribe to a channel
pubsub = redis_client.pubsub()
pubsub.subscribe(channel)

# Publish a message to a channel
redis_client.publish(channel, message)

# Listen for messages on subscribed channels
for message in pubsub.listen():
    print(message)
```

#### HyperLogLog
```python
# Add elements to a HyperLogLog
redis_client.pfadd(key, element1, element2)

# Count the unique elements in a HyperLogLog
count = redis_client.pfcount(key)

# Merge multiple HyperLogLogs
redis_client.pfmerge(dest_key, source_key1, source_key2)
```

#### Bitmap
```python
# Set a bit at a specific offset in a string value
redis_client.setbit(key, offset, value)

# Get the bit value at a specific offset
bit_value = redis_client.getbit(key, offset)

# Count the number of set bits (1s) in a string
bit_count = redis_client.bitcount(key)
```

#### Geospatial
```python
# Add a member with a longitude and latitude to a geospatial index
redis_client.geoadd(key, {member1: (longitude1, latitude1)})

# Get the longitude and latitude of a member
location = redis_client.geopos(key, member)

# Get the distance between two members
distance = redis_client.geodist(key, member1, member2)

# Get members within a radius of a point
members = redis_client.georadius(key, longitude, latitude, radius, unit='m')

# Get members within a radius of another member
members = redis_client.georadiusbymember(key, member, radius, unit='m')
```
#### Stream
```python
# Add an entry to a stream
redis_client.xadd(stream, {field1: value1, field2: value2})

# Get entries from a stream
entries = redis_client.xrange(stream, min='-', max='+')

# Read entries from multiple streams
entries = redis_client.xread({stream: '0'}, count=10, block=1000)

# Acknowledge a message in a consumer group
redis_client.xack(stream, group, message_id)

# Create a consumer group
redis_client.xgroup_create(stream, group, id='0')

# Read from a consumer group
entries = redis_client.xreadgroup(group, consumer, {stream: '>'}, count=10, block=1000)
```




### Command exmaples:
```
127.0.0.1:6379> SET Bahamas Nassau
OK
127.0.0.1:6379> SET Croatia Zagreb
OK
127.0.0.1:6379> GET Croatia
"Zagreb"
127.0.0.1:6379> GET Japan
(nil)
```

or in python like
```python
>>> capitals = {}
>>> capitals["Bahamas"] = "Nassau"
>>> capitals["Croatia"] = "Zagreb"
>>> capitals.get("Croatia")
'Zagreb'
>>> capitals.get("Japan")  # None
```

#### MSET and MGET
set and get multiple values
```
127.0.0.1:6379> MSET Lebanon Beirut Norway Oslo France Paris
OK
127.0.0.1:6379> MGET Lebanon Norway Bahamas
1) "Beirut"
2) "Oslo"
3) "Nassau"
```

or in python
```python
>>> capitals.update({
...     "Lebanon": "Beirut",
...     "Norway": "Oslo",
...     "France": "Paris",
... })
>>> [capitals.get(k) for k in ("Lebanon", "Norway", "Bahamas")]
['Beirut', 'Oslo', 'Nassau']
```




### REDIS PUB/SUB

#### commands:
```
SUBSCRIBE- used to subscribe the client to the specified channels
PUBLISH- used to post a message to given channel
UNSUBSCRIBE- it is used to unsubscribe to given channel
PSUBSCRIBE- subscribe client to given pattern
PUNSUBSCRIBE- unsubscribe client from given pattern
PUBUB CHANNELS- list currently active channels
PUBSUB NUMSUB- returns no of subscribers (exclusive of clients subscribed to patterns) for specified channels

PING
RESET
QUIT
```

In Pub Sub there are publisher (redis_pub.py) and Subscriber (redis_sub.py):
```redis_pub.py```
```python
import redis
import sys

if len(sys.argv) == 2:
    program, action = sys.argv
    channel = "shubh"
    client = redis.Redis(host='localhost', port=6379)
    client.publish(channel, action)
else:
    print("You must give action(what you want to publish)")
```
run in console with arguments: 
```
$ python3 redis_pub.py "my message"
```

```redis_sub.py```
```python
import redis
import sys

channel = "shubh"

client = redis.Redis(host="localhost", port=6379)
c = client.pubsub()
c.subscribe("shubh")

print("Subscribed to channel:", channel)
while True:
    try:
        msg = c.get_message()
        if msg and msg['data'] != 1:
            msg = msg["data"].decode('utf-8')
            print(msg)

    except KeyboardInterrupt:
        print("Intruded")
        sys.exit(0)
```




