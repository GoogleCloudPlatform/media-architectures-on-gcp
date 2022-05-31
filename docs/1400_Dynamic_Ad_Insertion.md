---
layout: default
title: Dynamic Ad Insertion
nav_order: 1400
permalink: /dai.html
has_children: false
parent: Overview of Digital Media
has_toc: false
---
# Dynamic Ad Insertion
{: .no_toc }

###### Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

Ads have been in video as long as there has been video. In the over-the-air broadcast era, all
viewers of a broadcast from a station saw the same ads, regardless of their individual
preferences. The promise of digital streaming is that video streamers can finally show viewers
ads that are relevant to the audience, down to the individual level.

The key to inserting ads is to manipulate the manifest file to insert urls to the ads that will
be part of the video stream.

## Ad Insertion Process
When a video viewer makes a request for a manifest, the manifest is served with various “markers”
(for example SCTE 35 markers) that indicate where an ad should be inserted in the content. The
originator of the content (such as a broadcaster) can decide where to place these markers
in a stream. In Live Streams, the markers can be inserted manually during the manifest generation
(such as when an ad needs to be shown during a sports time-out). In VOD Streams, the location of
the desired breaks is generally known (such as the ad breaks in a 1 hour show every ~15 minutes).

Manifests containing these markers indicate when an ad should start and end, and other metadata
that describes various settings for personalization and content description.

Here is an example of an HLS manifest that has basic information about when an ad should start
and end:

<pre>
#EXTINF:10,
http://media.example.com/fileSequence7796.ts
#EXTINF:6,
http://media.example.com/fileSequence7797.ts
<b>#EXT-X-CUE-OUT:DURATION=30</b>
#EXTINF:4,
http://media.example.com/fileSequence7798.ts
#EXTINF:10,
http://media.example.com/fileSequence7799.ts
#EXTINF:10,
http://media.example.com/fileSequence7800.ts
#EXTINF:6,
http://media.example.com/fileSequence7801.ts
<b>#EXT-X-CUE-IN</b>
#EXTINF:4,
http://media.example.com/fileSequence7802.ts
#EXTINF:10,
http://media.example.com/fileSequence7803.ts
#EXTINF:3,
http://media.example.com/fileSequence7804.ts
</pre>

The above manifest is provided to the ad server, which interprets the
cue-out and cue-in markers and replaces the content between them with ads
of the desired duration and profile like the following:

<pre>
#EXTINF:10,
http://media.example.com/fileSequence7796.ts
#EXTINF:6,
http://media.example.com/fileSequence7797.ts
<b>#EXT-X-CUE-OUT:DURATION=30</b>
<b>#EXTINF:10,</b>
<b>http://ads.example.com/fileSequence0001.ts</b>
<b>#EXTINF:10,</b>
<b>http://ads.example.com/fileSequence0002.ts</b>
<b>#EXTINF:10,</b>
<b>http://ads.example.com/fileSequence0003.ts</b>
<b>#EXT-X-CUE-IN</b>
#EXTINF:4,
http://media.example.com/fileSequence7802.ts
#EXTINF:10,
http://media.example.com/fileSequence7803.ts
#EXTINF:3,
http://media.example.com/fileSequence7804.ts
</pre>

