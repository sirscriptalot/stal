Stal
====

Set algebra solver for Redis.

Description
-----------

`Stal` receives an array with an s-expression composed of commands
and key names and resolves the set operations in [Redis][redis].

Community
---------

Meet us on IRC: [#lesscode](irc://chat.freenode.net/#lesscode) on
[freenode.net](http://freenode.net/).

Getting started
---------------

Install [Redis][redis]. On most platforms it's as easy as grabbing
the sources, running make and then putting the `redis-server` binary
in the PATH.

Once you have it installed, you can execute `redis-server` and it
will run on `localhost:6379` by default. Check the `redis.conf`
file that comes with the sources if you want to change some settings.

Usage
-----

`Stal` is a Lua script for Redis that receives a JSON formatted
s-expression and resolves the set algebra operations.

```
# Populate some sets
$ redis-cli SADD A 1 2 3
(integer) 3
$ redis-cli SADD B 2 3 4
(integer) 3
$ redis-cli SADD C 3 4 5
(integer) 3

# Calculate the intersection of A and B
$ redis-cli --eval stal.lua , '["SINTER", "A", "B"]'
1) "2"
2) "3"
```

More complex expressions are possible:

```
$ redis-cli --eval stal.lua , '["SDIFF", ["SUNION", "A", "C"], "B"]'
1) "1"
2) "5"
```

`Stal` translates the internal calls to  `SDIFF`, `SINTER`, `SUNION`,
`ZINTER` and `ZUNION` into `SDIFFSTORE`, `SINTERSTORE`, `SUNIONSTORE`,
`ZINTERSTORE` and `ZUNIONSTORE` respectively in order to perform
the underlying operations, and it takes care of generating and
deleting any temporary keys. Furthermore, the temporary keys are
not replicated to the slaves and are not appended to AOF.

All the nested commands in the s-expression will have a temporary
key passed as the first argument. That means only commands that can
make sense of that feature can be used in nested expressions.

```
$ redis-cli --eval stal.lua , '["SINTER", ["SADD", "4", "5" "6"], "C"]'
1) "4"
2) "5"
```

The outermost command can be practically anything, for example:

```
$ redis-cli --eval stal.lua , '["DBSIZE"]'
(integer) 3
$ redis-cli --eval stal.lua , '["SCARD", ["SUNION", "A", "B", "C"]]'
(integer) 5
```

Installation
------------

Copy the script and use it directly as shown in the examples. You
can also use a [wrapper](#wrappers) in your preferred programming
language.

Wrappers
--------

The following wrappers are available:

- [Stal for Ruby](https://github.com/soveran/stal-ruby)
- [Stal for Crystal](https://github.com/soveran/stal-crystal)

Development
-----------

The tests are written in Lua and can be run as follows:

```
 $ lua stal_test.lua
```

Alternatively you can run `make`. If there's no output, it means
all the assertions passed.

[redis]: http://redis.io
