---
layout: default
title: Google Cloud Storage
nav_order: 2300
permalink: /gcs.html
has_children: false
parent: Google Cloud for Media
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

# Google Cloud Storage
{: .no_toc }

###### Table of contents
{: .no_toc .text-delta }

- TOC
  {:toc}

# Introduction
Google Cloud Storage (GCS) is a mature, sophisticated storage system that is based
on technology that forms the bedrock foundation of not just the core systems in GCP,
but indeed also for Google’s largest products such as GMail, Drive, and YouTube.
An interesting overview of the Colossus system is given in this [blog post](https://cloud.google.com/blog/products/storage-data-transfer/a-peek-behind-colossus-googles-file-system).

GCS offers three types of bucket configurations:
* **Regional Buckets**: are located in a specific GCP region, such as us-central1 (Council Bluffs, Iowa) or europe-west6 (Zurich, Switzerland)
* **Dual Region Buckets**: a dual region bucket is synchronized across two specific regions in a continent; not all region pairs are supported.
* **Multi-Region Buckets**: a multi-region bucket is synchronized across two or more regions in a specific continent.
Details of all the various regions and region-pairs are available [here](https://cloud.google.com/storage/docs/locations).

GCS offers the following types of storage classes (or “tiers”):
* **Standard** storage is best for data that is frequently accessed ("hot" data) and/or stored for only brief periods of time. There is no minimum storage duration required.
* **Nearline** storage is a low-cost, highly durable storage service for storing infrequently accessed data. A 30-day minimum storage duration is required.
* **Coldline** storage is a very-low-cost, highly durable storage service for storing infrequently accessed data. A 90-day minimum storage duration is required.
* **Archive** storage is the lowest-cost, highly durable storage service for data archiving, online backup, and disaster recovery. Unlike the "coldest" storage services offered by other Cloud providers, your data is available within milliseconds, not hours or days. A 365-day minimum storage duration is required.
* **Autoclass** <<todo write about autoclass>>
Details of all the storage classes and their supported bucket types and other features is available here.

A full exposition on GCS is beyond the scope of this book, but a few important
aspects are worth mentioning:
* **Encryption At Rest**: GCS offers encryption at rest as standard. The default is to use Google managed keys, but you can opt to use your own keys instead. There is no option to turn this off: your data is always encrypted at rest, just that the keys can either be Google managed or customer managed. This feature is provided with no degradation in performance. Indeed, with either encryption mode, GCS still offers read/write performance with lower latencies than other clouds, and within a tight performance band. The [Encryption at Rest Whitepaper](https://cloud.google.com/security/encryption/default-encryption) is a great resource for more in-depth information about this capability.
* **Multi-regional Buckets**: This is a truly unique feature of GCS, made possible by foundational technologies such as the global network and the Colossus file store. Multi-regional buckets allow GCS to store data replicated across multiple regions in the same continent (e.g. us-east1 and us-west1) with a single storage call. This radically simplifies redundant storage, both in terms of cost and operational overhead.
  * **Global Strongly Consistent**: GCS offers various critical operations as globally strongly consistent for both data and metadata; this means that when an object is uploaded to Cloud Storage, and a success response is received, the object is immediately available for download and metadata operations from any location where Google offers service. It must be noted that while the status responses are globally consistent, the actual storage of objects is available in single and multi-region configurations within the same continent.
* **Interoperability API**: Google understands that customers might have made significant investments in utilizing other cloud storage services, particularly AWS S3, and many vendors offer software that natively integrates with AWS S3. To make the use of GCS more frictionless for these customers, GCS offers an “interoperability API” that allows the use of the AWS S3 API for various object and bucket operations with the GCS endpoint. This allows customers to adopt GCS with minimal code or workflow changes.

# Google Cloud Storage Design
<<todo>>

# Google Cloud Storage for Media
Any media system built on GCP will require the use of GCS for content storage.
While external storage systems (such as those on-prem or in other clouds) are
definitely usable, using GCS within GCP offers various advantages in terms of
performance, security configurations, scale, etc. that often make GCS the better
choice in most use cases.

## Origin for CDNs
GCS makes an ideal origin for Cloud CDN and Media CDN. While these CDNs can get
content from non-Google sources, GCS provides scale, resilience, and security
as a content origin to these CDNs. Cloud CDN and Media CDN can use GCS’s private
access functionality via service accounts and secure the content via signing keys.

GCS can be used with non-Google CDNs in various modes:
* **Public Access**: With public content (with object or bucket level permissions allowing non-authenticated access), any CDN that can reach the storage API can use GCS as an origin.
* **Interoperability API**: in ‘interop mode’, any CDN that can use AWS-style access and secret key combinations can access private objects for authenticated access. The CDN can choose to front this with signing keys if required, depending on the CDN’s implementation and configuration options.
* **GCS API**: CDNs that integrate directly with GCS can utilize service account keys to get authenticated access to private objects. The CDN can choose to front this with signing keys if required, depending on the CDN’s implementation and configuration options.

## Naming Conventions
GCS employs various sharding and load distribution mechanisms to provide scale
and resiliency. Object naming plays an important role in how objects are slotted
and stored. Suboptimal naming schemes can lead to degraded performance under load
by causing hotspotting and thus triggering various rebalancing processes within
the system.

This [discussion](https://cloud.google.com/storage/docs/request-rate#naming-convention) provides a good overview of how a naming system can be used to
effectively spread load across the object space and prevent hotspotting and other
artifacts.

In general, starting object names with randomized or hashed values creates enough
variance in the lexicographic order that objects can be distributed widely across
the storage topology.

## Ramping Considerations
In an ideal world, there would be infinite capacity with instantaneous availability.
Indeed, many Cloud operators are also customers of the consumer Google services
and have come to expect the seemingly infinite, instantaneous scale that Google’s
consumer-facing services apparently exhibit. However, it must be noted that
Google’s consumer-facing applications do a fair amount of work under the hood to
enable such seemingly limitless scale.

For Cloud operators looking to get the best performance from GCS, it is important
to think of GCS as a service that is allocating resources dynamically to service
a continuously changing demand profile. GCS is a sharded store and it ramps
serving capacity in the individual storage shards as needs arise. Shards are also
automatically split into smaller shards if there is heavy demand on a particular
shard. Therefore, a sudden demand spike for a certain cluster of assets (such
as, say caches from all over the world reaching out to the same set of objects
for cache fills due to sudden popularity of a certain video) can cause a slight
lag in serving capacity ramp as the system will try to compensate for the increased
load by dynamically managing the shards.

To prevent this potential issue, it is recommended that load be ramped in a
doubling formula as outlined in [this section of the documentation](https://cloud.google.com/storage/docs/request-rate). 

# Google Cloud Storage for Archival
GCS uses an innovative approach to object archival: for GCS, object archival is treated as a billing and operational decision instead of a technology decision. This means that all classes of storage: standard, nearline, coldline, etc. are on the same technology and can be accessed with the same speed (milliseconds). The only difference is that non-standard storage classes (such as coldline) optimize billing for long term object retention and have a higher charge for object retrieval.

GCS is a powerful enabler of media archives since archived content can be instantly retrieved in the event it is suddenly required. In this situation, the archived assets can be quickly replicated to a standard tier bucket for repeated read operations, and can be kept in standard storage for the duration of required usage. Afterwards, the object can be deleted from the standard tier bucket either manually or via an automatic retention policy.

<<todo ...>

# Google Cloud Storage AutoTier
<<todo>>

# Choosing the right GCS Option
<<todo>>






