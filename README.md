# buffering-cache

[![CircleCI](https://circleci.com/gh/xtrctio/buffering-cache.svg?style=svg)](https://circleci.com/gh/xtrctio/buffering-cache)

Cache a little cold? Cache misses slowing you down?

`buffering-cache` keeps your cache warm asynchronously. Minimizing cache misses and keeping the entries from becoming stale.

Currently works with Redis only. Welcoming PRs for memcached or any other in-memory database.

Also offers a multi-level cache option. In addition to Redis, you can use local memory cache courtesy of [lru-cache](https://github.com/isaacs/node-lru-cache) before failing over to a remote Redis instance.

## Installation:
```bash
npm install --save @xtrctio/buffering-cache
```

## Simple Example:
```javascript
const BufferingCache = require('@xtrctio/buffering-cache');
const rp = require('request-promise');

const bufferingCache = new BufferingCache({
  host: 'localhost',
  port: 6379,
  ttlMsec: 5000
});

const rawFunction = () => rp('http://www.google.com');

const bufferedAndCachedFunction = bufferingCache.wrapFunction(rawFunction);

bufferedAndCachedFunction()
.then((response) => {
  // No cache yet
  // 'response' returned after round-trip to google.com.
})
.delay(3000) // Wait a bit
.then(() => {
  return bufferedAndCachedFunction();
})
.then(() => {
  // Returns results from cache
  // By default bufferTtlMsec = ttlMsec / 2.
  // So at 3000 ms the cache is refreshed after the result is returned
  return bufferedAndCachedFunction();
})
.delay(3000)
.then(() => {
  // Returns results from cache again.
  // This time returns the result from the previous refresh.
  // Refreshes the cache again
  return bufferedAndCachedFunction();
})
```

## Configuration
```javascript
const configuration = {
  // Required:
  host: 'localhost',  // Redis host. Must be a string
  port: 6379,         // Redis port. Must be a number 0-65535
  ttlMsec: 500,       // Redis ttl. Must be a number gte 0

  // Optional:
  db: 0,              // Redis db. Defaults to 0. Must be a number 0-255
  keyPrefix: '',      // Redis key prefix. Defaults to ''. Must be a string
  bufferTtlMsec: 500, // Buffer TTL in msec. Defaults to ttlMsec / 2. Must be gt 0 and lt ttlMsec
  localCacheSize: 0,  // Local LRU cache size. Defaults to 0 (disabled). Must be gt 0.
  localTtlMsec: 500   // Local LRU cache TTL. Defaults to 500 ms, or bufferTtlMsec, whichever is less
};

const bufferingCache = new BufferingCache(configuration);

