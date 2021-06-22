# Changelog

## 6/17/2021

Changes this week:
- Updated V8 from 9.1 to 9.2.
- Wrangler tail now works on Durable Objects. Note that logs from long-lived WebSockets will not be visible until the WebSocket is closed.

## 6/4/2021

Changes this week:
- WebCrypto: We now support the “raw” import/export format for ECDSA/ECDH public keys.
- request.cf is no longer missing when writing workers using modules syntax.

## 5/14/2021

Changes this week:
- Improve error messages coming from the WebCrypto API.
- Updated V8: 9.0 → 9.1

Changes in an earlier release:
- WebCrypto: Implement JWK export for RSA, ECDSA, & ECDH.
- WebCrypto: Add support for RSA-OAEP
- WebCrypto: HKDF implemented.
- Fix recently-introduced backwards clock jumps in Durable Objects.
- WebCrypto.generateKey(), when asked to generate a key pair with algorithm RSA-PSS, would instead return a key pair using algorithm RSASSA-PKCS1-v1\_5. Although the key structure is the same, the signature algorithms differ, and therefore signatures generated using the key would not be accepted by a correct implementation of RSA-PSS, and vice versa. Since this would be a pretty obvious problem, but no one ever reported it to us, we guess that currently no one is using this functionality on Workers.

## 4/29/2021

Changes this week:
- WebCrypto: Implemented wrapKey() / unwrapKey() for AES algorithms.
- The arguments to WebSocket.close() are now optional, as the standard says they should be.

## 4/23/2021

Changes this week:
- In the WebCrypto API, encrypt and decrypt operations are now supported for the “AES-CTR” encryption algorithm.
- For Durable Objects, CPU time limits are now enforced on the object level rather than the request level. Each time a new request arrives, the time limit is “topped up” to 500ms. After the (free) beta period ends and Durable Objects becomes generally available, we’ll increase this to 30 seconds.
- When a Durable Object exceeds its CPU time limit, the entire object will be discarded and recreated. Previously, we allowed subrequest requests to continue using the same object, but this was dangerous because hitting the CPU time limit can leave the object in an inconsistent state.
- Long running Durable Objects are given more subrequest quota as additional WebSocket messages are sent to them, to avoid the problem of a long-running Object being unable to make any more subrequests after it’s been held open by a particular WebSocket for a while.
- When a Durable Object’s code is updated, or when its isolate is reset due to exceeding the memory limit, all stubs pointing to the object will become invalidated and have to be recreated. This is consistent with what happens when the CPU time is exceeded, or when stubs become disconnected due to random network errors. This behavior is useful, as apps can now assume that two messages sent to the same stub will be delivered to exactly the same live instance (if they are delivered at all). Apps which don’t care about this property should recreate their stubs for every request; there is no performance penalty from doing so.
- When a Durable Object’s isolate exceeds its memory limit, an exception with an explanatory message will now be thrown to the caller, instead of “internal error”.
- When a Durable Object exceeds its CPU time limit, an exception with an explanatory message will now be thrown to the caller, instead of “internal error”.
- `wrangler tail` now reports CPU-time-exceeded exceptions with an explanatory message instead of “internal error”.

## 4/19/2021

Changes since the last time we posted (which was apparently 3/26):
- Cron Triggers now have a 15 minute wall time limit, in addition to the existing CPU time limit. (Previously, there was no limit, so a cron trigger that spent all its time waiting for I/O could hang forever.)
- Our WebCrypto implementation now supports importing and exporting HMAC and AES keys in JWK format.
- Our WebCrypto implementation now supports AES key generation for CTR, CBC, and KW modes. AES-CTR encrypt/decrypt and AES-KW key wrapping/unwrapping support will land in a later release.
- Fixed bug where crypto.subtle.encrypt() on zero-length inputs would sometimes throw an exception.
- Errors on script upload will now be properly reported for module-based scripts, instead of appearing as a ReferenceError.
- WebCrypto: Key derivation for ECDH.
- WebCrypto: Support ECDH key generation & import.
- WebCrypto: Support ECDSA key generation.
- Fixed bug where crypto.subtle.encrypt() on zero-length inputs would sometimes throw an exception.
- Improved exception messages thrown by the WebCrypto API somewhat.
- waitUntil is now supported for module workers. An additional argument called ‘ctx’ is passed after ‘env’, and waitUntil is a method on ‘ctx’.
- passThroughOnException is now available under the ctx argument to module handlers
- Reliability improvements for Durable Objects
- Reliability improvements for Durable Objects persistent storage API
- ScheduledEvent.cron is now set to the original cron string that the event was scheduled for.

## 3/26/2021

Changes this week:
- Existing WebSocket connections to Durable Objects will now be forcibly disconnected on code updates, in order to force clients to connect to the instance running the new code.

## 3/11/2021

New this week:
- When the Workers Runtime itself reloads due to us deploying a new version or config change, we now preload high-traffic workers in the new instance of the runtime before traffic cuts over. This ensures that users do not observe cold starts for these workers due to the upgrade, and also fixes a low rate of spurious 503 errors that we had previously been seeing due to overload during such reloads.

