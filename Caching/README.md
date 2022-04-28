Thinking about how to do caching during an interview. My preference is in the following order:

## My Approaches
- Just rely on `URLCache` + `cache-control` which is the best way in my opinion. Server drives caching logic and no coding is required.
- Use cache directory to store stuff on disk + internal memory using a Singleton
- If sever doesn’t offer any `cache-control` then you can still manually store everything in the cache upon every request you make i.e. do `cache.storedCachedResponse(cachedData, for: request)`
- Use `NSCache` to just store the image. But that’s not ideal as it would get evicted upon app launch
- `NSCache` + `cachesDirectory`.

Is there an alternative to these?

## Answer
Maybe you don’t control the server implementation and there are no caching headers on the HTTP responses

Say: "My preference is in the following order:" OR ask for clarification

But relying on server has two problems: 
- Your server controls the logic. Not client. This can be good or bad. 
- There a few transient layers between code and server or often the server is faulty and the caching isn't done correctly. 
