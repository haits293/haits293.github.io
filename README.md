# Haizekk's Page

### Crystal
Today, I have a task with Crystal API combined with Redis. Actually, before this API used MySQL to store data. But right now, when the app's got a really big amount of users, it's getting worse when getting a list of ignored users (a list of users that you're ignored). Because now the app has over 300.000 users and this really have to change. So I have to change it to use data from Redis server instead of from MySQL Database. I reached crystal-redis, which is a redis client developed for Crystal.
The setup is easy, really.
```crystal
redis = Redis.new(url: "redis://[ip]:[port]")
```
But I really got struggled when I tried to get a list of ignored users of current user from Redis
I used
```crystal
def ids_by_redis(uid)
    ids = @redis.zrange('namespace:ignored_user_#{uid}', -5000, -1)
    ids
end
```
But the type of the return value is Array(Redis::RedisValue), and literally, I saw it just an Array of String like this
```bash
=> ["1", "40", "100"....]
```
And along with this I used some other functions to retrieve data from MySQL, the type is all `Int32`
So I found my mission to convert this Array(Redis::RedisValue) to Array(Int32)
I go to search from all over the Internet, read some crystal-redis issues. I found someone has the same problems with me but nowhere has a solution. So I got back to Crystal-lang official site. I tried to use map to loop over the Array and convert each one to Int32:
```crystal
ids = @redis.zrange('namespace:ignored_user_#{uid}').map do |id|
    id.to_i
end
```
I wrote the code above because I'm a Ruby on Rails Developer, and Crystal has a lot of common with Ruby, so I came with that. But it didn't have a method `to_i`. Or maybe it has, but it's used by another way.

I found in the documentation of crystal, they have `as` to convert type. So I tried again
```crystal
ids = redis.zrange('namespace:ignored_user_#{uid}').map do |id|
    id.as(Int32)
end
```

This time I got some error related to Pointer, I kept digging in that page about `as`, and I found my solution. That's maybe not the best solution, but it worked well. And I wrote this to help me remember, to strengthen my knowledge about this new, exciting language.

```crystal
def ids_by_redis(uid)
    ids = @redis.zrange("namespace:ignored_user_#{uid}", -5000, -1).to_a
    ptr = Pointer(Void).new(ids.object_id)
    result = ptr.as(Array(Int32))
    result
end
```
with this I converted the Array of Redis::RedisValue to Array(Int32). What a wonderful day. Hope it could help someone who is digging in this new, powerful language. It doesn't have a big community recently, but I believe it will become popular soon.
