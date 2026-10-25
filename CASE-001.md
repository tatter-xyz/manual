# The feature is present. What follows?

**Case 001 · edition 0.1.0 · 25 October 2026.**

You hear about a decoder problem. Your version output mentions Brotli. Before announcing that an application is exposed, there are a few rather inconvenient questions between those two facts.

## First, read what was actually supplied

The [curl reader 1.0.0](https://github.com/tatter-xyz/curl-reader/tree/v1.0.0) parses pasted inventory text. It makes no network request, executes no command and has no vulnerability database. It keeps the reported curl and libcurl versions separate.

Clone or download that exact release. In its directory, save the following as `example.mjs`, then run `node example.mjs` with Node's built-in ES-module support:

```js
import assert from 'node:assert/strict';
import {readCurlVersion} from './curl-reader.mjs';

// Invented inventory, not an affected-version claim.
const listed = `curl 8.10.1 (example-platform) libcurl/8.10.1 brotli/1.1.0
Protocols: http https
Features: SSL brotli`;

const absent = `curl 8.10.1 (example-platform) libcurl/8.10.1
Protocols: http https
Features: SSL`;

const conflicting = `curl 8.10.1 (example-platform) libcurl/8.10.1 brotli/1.1.0
Protocols: http https
Features: SSL`;

assert.equal(readCurlVersion(listed).brotli, 'listed');
assert.equal(readCurlVersion(absent).brotli, 'not-listed');
assert.equal(readCurlVersion(conflicting).state, 'unassessed');
console.log('3 inventory examples passed. Deployment exposure remains unassessed.');
```

The published module produces that message for these examples. The first record lists a feature; the second does not; the third contradicts itself and stays unassessed. These are offline parser observations. They establish nothing about a live installation or an exploit.

For your own inventory, collect one complete `curl --version` output from the relevant environment; Windows PowerShell users can use `curl.exe --version`. An application's embedded libcurl or another container can differ from your terminal's executable. Never paste credentials or authenticated traffic into a public issue.

## Then follow the actual route

Consider the curl source revision [5108047d15ad6d9fa17ebd80ae26ebd7c949264e](https://github.com/curl/curl/tree/5108047d15ad6d9fa17ebd80ae26ebd7c949264e), examined on 24 October 2026. This is a version-specific source example, not a claim about every later release.

The [encoding option documentation](https://github.com/curl/curl/blob/5108047d15ad6d9fa17ebd80ae26ebd7c949264e/docs/libcurl/opts/CURLOPT_ACCEPT_ENCODING.md) explains that a server can return an encoding other than the requested one. Setting this option to a non-null value enables automatic response decoding; a manually supplied request header alone does not do that.

In the selected [response-header path](https://github.com/curl/curl/blob/5108047d15ad6d9fa17ebd80ae26ebd7c949264e/lib/http.c#L3266-L3275), response headers can lead to the decoding stack. The [Brotli writer](https://github.com/curl/curl/blob/5108047d15ad6d9fa17ebd80ae26ebd7c949264e/lib/content_encoding.c#L427-L460) calls the streaming decoder; its registration depends on the build. The separate [content-decoding option](https://github.com/curl/curl/blob/5108047d15ad6d9fa17ebd80ae26ebd7c949264e/docs/libcurl/opts/CURLOPT_HTTP_CONTENT_DECODING.md) can disable this content decoding.

That gives a conditional route:

```text
application's actual libcurl build and options
  -> response headers select a content-decoding path
  -> returned body reaches the registered decoder
  -> the actual linked decoder and patch provenance matter
```

Requesting `identity` is therefore not, by itself, proof that this configured route cannot decode a Brotli response. A very polite request, though.

## The questions left on the page

| Question | Evidence needed |
| --- | --- |
| Which build does the application use? | Actual loaded or static dependency provenance from that environment. |
| Is this decoding route enabled? | The application's relevant options and build configuration. |
| What reaches it? | Appropriate controlled response evidence from an authorized environment. |
| Does that decoder contain a particular flaw or fix? | Exact implementation, vendor patch/backport and advisory evidence for the stated flaw. |
| Was impact demonstrated? | A separate appropriately scoped validation; presence alone is insufficient. |

This case does not supply those deployment facts. Changing a decoding option can change behavior; it does not patch a library or cover every other consumer.

The useful result is a narrower conclusion and the next evidence to collect. A missing answer stays missing. It does not inherit confidence from the rather nice diagram.

[Propose a correction or another case](CONTRIBUTING.md) · [Back to the Manual](README.md)
