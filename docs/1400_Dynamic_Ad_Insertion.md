---
layout: default
title: Dynamic Ad Insertion
nav_order: 1200
permalink: /dai
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
```text
#EXTINF:10,
http://media.example.com/fileSequence7796.ts
#EXTINF:6,
http://media.example.com/fileSequence7797.ts
`**#EXT-X-CUE-OUT:DURATION=30`**
#EXTINF:4,
http://media.example.com/fileSequence7798.ts
#EXTINF:10,
http://media.example.com/fileSequence7799.ts
#EXTINF:10,
http://media.example.com/fileSequence7800.ts
#EXTINF:6,
http://media.example.com/fileSequence7801.ts
#EXT-X-CUE-IN
#EXTINF:4,
http://media.example.com/fileSequence7802.ts
#EXTINF:10,
http://media.example.com/fileSequence7803.ts
#EXTINF:3,
http://media.example.com/fileSequence7804.ts
```