(It looks like no release notes were posted the last few weeks, but there were no new user-visible changes to report.)
## 2/11/2021

Changes this week:
- In the preview mode of the dashboard, a worker that fails during startup will now return a 500 response, rather than getting the default passthrough behavior, which was making it harder to notice when a worker was failing.
- A Durable Object’s ID is now provided to it in its constructor. It can be accessed off of the `state` provided as the constructor’s first argument, as in `state.id`.

## 2/5/2021

New this week:
- V8 has been updated from 8.8 to 8.9.
- During a `fetch()`, if the destination server commits certain HTTP protocol errors, such as returning invalid (unparseable) headers, we now throw an exception whose description explains the problem, rather than an “internal error”.

New last week (forgot to post):
- Added support for `waitUntil()` in Durable Objects. It is a method on the state object passed to the Durable Object class’s constructor.

## 1/22/2021

New in the past week:
- Fixed a bug which caused scripts with WebAssembly modules to hang when using devtools in the preview service.

## 1/14/2021

Changes this week:
- Implemented File and Blob APIs, which can be used when constructing FormData in outgoing requests. Unfortunately, FormData from incoming requests at this time will still use strings even when file metadata was present, in order to avoid breaking existing deployed workers. We will find a way to fix that in the future.

## 1/7/2021

Changes this week:
- No user-visible changes.

Changes in the prior release:
- Fixed delivery of WebSocket “error” events.
- Fixed a rare bug where a WritableStream could be garbage collected while it still had writes queued, causing those writes to be lost.

## 12/10/2020

Changes this week:
- Major V8 update: 8.7.220.29 -> 8.8.278.8

## 9/19/2019

Changes this week:
- Unannounced new feature. (Stay tuned.)
- Enforced new limit on concurrent subrequests (see below).
- Stability improvements.

### Concurrent Subrequest Limit

As of this release, we impose a limit on the number of outgoing HTTP requests that a worker can make simultaneously. **For each incoming request**, a worker can make up to 6 concurrent outgoing fetch() requests.

If a worker’s request handler attempts to call fetch() more than six times (on behalf of a single incoming request) without waiting for previous fetches to complete, then fetches after the sixth will be delayed until previous fetches have finished. A worker is still allowed to make up to 50 total subrequests per incoming request, as before; the new limit is only on how many can execute simultaneously.

#### Automatic deadlock avoidance

Our implementation automatically detects if delaying a fetch would cause the worker to deadlock, and prevents the deadlock by cancelling the least-recently-used request. For example, imagine a worker that starts 10 requests and waits to receive all the responses *without reading the response bodies*. A fetch is not considered complete until the response body is fully-consumed (e.g. by calling `response.text()` or `response.json()`, or by reading from `response.body`). Therefore, in this scenario, the first six requests will run and their response objects would be returned, but the remaining four requests would not start until the earlier responses are consumed. If the worker fails to actually read the earlier response bodies and is still waiting for the last four requests, then the Workers Runtime will automatically cancel the first four requests so that the remaining ones can complete. If the worker later goes back and tries to read the response bodies, exceptions will be thrown.

#### Most Workers are Not Affected

The vast, vast majority of workers make fewer than six outgoing requests per incoming request. Such workers are totally unaffected by this change.

Of workers that do make more than six outgoing requests concurrently for a single incoming request, the vast majority either read the response bodies immediately upon each response returning, or never read the response bodies at all. In either case, these workers will still work fine – although they may be a little slower due to outgoing requests after the sixth being delayed.

A very very small number of deployed workers (about 20 total) make more than 6 requests concurrently, wait for all responses to return, and then go back to read the response bodies later. For all known workers that do this, we have temporarily grandfathered your zone into the old behavior, so that your workers will continue to operate. However, we will be communicating with customers one-by-one to request that you update your code to proactively read request bodies, so that it works correctly under the new limit.

#### Why did we do this?

Cloudflare communicates with origin servers using HTTP/1.1, not HTTP/2. Under HTTP/1.1, each concurrent request requires a separate connection. So, workers that make many requests concurrently could force the creation of an excessive number of connections to origin servers. In some cases, this caused resource exhaustion problems either at the origin server or within our own stack.

On investigating the use cases for such workers, every case we looked at turned out to be a mistake or otherwise unnecessary. Often, developers were making requests and receiving responses, but they only cared about the response status and headers but not the body. So, they threw away the response objects without reading the body, essentially leaking connections. In some other cases, developers had simply accidentally written code that made excessive requests in a loop for no good reason at all. Both of these cases should now cause no problems under the new behavior.

We chose the limit of 6 concurrent connections based on the fact that Chrome enforces the same limit on web sites in the browser.

## 12/4/2020

Changes this week:
- Durable Objects storage API now supports listing keys by prefix.
- Improved error message when a single request performs more than 1000 KV operations to make clear that a per-request limit was reached, not a global rate limit.
- `wrangler dev` previews should now honor non-default resource limits, e.g. longer CPU limits for those in the Workers Unbound beta.
- Fixed off-by-one line numbers in worker exceptions.
- Exceptions thrown in a Durable Object’s `fetch()` method are now tunneled to its caller.
- Fixed a bug where a large Durable Object response body could cause the Durable Object to become unresponsive.

