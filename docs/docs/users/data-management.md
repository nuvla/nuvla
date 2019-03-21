---
layout: default
title: Data Management
parent: Users
nav_order: 1
---

# Data Management

The data management infrastructure takes advantage of object storage
services that expose the S3 protocol, a _de facto_ standard from
Amazon Web Services.

## Model

Write-Once, Read-Mostly (WORM) semantics

Provides best guarantees for consistency and integrity in distributed
environment.  Facilitates replication of data objects for increased
performance or for redundancy.  Versioning in WORM model allows for
“updatable” objects.

Streaming is supported, but expected to be limited mostly to direct
sensor data at the edge.

## `data-object` Resources

Represents a blob of data (“file”) stored as an S3 object

 - Ubiquity: S3 is a de facto standard available everywhere.
 
 - Performance: data directly stored to and accessed from the S3
   service.
 
 - Security: use of time-limited, pre-signed S3 requests, allows
   uniform AA model based on SlipStream users and ACLs.
 
 - Global View: collection of “data-object” resources provides global
   view of available data across all computing infrastructures.
 
 - Core Metadata: can provide name, description, tags, size, checksum
   etc. for simple searches over the available data.

### Managing `data-object` Resources with the API

## `data-record` Resources

Provides rich metadata for a “data-object”.

 - Provides an open schema to allow collaborations to provide their
   own metadata for data-object resources.

 - Simple filter syntax can use any defined attributes to refine
   queries over the stored objects.

 - Forced definition of prefixes avoids name conflicts between
   collaborations.

 - Recommended definition of keys provides semantic information for
   humans creating the metadata.

### Managing `data-record` Resources with the API

## `data-set` Resources

Provides dynamic grouping of “data-object” resources.

A data set definition contains filters for:

 - data-object resources,

 - data-record resources, and/or

 - linked applications.

Users can create their own data sets by filtering the existing objects
and share data set definitions with others.

### Managing `data-set` Resources with the API

