---
layout: default
title: Content Delivery Networks
nav_order: 1300
permalink: /cdns
has_children: false
parent: Overview of Digital Media
has_toc: false
---
# Content Delivery Networks
{: .no_toc }

###### Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

Content Delivery Networks (CDNs) cache frequently used content physically near the target
consumers to reduce the latency in delivering that content and improving overall user experience.

The rise of streaming media (Live-streaming of news and sports, as well as Video on Demand, VoD,
services) is making it increasingly important for media and other companies to use CDNs to
deliver their content in an efficient and performant manner that gives their users a good
experience (e.g. fast starts of videos, low to no buffering, high quality and jitter free
delivery of content, etc.)

CDN companies establish physical caching nodes in various data centers around the world and peer
them with ISPs in the area. When an asset is requested from the CDN, the CDN checks the local
cache, and if that asset is present in the cache (a “cache hit”) it is delivered to the requester
immediately; if that asset is not present in the cache (a “cache miss”) it is requested from the
storage source (the “origin”) and delivered to the requester. The next requester for the same
asset can then benefit from the prior request since the asset will now be cached and served
immediately.

In a streaming scenario, CDNs serve the media segment files. The manifest files themselves may or
may not be served from CDNs depending on the use case. If the manifest file needs to contain
customized information for each viewer, then the manifest file will be served directly by an API
endpoint and refer to media segments being served from the CDN. In cases where all viewers are
likely to view the same version of the program (e.g. in a VOD service), the manifests can also
be served from the CDN.

## CDN for Live Streaming
The profile of Live Streaming is very “bursty”, such as when a large number of people
simultaneously watch significant events (such as popular sporting events, breaking news, etc.)

Live streaming requires a CDN but is very sensitive to latency. Since, in most cases, time
segments are delivered atomically, the latency for watching a live stream is at least as much as
the length of the time segment (plus any time overhead to encode the video at the various
resolutions). Shorter time segments can lead to live stream video being closer in real-time to the
actual event, however, there is a tradeoff with more buffering since the client will have to
update its manifest and content caches more frequently. Some of the more advanced caching systems
are able to deliver a segment before the entirety of the segment has been uploaded to the cache
(by using variations of chunked transfer); in such cases, the latency will be at least as long as
the time represented by the length of the chunk, plus any encoding and processing overhead.

Usually, the time segments are 5-10 seconds long for live streaming content, so a digital
streaming broadcast of an event usually lags the actual real-time cable or broadcast feed by
upwards of 20 seconds or more (when all the cumulative encoding and transit latencies along the
delivery path are accounted for). In many cases, users find out the result of a play
(like a touchdown or a goal) faster via Twitter (and definitely faster on cable or
over-the-air broadcast) than when watching a digital broadcast over a delivery box
(like Chromecast).

Since the live content is always updating, the cache miss rates at the edge are high and
therefore the cache fill dynamics (price and performance) become important. A more detailed
discussion of live streaming and implications for CDNs is given in the media architectures section.

Achieving short time segments with low latency is the holy grail of live video delivery.

## CDN for Video on Demand
VOD serves media from a media library that has already been created. VoD viewing has different
implications for CDNs than live streaming.

Unlike live streaming where a large number of viewers are viewing a very small number of assets,
the situation with VOD is the inverse: customers generally watch different parts of a provider’s
VOD library, and the spread of assets that need to be delivered is wider. For example, if a
streaming service has a library of 5,000 movies, then a few movies will be very popular
(say 10% of the movies will be watched by 90% of the subscribers, though perhaps not
simultaneously), and the remainder of the movies will be watched sporadically by the remaining
10% of subscribers (this phenomenon is often referred to as the “long tail”). Large video
streaming libraries such as YouTube have very significant long tail viewing effects. Specialized
video streaming services (such as those catering to regional soap operas for expats, etc.) have
more concentrated viewing patterns.

## Securing streaming Content on CDNs
Most streaming content (VOD and Live) requires protection from piracy. This means that the
segments in a manifest are available only to authorized users, and indeed, the manifest itself
is only available to authorized users.