## 11/13/2020

Changes over the past week:
- ReadableStream.cancel() and ReadableStream.getReader().cancel() now take an optional, instead of a mandatory, argument, to conform with the Streams spec.
- Fixed an error that occurred when a WASM module declared that it wanted to grow larger than 128MB. Instead, the actual memory usage of the module is monitored and an error is thrown if it exceeds 128MB used.

## 11/5/2020

Changes this week:
- Major V8 update: 8.6 -> 8.7
- Limit the maximum number of Durable Objects keys that can be changed in a single transaction to 128.

## 10/5/2020

We had our usual weekly release last week, but:
- No user-visible changes.

## 9/24/2020

Changes this week:
- Internal changes to support upcoming features.

Also, a change from the 2020-09-08 release that it seems we forgot to post:
- V8 major update: 8.5 -> 8.6

## 8/3/2020

Changes last week:
- Fixed a regression which could cause HTMLRewriter.transform() to throw spurious “The parser has stopped.” errors.
- Upgraded V8 from 8.4 to 8.5.

## 7/9/2020

Changes this week:
- Fixed a regression in HTMLRewriter: [https://github.com/cloudflare/lol-html/issues/50](<https://github.com/cloudflare/lol-html/issues/50>)
- Common HTTP method names passed to `fetch()` or `new Request()` are now case-insensitive as required by the Fetch API spec.

Changes last week (… forgot to post):
- setTimeout/setInterval can now take additional arguments which will be passed on to the callback, as required by the spec. (Few people use this feature today because it’s usually much easier to use lambda captures.)

Changes the week before last (… also… forgot to post… we really need to code up a bot for this):
- The HTMLRewriter now supports the `:nth-child` , `:first-child` , `:nth-of-type` , and `:first-of-type` selectors.

## 5/15/2020

Changes this week:
- Implemented API for yet-to-be-announced new feature.

## 4/20/2020

Looks like we forgot to post release notes for a couple weeks. Releases still are happening weekly as always, but the “post to the community” step is insufficiently automated…
4/2 release:
- Fixed a source of long garbage collection paused in memory limit enforcement.

4/9 release:
- No publicly-visible changes.

4/16 release:
- In preview, we now log a warning when attempting to construct a `Request` or `Response` whose body is of type `FormData` but with the `Content-Type` header overridden. Such bodies would not be parseable by the receiver.

## 3/26/2020

New this week:
- Certain “internal errors” that could be thrown when using the Cache API are now reported with human-friendly error messages. For example, `caches.default.match("not a URL")` now throws a TypeError.

## 2/28/2020

New from the past two weeks:
- Fixed a bug in the preview service where the CPU time limiter was overly lenient for the first several requests handled by a newly-started worker. The same bug actually exists in production as well, but we are much more cautious about fixing it there, since doing so might break live sites. If you find your worker now exceeds CPU time limits in preview, then it is likely exceeding time limits in production as well, but only appearing to work because the limits are too lenient for the first few requests. Such workers will eventually fail in production, too (and always have), so it is best to fix the problem in preview before deploying.
- Major V8 update: 8.0 -> 8.1
- Minor bug fixes.

## 2/13/2020

Changes over the last couple weeks:
- Fixed a bug where if two differently-named scripts within the same account had identical content and were deployed to the same zone, they would be treated as the “same worker”, meaning they would share the same isolate and global variables. This only applied between workers on the same zone, so was not a security threat, but it caused confusion. Now, two differently-named worker scripts will never be considered the same worker even if they have identical content.
- Performance and stability improvements.

## 1/24/2020

It’s been a while since we posted release notes, partly due to the holidays. Here’s what’s new over the past month:
- Performance and stability improvements.
- A rare source of daemonDown errors when processing bursty traffic over HTTP/2 has been eliminated.
- Updated V8 7.9 -> 8.0.

## 12/12/2019

New this week:
- We now pass correct line and column numbers more often when reporting exceptions to the V8 inspector. There remain some cases where the reported line and column numbers will be wrong.
- Fixed a significant source of daemonDown (1105) errors.

## 12/6/2019

Runtime release notes covering the past few weeks:
- Increased total per-request Cache.put() limit to 5GiB.
- Increased individual Cache.put() limits to the lesser of 5GiB or the zone’s normal cache limits ([https://support.cloudflare.com/hc/en-us/articles/200172516-Understanding-Cloudflare-s-CDN](<https://support.cloudflare.com/hc/en-us/articles/200172516-Understanding-Cloudflare-s-CDN>)).
- Added a helpful error message explaining AES decryption failures.
- Some overload errors were erroneously being reported as daemonDown (1105) errors. They have been changed to exceededCpu (1102) errors, which better describes their cause.
- More “internal errors” were converted to useful user-facing errors.
- Stability improvements and bug fixes.