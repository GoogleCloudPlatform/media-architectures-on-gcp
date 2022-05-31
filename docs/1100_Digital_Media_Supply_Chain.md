---
layout: default
title: Digital Media Supply Chain
nav_order: 1100
permalink: /dmsc.html
has_children: false
parent: Overview of Digital Media
has_toc: false
---
<!--
Copyright 2022 Google LLC

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# Digital Media Supply Chain
{: .no_toc }

###### Table of contents
{: .no_toc .text-delta }

- TOC 
{:toc}

## Introduction
At a high level, the digital media supply chain takes videos from a source (a camera in case of live video,
and storage in case of on-demand video) and delivers it successfully to a viewing device over the internet.

The digital media supply chain varies from traditional broadcast in that the cloud and distributed systems
play a substantial role in the management and distribution of the content.

There are two broad categories of media supply chains: Segments and Streams.

**Segments** are generally “finished” media files of limited time length (generally between 2-30 seconds) that
represent media chunks of a larger media asset. For instance, a 2 hour movie might be fragmented into 720 segments
of 10 seconds each. In reality, these segments will be replicated for various resolutions, so if there are 4
resolutions (e.g. 480p, 720p, 1080p, and 4k), then there will be 2880 segments of 10 seconds each. In a simple sense,
one can think of “segments” as being relevant for “B2C” distribution, as when a digital video service delivers content
to a viewers home TV; this is also referred to as “Direct to Consumer” or DTC. Segments are applicable for both live
and on-demand DTC video.

**Streams** are generally intermediate assets, and instead of being fragmented, represent a continuous flow of media
information. These streams are generally used in transmitting live events, and generally between various media
distribution partners (e.g. a sports league takes their streams from the various stadiums where games are occurring
and relays them to various media channels that are carrying those games live). In a simple sense, one can think of
streams as being relevant for “B2B” distribution, as when a content producer is handing off content to a content
distributor for further processing and distribution. This is generally referred to as _“Broadcast in the Cloud”_ or
_“Broadcast Modernization”_.


## Direct to Consumer (DTC) Supply Chain (B2C)
The DTC supply chain can be thought of as the “business-to-consumer” aspect of media distribution, and one that most
readers probably think of when they think of “digital media”. DTC is represented by the ecosystem of the various
dongles and apps that deliver content for viewing at home (such as on smart TVs, tablets, phones, and dedicated
streaming devices).

Of the two categories outlined above, DTC is mostly the “segment” style of distribution. The viewer at home is seeing
a video synthesized from these various segments into a continuous experience.

The DTC supply chain has 4 broad steps:
1. Ingest
2. Processing 
3. Packaging 
4. Distribution

Each of the above is detailed below.

### Ingest
Ingestion is the process of receiving a digital media source. This can be a continuous stream (for live streaming)
or a defined asset file (such as, perhaps, a multi-GB high resolution version of a movie, for
video on demand streaming).

In the case of a stream, content is ingested by a cloud-based system that is receiving the stream traffic on a
certain IP:port combination. In the case of the file-based asset, the asset is usually dropped into an object store.

In both cases, the originator of the content usually sends the content to the cloud via a high bandwidth network
connection, such as a cloud interconnect.

### Processing
Once the content has been received, it is processed (encoded or transcoded) into a format that will eventually be
used by the customer. While in many cases, the words “encode” and “transcode” are used interchangeably, there is a
slight difference between the two; in the most basic sense:
* **Encoding** is the process of initially compressing a raw byte stream (such as one being received directly from a digital camera); commonly, on-premise hardware-based encoders (such as at a studio or a stadium) create a high quality feed from the live footage and transmit that to the cloud
* **Transcoding** is the process of taking an encoded stream, decoding it, and then re-encoding it to a different format or specification; commonly, the encoded stream received from a live or VOD source is transcoded into various formats suitable for consumption by the different types of clients and devices that will view the content

### Packaging
Once the media has been suitably transcoded, it needs to be “packaged” in the correct format, commonly the
HLS and DASH formats. HLS and DASH are both manifest-based formats (explained in the next section), and the
packager takes the transcoded stream and creates the correct segments and manifest files from the transcoded content.

### Distribution
The packaged content is deposited in an object store that can function as an origin for wide distribution via a CDN.
Distribution makes this packaged content widely available to end viewers for viewing live or on-demand.

## Broadcast In the Cloud (B2B)
The “_Broadcast Modernization_” or “_Broadcast in the Cloud_” supply chain can be thought of as the B2B supply chain,
where two different media intermediaries (such as a broadcaster and a media channel) are involved. For example,
a broadcaster or a sports league might be sending a signal to a digital channel for that channel to transmit the
content to its viewers. In this case, the digital channel will incorporate the DTC flow (its ingest will be of the
stream being sent by the broadcaster or the sports league), while the broadcaster will incorporate the B2B flow up to
the point the content is delivered to the digital channel.

Of the two categories mentioned at the start of the chapter, this is the “stream” based flow. In this case, the parties
are connected by a continuous stream of data over a network connection. Usually, to guard against latency and jitter
and other network artifacts, these streams are wrapped in various transmission protocols (such as SRT): these protocols
generally use UDP for speed but have packet counting and retransmit mechanisms to account for losses and drops.
This allows for higher bandwidth utilization and lower latency using UDP while getting the transmission assurances
associated with TCP.

In many cases, the same broadcaster will have a Broadcast workflow as a precursor to the DTC workflow outlined above.

The Broadcast in the Cloud supply chain has the following broad steps:
1. Ingest 
2. Production 
3. Processing 
4. Network Distribution
5. Optional: Direct Distribution (DTC)

### Ingest
Ingestion of media in a B2B scenario is generally as a continuous transport stream. Latency and bandwidth are critical
considerations in this situation, so this ingest usually occurs over a dedicated link into the cloud. This dedicated
link allows a reliable network connection with defined bandwidth characteristics to transport the media stream from the
source (such as a studio or a stadium) into a geographically close cloud region for production and processing.

### Production
Often, the video captured on-site (studio, stadium, etc.) has some level of production workflow applied to it.
However, in a traditional broadcast workflow, these extra production changes (e.g. adding graphics, score overlays,
etc.) occur in the broadcast center that is in a production facility at a broadcaster’s HQ. Increasingly, these
production functions are getting virtualized so it is possible to produce the event using virtual desktop
infrastructure (VDI) in the cloud. This cloud-based approach has significant advantages in terms of flexibility and
convenience for the video event producers.

### Processing
Before the produced content can be distributed to the various partners, some level of preparation of the content might
be required. This can include converting the stream into an appropriate format requested by the distribution partners,
adding various types of metadata, etc. In a broadcast scenario, the output of this step is still a stream.

### Network Distribution
The stream output from the above steps is then transmitted to the distribution partners over a network connection. In
traditional broadcasting, this is done via satellite or dedicated links that various producer/distributor counterparties
have created over time. However, in a cloud based system, the IP-packet based networks are the primary transmission
infrastructure. As noted earlier, this type of distribution usually occurs via an overlay protocol like SRT to
minimize packet loss.

Various optimizations and configurations are essential to ensure proper delivery of the stream from the producer
to the distributor. These requirements are discussed in more detail in the cloud based architecture chapters in the
second section of this book.

### Optional: Direct Distribution
In some cases, the broadcaster also has their own proprietary streaming channels. In this case, the broadcaster will
likely also be a receiver of their processed streams and act like a distributor. In that case, these streams will be
segmented (transcoded and packaged) and then delivered to the viewing apps via a CDN. The workflows required for this
step will be very similar to what a 3rd party distributor will be performing to bring the content to their respective
end viewers.