In the above example, the segments between the cue-out and cue-in tags
have been substituted by the ad server to point to segments representing
the ads that the ad server wants to show. Refer to Google Ad Manager [HLS
Integration](https://support.google.com/admanager/answer/7245661?hl=en&ref_topic=7335768#zippy=%2Ccue-outcue-in%2Cscte-binary-splice-insert%2Cdaterange%2Csee-examples) for more information (above example sourced from this site).

Here is an example of a DASH manifest that has basic information about when
an ad should start and end:

```xml
<MPD>

  <Period start="PT0S" id="1">
     <!-- Content Period -->
     ...
  </Period>

  <Period start="PT32S" id="2">
      <!-- Ad Break Period -->
      <EventStream timescale="90000" schemeIdUri="urn:scte:scte35:2014:xml+bin">
       <Event duration="2520000" id="1">
       <!-- The duration specified in this event should match the actual
       duration of the period as close as possible -->
         <Signal xmlns="urn:scte:scte35:2013:xml">
           <Binary>
             /DAlAAAAAAAAAP/wFAUAAAAEf+/+kybGyP4BSvaQAAEBAQAArky/3g==
           <Binary>
         </Signal>
        </Event>
      </EventStream> 
  </Period>

  <Period start="PT60S" id="3"> 
    <!-- Content Period -->
    ...
   </Period>
</MPD>
```

In the above example, the `EventStream` marker from the SCTE35 standard
indicates an ad break and the ad server inserts segments that point to
ad content. Refer to the Google Ad Manager [DASH integration](https://support.google.com/admanager/answer/9087202?hl=en&ref_topic=7335768#zippy=%2Cscte-xml-splice-insert%2Csee-examples) for
more information (above example sourced from this site).

As can be seen, there is no information about what ad to actually show in these
manifests. That is the function of the “ad server” that decides what ads are to
be inserted based on the ad tag and other metadata passed to it (this process
is called "decisioning"). When an ad needs to be inserted into a manifest based
on the splice points, a request is made to the ad server for that content. The
request includes the metadata for that particular use case (e.g. type of
program, type of ads designed, name of campaign registered with the ad server,
etc.), and the ad server responds with the appropriate content links.

Ad server responses, and the two means to insert ad content in a video
playback (client side insertion and server side insertion) are all discussed
briefly in the following subsections.

## Ad Server Responses
When an ad break marker is detected in a manifest, an ad server is called that
returns something called a `VMAP` and/or a `VAST` response. This request can be
made by the client viewer showing the content, or by the manifest server sending
the manifest to the client viewer.

`VAST` is an acronym for `Video Ad Serving Template`, which is an XML file that
contains all the information required to serve one or more ads; `VAST` files
contain details such as the ad duration, the display assets (“creatives”)
targeted, tracking events (such as quartiles of the ad viewed), clickthrough
URLs (i.e. where to take the viewer if the ad is clicked upon), etc. Each `VAST`
response refers to a single ad. In many cases, multiple ads are part of an ad
break.

A simple VAST example is viewable [here](https://pubads.g.doubleclick.net/gampad/ads?sz=640x480&iu=/124319096/external/single_ad_samples&ciu_szs=300x250&impl=s&gdfp_req=1&env=vp&output=vast&unviewed_position_start=1&cust_params=deployment%3Ddevsite%26sample_ct%3Dlinear&correlator=).
In this XML, you can see the various sections that detail various aspects of
the ad, such as: media files, impression and error beacons, tracking event
beacons, etc.


`VMAP` (`Video Multiple Ad Playlist`) are XML files that contain multiple `VAST`
XMLs, either as references to `VAST` URIs to ad servers or as direct embedded
XML. Ads that appear in line with the content before (“pre-roll”), during
(“mid-roll”), or after (“post-roll”) the content are called “linear” ad 
breaks. A sample of a `VMAP` containing Pre-, Mid-, and Post-rolls, Single Ads
is given here. If you follow the links within each
```xml
<vmap:AdTagURI templateType="vast3">...</vmap:AdTagURI>
```
section, you’ll be taken to a `VAST` XML file that contains data about one ad.

Unfortunately, a full study of `VAST` and `VMAP` is beyond the scope of this
document. The IAB (Interactive Advertising Bureau, an industry consortium of
leading advertisers) has a detailed handbook on VAST and VMAP available
[here](https://www.iab.com/wp-content/uploads/2015/06/VMAPv1_0.pdf). Additional
samples of various VAST and VMAP formats can be found [here](https://developers.google.com/interactive-media-ads/docs/sdks/html5/client-side/tags).

## Client-Side Ad Insertion (CSAI)
In this case, the video player showing the content inserts the ads. When the
client receives the manifest with ad breaks, it calls out to a preconfigured
ad server and requests the segments for the ad break. The server responds with
the appropriate `VAST` or `VMAP` XML. The client then makes the request for the
ad segments listed in the VAST/VMAP response from the server. The ad content is
then shown to the end viewer as part of the program. The overall flow can be
shown as follows:

![client side ad insertion](/assets/images/dai-csai.png)

In the above example:
1. the client SDK makes a call for the content manifest
2. and gets a manifest that includes the ad break
3. the client then calls the ad server
4. and is returned a VAST or VMAP response
5. The client SDK then requests the individual media files from the specific locations (same process as steps 1 & 2)
6. and transmits the viewership and other metrics like quartiles to the analytics service.

CSAI does have a few caveats:
* If a customer has enabled ad blockers, the calls to the ad server could be blocked and it is possible that the ads don’t load, thus resulting in loss of revenue to the content provider
* Depending on the device capability and network latencies, the ads may take some time to load, and could lead to a suboptimal user experience where they see the post-ad content (in the case of VOD) or see a brief gap (in the case of Live) before the ad starts playing

## Server-Side Ad Insertion (SSAI)
In this case, ads are inserted by the ad server and the manifests received by
the client are fully pre-baked with ad segment URLs.

![server side ad insertion](/assets/images/dai-ssai.png)

In SSAI, the DAI server is responsible for serving the manifests to the client.
1. When a client requests content from the DAI Server
2. the DAI server contacts the origin and gets the manifest and analyzes it.
3. For the ad markers contained in the manifest, the DAI server contacts the ad server and receives the VAST/VMAP response.
4. The DAI server then merges the media content links included in the VAST files into the manifest that is sent to the client viewer.
5. The client requests the various segments for the content and ads as indicated in the combined manifest.
6. The client continues to send the various tracking information (beacons) to the analytics service based on the URLs of the tracking links.

A key item to note is that in the case of SSAI, the DAI server is responsible
for serving the HLS or DASH manifest files to the client, whereas in CCAI,
the broadcaster serves the content manifest files to the client.

