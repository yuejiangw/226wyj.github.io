---
layout: post
title: template page
categories: [cate1, cate2]
description: some word here
keywords: keyword1, keyword2
---

<!-----



Conversion time: 2.579 seconds.


Using this Markdown file:

1. Paste this output into your source file.
2. See the notes and action items below regarding this conversion run.
3. Check the rendered output (headings, lists, code blocks, tables) for proper
   formatting and use a linkchecker before you publish this page.

Conversion notes:

* Docs to Markdown version 1.0β35
* Thu Jan 04 2024 22:30:15 GMT-0800 (PST)
* Source doc: Elasticsearch Learn
* Tables are currently converted to HTML tables.
----->



## Elasticsearch Learn

[Udemy Course Link](https://www.udemy.com/course/elasticsearch-complete-guide/)

## Basic Architecture

### Node

An instance of Elasticsearch that stores data. We can run as many nodes as we want, each node will store part of our data. In this way we can store data on multiple virtual or physical machines.

### Cluster

Each node belongs to a cluster. A cluster is a collection of related nodes that together contain all of our data. You might run multiple clusters that serve different purposes.

When a node starts up, it will either join an existing cluster if configured to do so, or it will create its own cluster consisting of just that node. A Elasticsearch node will always belong to a cluster.

### Document

Each unit of data that you store within your cluster is called a document. Documents are JSON objects containing whatever data you desire. 

When you index a document, the original JSON object that you sent to Elasticsearch is stored along with some metadata that Elasticsearch uses internally.

Documents are immutable.

### Index

An index is a collection of documents that have similar characteristics and are logically related.

Every document within Elasticsearch is stored within an index. An index groups documents together logically, as well as provide configuration options that are related to scalability and availability.

### Summary

* Nodes store the data that we add to Elasticsearch.
* A cluster is a collection of nodes.
* Data is stored as documents, which are JSON objects.
* Documents are grouped together with indices.

## Sharding

### Introduction

* Sharding is a way to divide indices into smaller pieces.
* Each piece is referred to as a shard.
* Sharding is done at the index level.
* The main purpose is to horizontally scale the data volume.

### Purpose of sharding

* Mainly to be able to store more documents.
* To easier fit large indices onto nodes.
* Improved performance.
    * Parallelization of queries increases the throughput of an index.

### Configure the number of shards

* An index contains a single shard by default.
* Indices in Elasticsearch &lt; 7.0.0 were created with five shards
    * This often led to over-sharding.
* Increase the number of shards with the Split API.
* Reduce the number of shards with the Shrink API.

## Replication

### Introduction

* Hardware can fail at any time, so we need to handle that somehow.
* Elasticsearch supports replication for fault tolerance. It is supported natively and enabled by default.
* Replication is configured at the index level.

### Increase availability

* Replication works by creating copies of shards, referred to as replica shards
* A shard that has been replicated, is called a **_primary shard_**. A copy of the primary shard is called a **_replica shard_**.
* A primary shard and its replica shards are referred to as a **_replication group_**.
* Replica shards are a complete copy of a shard, and can serve search requests, exactly like its primary shard.
* The number of replicas can be configured at index creation.

### Increase query throughput

* Replica shards of a replication group can serve different search requests simultaneously.
* Elasticsearch intelligently routes requests to the best shard.

## Snapshots

* Commonly used for daily backups
* Manual snapshots may be taken before applying changes to data

## Node roles

### Master-eligible

* The node may be elected as the cluster’s master node.
* A master node is responsible for creating and deleting indices, among others.
* A node with role will not automatically become the master node
    * Unless there are no other master-eligible nodes
* May be used for having dedicated master nodes
    * Useful for large clusters

### Data

* Enables a node to store data
* Storing data includes performing queries related to that data, such as search queries.
* For relatively small clusters, this role is almost always enabled
* Useful for having dedicated master does
* Used as part of configuring a dedicated master node

### Ingest

* Enables a node to run ingest pipelines
* Ingest pipelines are a series of steps (processors) that are performed when indexing documents
    * Processors may manipulate documents, e.g. resolving an IP to lat/lon
* A simplified version of Logstash, directly within Elasticsearch
* This role is mainly useful for having dedicated ingest nodes

### Machine learning

* Node.ml identifies a node as a machine learning node
    * This lets the node run machine learning jobs
* Xpack.ml.enabled enables or disables the machine learning API for the node

### Coordination

* Coordination refers to the distribution of queries and the aggregation of results
* Useful for coordination-only nodes (for large clusters)
* Configured by disabling all other roles.

### Voting-only

* Rarely used, and you almost certainly won’t use it either
* A node with this role, will participate in the voting for a new master node
* The node cannot be elected as the master node itself, though
* Only used for large clusters

## Optimistic Concurrency Control

* Sending write requests to Elasticsearch concurrently may overwrite changes made by other concurrent processes.
* Traditionally, the _version field was used to prevent this.
* Today, we use the _primary_term and _seq_no fields.
* Elasticsearch will reject the write operation if it contains the wrong primary term or sequence number.
    * This should be handled at the application level.

## Analyzer

### Character filters

* Adds, removes, or changes characters
* Analyzers contain zero or more character filters
* CHaracter filters are applied in the order in which they are specified

### Tokenizer

* An analyzer contains one tokenizer
* Tokenizes a string, i.e. splits it into tokens
* Characters may be stripped as part of the tokenization

### Token filters

* Receive the output of the tokenizer as input
* A token filter can add, remove, or modify tokens
* An analyzer contains zero or more token filters
* Token filters are applied in the order in which they are specified

## Inverted Indices

* Mapping between terms and which documents contain them
    * Terms refers to the tokens that are emitted by the analyzer
    * Token is only used in the context of analyzers, outside the context of analyzers, we use the terminology “terms”
* Terms are sorted alphabetically
* Inverted indices contain more than just terms and document IDs
    * E.g. information for relevance scoring
* One inverted index per text field
* Other data types use BKD trees, for instance

## Date in Elasticsearch

* Dates are specified in one of three ways:
    * Specially formatted strings (defaults to ISO 8601 - can use custom formats)
    * Milliseconds since the epoch
    * Seconds since the epoch
* Dates are stored as long values internally (converted to UTF first)
    * The same conversion happens for search queries
* Don’t provide UNIX timestamps for default date fields

## Stop words



* Words that are filtered out during text analysis
    * Common words such as “a”, “the”, “at”, “of”, “on”, etc
* They provide little to no value for relevance scoring
* Fairly common to remove such words	
    * Less common in Elasticsearch today than in the past
        * The relevance algorithm has been improved significantly
* Not removed by default.


## Built-in analyzers


### Standard analyzer



* Splits text at word boundaries and removes punctuation
    * Done by the standard tokenizer
* Lowercases letters with the lowercase token filter
* Contains the stop token filter (disabled by default)


### Simple analyzer



* Similar to the standard analyzer
    * Splits into tokens when encountering anything else than letters
* Lowercases letters with the lowercase tokenizer
    * Unusual and a performance hack


### Whitespace analyzer



* Splits text into tokens by whitespace
* Does not lowercase letters


### Keyword analyzer



* No-op analyzer that leaves the input text intact
    * It simply outputs it as a single token
* Used for keyword fields by default
    * Used for exact matching


### Open & Closed indices



* An open index is available for indexing and search requests
* A closed index will refuse requests
    * Read and write requests are blocked
    * Necessary for performing some operations


### Dynamic and static settings



* Dynamic settings can be changed without closing the index first
    * Requires no downtime
* Static settings require the index to be closed first
    * The index will be briefly unavailable
* Analysis settings are static settings


## Query Samples

Suppose we’re having an index whose name is `products` . The structure of such index is shown as follows:


### View the mapping structure


### Range Searches



* Use the `range` query to perform range searches
* Specify one or more of the `gt`, `gte`, `lt`, or `lte` parameters
* Supports both numbers and dates
* Dates are automatically handled for `date` fields
    * Specifying the time is optional, but recommended if possible
    * Custom formats are supported through the `format` parameter
    * Time zones are handled with the `time_zone` parameter (UTC offset)

<table>
  <tr>
   <td>
<strong>Name</strong>
   </td>
   <td><strong>Represents for</strong>
   </td>
   <td><strong>Symbol</strong>
   </td>
  </tr>
  <tr>
   <td>lt
   </td>
   <td>Less than
   </td>
   <td>&lt;
   </td>
  </tr>
  <tr>
   <td>gt
   </td>
   <td>Greater than
   </td>
   <td>>
   </td>
  </tr>
  <tr>
   <td>lte
   </td>
   <td>Less than or equal to
   </td>
   <td>&lt;=
   </td>
  </tr>
  <tr>
   <td>gte
   </td>
   <td>Greater than or equal to
   </td>
   <td>>=
   </td>
  </tr>
</table>



### Prefix Query


### Wildcard Query

There are two types of wildcards:



* ?: a question mark matches any single character
* *: an asterisk matches any character zero or more times


### Regular Expressions


### Query by field existence



* The `exists` query matches fields that have an `indexed` value
* Field values are only indexed if they are considered non-empty
    * `NULL` and empty arrays are empty values - empty strings are not
    * There are a few other cases where values are not indexed
* The `exists` query can be inverted by using the `bool` query’s `must_not` occurrence type

For the query above, it is the same as: `SELECT * FROM products WHERE tags IS NOT NULL`

To invert the query, the only way is to use the bool query:


### Full text query



* Term level queries are used for exact matching on structured data
* Full text queries are used for searching unstructured text data
    * E.g. website content, news articles, emails, chats, transcripts, etc.
    * Often used for long texts
    * We don’t know which values a may field contain
* Full text queries are analyzed and are not used for exact matching
    * But term level queries are not and therefore used for exact matching
* Don’t use full text queries on `keyword` fields
    * That compares analyzed values with non-analyzed values


#### `Match` query



* The most widely used full text query
* Matches documents that contain one or more of the specified terms
* The search term is analyzed and the result is looked up in the field’s inverted index
* It has a `operator` field, the default value is `or `


#### Relevance Scoring



* Query results are sorted descendingly by the `_score` metadata field
    * A floating point number of how well a document matches a query
* Documents matching term level queries are generally scored 1.0
    * Either a documents matches, or it doesn’t (simply filtered out)
* Full text queries are not for exact matching
    * How well a document matches is now a factor
    * The most relevant results are placed highest (e.g., like on Google)


### Multiple Fields Query



* The `multi_match` query performs full text searches on multiple fields
    * A document matches if at least one field is matched 
* Individual fields can be relevance boosted by modifying the field name
* Internally, Elasticsearch rewrites the query to simplify things for us
* By default, the best matching field’s relevance score is used for the document
    * Can be configured with the `type` parameter 
* A tie breaker can be added to factor in every matching field’s relevance score

The above query is equivalent to the following queries:


### Phrase searches



* The `match_phrase` query is similar to the `match` query in some ways
* For the `match_phrase` query, the position (and thereby order) of terms matters
* Terms must appear in the correct order and with no other terms in-between
* The `standard` analyzer’s tokenizer outputs term positions that are stored within the field’s inverted index
    * These positions are then used for phrase searches (among others)

A phrase is a sequence of multiple words

For phrases to match, the terms must appear in the correct order and with no other terms in-between.


### Boolean Query



* If a `bool` query only contains `should` clauses, at least one must match
* Useful if you just want something to match and reward matching documents
    * If nothing were required to match, we would get irrelevant results
* If a query clause exists for `must`, `must_not`, or `filter`, no `should` clause is required to match
    * Any `should` clauses are only used to boost relevance scores


#### The `filter` occurrence type



* Query clauses must match
* Similar to the `must` occurrence type
* Ignores relevance scores
    * This improves the performance of the query
    * Query results can be catched and reused


### Boosting Query



* The `bool` query can increase relevance scores with `should`
* The `boosting` query can decrease relevance scores with `negative`
    * Documents must match the `positive` query clause
    * Documents that match the `negative` query clause have their relevance scores decreased
    * Use `match_all` query for `positive` if you don’t want to filter documents
    * Can be used with any query (including compound queries, such as `bool`)


### Disjunction max (dis_max)

* The `dis_max` query is a compound query
    * A document matches if at least one leaf query matches
* The best matching query clause’s relevance score is used for a document’s `_score`
* `tie_breaker` can be used to “reward” documents that match multiple queries
* `multi_match` queries are often translated into `dis_max` queries internally


### Nested Query



* Use the `nested` data type if you want to query objects independently
    * Otherwise the relationships between object values are not maintained
    * Each nested object is indexed as a hidden Lucene document
* Use the `nested` query on fields with the `nested` data type
    * Elasticsearch then handles everything automatically
* Use the `score_mode` parameter to adjust relevance scoring


#### Nested inner hits

* Nested inner hits tell us which nested object(s) matched a query
    * E.g. which ingredient(s) matched in a recipe
* Without inner hits, we only know that something matched
* Simply add the `inner_hits` parameter to the `nested` query
    * Supply `{}` as the value for the default behavior
    * Information about the matched nested object(s) is added to search results 
    * Use the `offset` key to find each object’s position within `_source`
    * Customize results with the `name` and `size` parameters


#### Nested fields limitations

* We need to use a specialized data type (nested) and query (nested)
* Max 50 `nested` fields per index
    * Can be increased with the `index.mapping.nested_fields.limit` setting
* A document can have up to 10000 nested objects by default
    * This includes all `nested` fields


## Controlling Query Results


### Format query results

To format the query results, add `?pretty` as a parameter at the end of your query URL


### Source filtering

This is useful if you only want certain fields of the results

Suppose you’re only interested in the `ingredients.name` field

Suppose you’re interested in all the other fields in `ingredients` except `name`


### Specifying the result size

There are two ways to do this:



* By adding a size query parameter to the request URL
* By adding a size parameter within the request body

The default result size is 10.


### Specifying an offset

By adding a `from` in the request body


### Pagination

total_pages_ = _ceil(total_hits / page_size)

from = (page_size * (page_number - 1))

Pagination is limited to 10,000 results, that’s because as the pagination goes deeper and deeper, it will take more and more heap memory, and the request will take longer


### Sorting results

Suppose we want to sort the query results based on the value of `preparation_time_minutes`

To sort the results based on the created time in an descending order

To sort the results based on multiple fields:

To sort based on a field that has multiple values, we need to specify a sort mode:

There are five modes: `avg`, `sum`, `median`, `min`, `max`

`Min` and `max` are not limited to numbers, they work on Date as well

Suppose `ratings` is an array of float numbers


### Filters