There are two broad mechanisms for content protection (which can be combined if needed):
* **Digital Rights Management (DRM)**: in this mode, the content is encrypted after it is
encoded into the different time segments at different resolutions. The encryption keys are
maintained by the DRM provider. The client player on the user’s device presents the end user’s
credentials to obtain the decryption key and then decrypts the content on the device.
* **URL Signing or Signed Cookies**: In some cases, DRM may not be suitable: for instance, the
streaming provider may have customers with older or simpler devices that cannot efficiently
execute content decryption. Also, streaming services may want non-DRM means to control access to
their content, which is where URL Signing or Cookies come in. Signed URLs, if leaked, can provide
access to the content. Coupling signed urls with cookies increases overall security of the asset
since simply sharing a manifest url or a signed asset url will not permit other users (ones
without the right cookies) to view the content.

## Securing the Manifest
Manifests are generated dynamically for live streaming use cases, and are often pre-generated for
VOD. In the case of ad-supported VOD, the manifest might be served via a manifest manipulator
operated by the ad service that inserts the ad segments before sending it to the user.

Media customers protect manifest access through access controls, which includes delivering the
manifest as a signed URL to the end customer.

### Signed Manifest URL: Request Parameters
In this format, the URL address for the manifest is delivered to the authenticated end user with
request parameters appended to the manifest address. So, a logged in user will get the following
style of manifest link to view a video:
```
https://<cdnhost>/<folder>/manifest.m3u8?Expires=<Date>&KeyName=<key_name>&Signature=<hmac_sha1_signature>
```

### Signed Manifest URL: Request Path
In this format, the URL address for the manifest is delivered to the authenticated end user with
signing parameters embedded in the path of the manifest address. So, a logged in user will get
the following style of manifest link to view a video:
```
https://<cdnhost>/<authentication_token>/<folder>/manifest.m3u8
```

In this case, the CDN validates the request for this manifest using logic for inspecting the
path instead of the query parameters.

## Securing the Segments
Any manifest delivered to the end user contains references to the time segments encapsulated by
that manifest. In the case of VOD, the manifests can be very large as they list all the segments
that make up the entire video (e.g. a full length movie). The manifests can embed the time
segment URLs as absolute or relative paths.

### Signed Segment URL: Request Parameters
In this format, the manifest, when delivered to the end customer, contains its media segments to
include the auth parameters for each segment URL. This means that before the manifest is
delivered to a user, the full manifest is updated (re-written) on egress to include the request
parameters for each time segment. This means that manifest manipulation before serving is
required.

In the case of Live Streaming, signed urls are usually lower friction since the manifests are
being continuously generated and the relevance of the content is short lived. Multiple
authenticated users can share the same manifest that contains signed urls with request parameters.

In the case of VOD, where all the manifests have been pre-generated, this can be problematic
since each manifest for each customer will have to be uniquely re-generated on egress. Unless
the CDN provides intrinsic capability to do this operation, the media customers have to
themselves write utilities that perform this action and this increases their overall cost and
complexity. Dynamic packaging can be helpful in this use case.

### Signed Segment URL: Request Path
In this format, the manifest, when delivered to the end customer, contains relative references
to the segments. This means that the url path containing the auth token that was used to deliver
the original manifest can continue to be used to deliver the segments as well.

The CDN has to have the logic to process the token-containing path as part of computing the cache
keys.

In the case of Live Streaming, this can simplify the manifest generation since all segments are
referenced via a relative path, and the prefix part of the path can be used to validate the
access to the content.

In the case of VOD, this is extremely helpful as the manifests do not have to be manipulated by
either the customer or the CDN. The CDN processes the request by disregarding the token
component of the URL when calculating the cache key.

### Signed Segment URL: Signed Prefixes
In some cases, authentication systems can provide a signature that applies to a certain prefix of
URLs (e.g. https://cdn.example.com/video/<assetId>/). Upon loading the manifest, requests to the
segments that start with the prefix can be suffixed with this signature grant and the CDN can
then return the segments that match the signed prefix
(e.g. `https://cdn.example.com/video/<assetId>/segment_001.mp4`)

### Signed Cookie-based Access
In this mechanism, when the manifest is delivered, a signed cookie is delivered along with it.
Cookie signing is done in accordance with the CDN’s requirements, and with a key known to the
CDN instance. This means that all subsequent requests for segments back to the server include the
signed cookie and the CDN is able to validate the cookie before returning the contents of the
segment.

The cookie based mechanism greatly simplifies manifest management, since cookies are part of the
headers in the requests and do not form part of the URL to calculate cache keys. The cookie based
mechanism makes secure access to the content completely transparent and decoupled from the actual
serving URL.

However, in the event of a cache miss, the origin must be able to validate the cookie being
offered by the client.
