KIP-994: Minor Enhancements to ListTransactions and DescribeTransactions
APIs
================

Created by [Raman
Verma](https://cwiki.apache.org/confluence/display/~ramanverma), last
modified on [Jan 17,
2024](https://cwiki.apache.org/confluence/pages/diffpagesbyversion.action?pageId=276106184&selectedPageVersions=24&selectedPageVersions=25 "Show changes")

- [Status](https://cwiki.apache.org/confluence/display/KAFKA/KIP-994%3A+Minor+Enhancements+to+ListTransactions+and+DescribeTransactions+APIs#KIP994:MinorEnhancementstoListTransactionsandDescribeTransactionsAPIs-Status)

- [Motivation](https://cwiki.apache.org/confluence/display/KAFKA/KIP-994%3A+Minor+Enhancements+to+ListTransactions+and+DescribeTransactions+APIs#KIP994:MinorEnhancementstoListTransactionsandDescribeTransactionsAPIs-Motivation)

- [Public
  Interfaces](https://cwiki.apache.org/confluence/display/KAFKA/KIP-994%3A+Minor+Enhancements+to+ListTransactions+and+DescribeTransactions+APIs#KIP994:MinorEnhancementstoListTransactionsandDescribeTransactionsAPIs-PublicInterfaces)

  - [Command Line Tool
    Changes](https://cwiki.apache.org/confluence/display/KAFKA/KIP-994%3A+Minor+Enhancements+to+ListTransactions+and+DescribeTransactions+APIs#KIP994:MinorEnhancementstoListTransactionsandDescribeTransactionsAPIs-CommandLineToolChanges)

- [Proposed
  Changes](https://cwiki.apache.org/confluence/display/KAFKA/KIP-994%3A+Minor+Enhancements+to+ListTransactions+and+DescribeTransactions+APIs#KIP994:MinorEnhancementstoListTransactionsandDescribeTransactionsAPIs-ProposedChanges)

- [Compatibility, Deprecation, and Migration
  Plan](https://cwiki.apache.org/confluence/display/KAFKA/KIP-994%3A+Minor+Enhancements+to+ListTransactions+and+DescribeTransactions+APIs#KIP994:MinorEnhancementstoListTransactionsandDescribeTransactionsAPIs-Compatibility,Deprecation,andMigrationPlan)

- [Rejected
  Alternatives](https://cwiki.apache.org/confluence/display/KAFKA/KIP-994%3A+Minor+Enhancements+to+ListTransactions+and+DescribeTransactions+APIs#KIP994:MinorEnhancementstoListTransactionsandDescribeTransactionsAPIs-RejectedAlternatives)

# Status

**Current state**: *Accepted*

**Discussion thread**:
[*here*](https://lists.apache.org/thread/02otvz49hbk9zxvg850sr07dvbrjxypd)
*(Vote thread:
[here](https://lists.apache.org/thread/yknx3bc4mk17bz2cpfr789lh8sx2lc39))  
*

**JIRA**: [*here*](https://issues.apache.org/jira/browse/KAFKA-15923)*  
*

Please keep the discussion on the mailing list rather than commenting on
the wiki (wiki discussions get unwieldy fast).

# Motivation

KIP-664 introduced tooling to detect, analyze and resolve hanging Kafka
transactions. We propose to enhance this tooling to serve some
operationally useful scenarios like reducing the size of response for
`ListTransactionsRequest` and allowing users to build some observability
/ automation around long running transactions.

## [![](https://issues.apache.org/jira/secure/viewavatar?size=xsmall&avatarId=21148&avatarType=issuetype)KAFKA-15546](https://issues.apache.org/jira/browse/KAFKA-15546)

Transactions tool duration field confusing for completed transactions
<u>Open</u>

reported a bug where tooling reports incorrect run duration for
completed transactions. This gives us an opportunity to add information
about the time of last state change which can be useful to analyze stale
transactions.

# Public Interfaces

- **`ListTransactionsRequest`** 

Add a new field, `DurationFilter`  to the `ListTransactionsRequest` and
bump the version to 1

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><code>{</code></p>
<p><code>"apiKey": 66,</code></p>
<p><code>"type": "request",</code></p>
<p><code>"listeners": ["zkBroker", "broker"],</code></p>
<p><code>"name": "ListTransactionsRequest",</code></p>
<p><code>"validVersions": "0-1",</code></p>
<p><code>"flexibleVersions": "0+",</code></p>
<p><code>"fields": [</code></p>
<p><code>{ "name": "StateFilters", "type": "[]string", "versions": "0+",</code></p>
<p><code>"about": "The transaction states to filter by: if empty, all transactions are returned; if non-empty, then only transactions matching one of the filtered states will be returned"</code></p>
<p><code>},</code></p>
<p><code>{ "name": "ProducerIdFilters", "type": "[]int64", "versions": "0+", "entityType": "producerId",</code></p>
<p><code>"about": "The producerIds to filter by: if empty, all transactions will be returned; if non-empty, only transactions which match one of the filtered producerIds will be returned"</code></p>
<p><code>},</code></p>
<p><code>// Add a DurationFilter field</code></p>
<p><code>{ "name": "DurationFilter", "type": "long", "versions": "1+",</code></p>
<p><code>"about": "Return transactions running longer than this time duration, specified in milliseconds"</code></p>
<p><code>}</code></p>
<p><code>]</code></p>
<p><code>}</code></p></td>
</tr>
</tbody>
</table>

Add `DurationFilter`  to the `ListTransactionsOptions`  class used by
`AdminClient`  to pass on filters to the Kafka broker.

<table>
<colgroup>
<col style="width: 101%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><code>@InterfaceStability.Evolving</code></p>
<p><code>public</code> <code>class</code>
<code>ListTransactionsOptions extends</code>
<code>AbstractOptions&lt;ListTransactionsOptions&gt; {</code></p>
<p><code>...</code></p>
<p><code>// return transactions open for more than this time duration specified in milliseconds</code></p>
<p><code>Duration durationFilter;</code></p>
<p><code>public</code>
<code>ListTransactionsOptions durationFilter(Duration timeDuration) {</code></p>
<p><code>this.durationFilter = timeDuration;</code></p>
<p><code>return</code> <code>this;</code></p>
<p><code>}</code></p>
<p><code>public</code> <code>Duration durationFilter() {</code></p>
<p><code>return</code> <code>this.durationFilter;</code></p>
<p><code>}</code></p>
<p><code>...</code></p>
<p><code>}</code></p></td>
</tr>
</tbody>
</table>

- **`ListTransactionsResponse`**

`Version will be bumped to 1 to match request. No changes otherwise.`

- **`DescribeTransactionsResponse`** 

Add a new field,
`TransactionLastUpdateTimeMs to DescribeTransactionsResponse and bump the version to 1`.

Broker will populate this field from `txnLastUpdateTimestamp`  contained
at `TransactionMetadata.` This field is updated at the broker every time
the transaction’s state changes.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><code>{</code></p>
<p><code>"apiKey": 65,</code></p>
<p><code>"type": "response",</code></p>
<p><code>"name": "DescribeTransactionsResponse",</code></p>
<p><code>"validVersions": "0-1",</code></p>
<p><code>"flexibleVersions": "0+",</code></p>
<p><code>"fields": [</code></p>
<p><code>{ "name": "ThrottleTimeMs", "type": "int32", "versions": "0+",</code></p>
<p><code>"about": "The duration in milliseconds for which the request was throttled due to a quota violation, or zero if the request did not violate any quota."</code>
<code>},</code></p>
<p><code>{ "name": "TransactionStates", "type": "[]TransactionState", "versions": "0+", "fields": [</code></p>
<p><code>{ "name": "ErrorCode", "type": "int16", "versions": "0+"</code>
<code>},</code></p>
<p><code>{ "name": "TransactionalId", "type": "string", "versions": "0+", "entityType": "transactionalId"</code>
<code>},</code></p>
<p><code>{ "name": "TransactionState", "type": "string", "versions": "0+"</code>
<code>},</code></p>
<p><code>{ "name": "TransactionTimeoutMs", "type": "int32", "versions": "0+"</code>
<code>},</code></p>
<p><code>{ "name": "TransactionStartTimeMs", "type": "int64", "versions": "0+"</code>
<code>},</code></p>
<p><code>// New field to indicate the timestamp when transaction state was last changed</code></p>
<p><code>{ "name": "TransactionLastUpdateTimeMs", "type": "int64", "versions": "1+"</code>
<code>},</code></p>
<p><code>{ "name": "ProducerId", "type": "int64", "versions": "0+", "entityType": "producerId"</code>
<code>},</code></p>
<p><code>{ "name": "ProducerEpoch", "type": "int16", "versions": "0+"</code>
<code>},</code></p>
<p><code>{ "name": "Topics", "type": "[]TopicData", "versions": "0+",</code></p>
<p><code>"about": "The set of partitions included in the current transaction (if active). When a transaction is preparing to commit or abort, this will include only partitions which do not have markers.",</code></p>
<p><code>"fields": [</code></p>
<p><code>{ "name": "Topic", "type": "string", "versions": "0+", "entityType": "topicName", "mapKey": true</code>
<code>},</code></p>
<p><code>{ "name": "Partitions", "type": "[]int32", "versions": "0+"</code>
<code>}</code></p>
<p><code>]</code></p>
<p><code>}</code></p>
<p><code>]}</code></p>
<p><code>]</code></p>
<p><code>}</code></p></td>
</tr>
</tbody>
</table>

Add t`ransactionLastUpdateTimeMs` to `TransactionDescription`  class.
This object is used to build `APIResult`  at
`org.apache.kafka.clients.admin.internals.DescribeTransactionsHandler#handleResponse` 
from `DescribeTransactionsResponse` .

We will also add an overloaded public constructor to this class to
incorporate the value for `transactionLastUpdateTimeMs`  field.

<table>
<colgroup>
<col style="width: 100%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p><code>package</code>
<code>org.apache.kafka.clients.admin;</code></p>
<p><code>public</code> <code>class</code>
<code>TransactionDescription {</code></p>
<p><code>...</code></p>
<p><code>private</code> <code>final</code>
<code>OptionalLong transactionLastUpdateTimeMs;</code></p>
<p><code>...</code></p>
<p><code>public</code> <code>TransactionDescription(</code></p>
<p><code>int</code> <code>coordinatorId,</code></p>
<p><code>TransactionState state,</code></p>
<p><code>long</code> <code>producerId,</code></p>
<p><code>int</code> <code>producerEpoch,</code></p>
<p><code>long</code> <code>transactionTimeoutMs,</code></p>
<p><code>OptionalLong transactionStartTimeMs,</code></p>
<p><code>Set&lt;TopicPartition&gt; topicPartitions</code></p>
<p><code>) {</code></p>
<p><code>new</code>
<code>TransactionDescription(coordinatorId, state, producerId, producerEpoch, transactionTimeoutMs, transactionStartTimeMs, OptionalLong.empty(), topicPartitions);</code></p>
<p><code>}</code></p>
<p><code>// new overloaded public constructor</code></p>
<p><code>public</code> <code>TransactionDescription(</code></p>
<p><code>int</code> <code>coordinatorId,</code></p>
<p><code>TransactionState state,</code></p>
<p><code>long</code> <code>producerId,</code></p>
<p><code>int</code> <code>producerEpoch,</code></p>
<p><code>long</code> <code>transactionTimeoutMs,</code></p>
<p><code>OptionalLong transactionStartTimeMs,</code></p>
<p><code>OptionalLong transactionLastUpdateTimeMs,</code></p>
<p><code>Set&lt;TopicPartition&gt; topicPartitions</code></p>
<p><code>) {</code></p>
<p><code>...</code></p>
<p><code>this.transactionLastUpdateTimeMs = transactionLastUpdateTimeMs;</code></p>
<p><code>...</code></p>
<p><code>}</code></p>
<p><code>...</code></p>
<p><code>public</code>
<code>OptionalLong transactionLastUpdateTimeMs() {</code></p>
<p><code>return</code> <code>transactionLastUpdateTimeMs;</code></p>
<p><code>}</code></p>
<p><code>...</code></p>
<p><code>}</code></p></td>
</tr>
</tbody>
</table>

<u>`Fixing the TransactionsCommand tool output`</u>

`TransactionDescription`  is further utilized at
`org.apache.kafka.tools.TransactionsCommand.DescribeTransactionsCommand#execute`
to build a printable description of the transaction. This method will be
changed to calculate `transactionDuration`  as a difference between
`transactionStartTime`  and
`transactionLastUpdateTimeMs` `, if state == COMPLETE_COMMIT || state == COMPLETE_ABORT`

- **`DescribeTransactionsRequest`**

`Version will be bumped to 1 to match response. No changes otherwise.`

## Command Line Tool Changes

`kafka-transactions.sh --list`  command will have a new option,
`--durationFilter`  to return transactions running longer than this time
duration, specified in milli seconds.

`kafka-transactions.sh --describe`  command will change the way it
prints the value for transaction duration for completed transactions.

If TransactionLastUpdateTimeMs field is present in
DescribeTransactionsResponse, transaction duration (for completed
transactions) will be printed as the difference of
TransactionLastUpdateTimeMs  and transactionStartTime. On the other
hand, if this field is not present, transaction duration value cannot be
determined correctly (for completed transactions) and we will print -1.

# Proposed Changes

We propose to add a `Duration` filter to `ListTransactionsOptions` in
order to list only the transactions older than a certain time duration.

We also propose to add `lastUpdateTimestamp` to the
`DescribeTransactionsResponse.` This bit can be used to calculate the
correct run duration for a completed transaction and can be helpful in
analyzing stale transactions.

# Compatibility, Deprecation, and Migration Plan

We will bump API version for
`ListTransactionsRequest and DescribeTransactionsResponse from 0 to 1.`

`When  using a new AdminClient to send durationFilter (value greater than 0)  to an older broker, AdminClient will fail to build the  ListTransactionsRequest and throw an UnsupportedVersionException. This  check will be made at ListTransactionsRequest.Builder.build(short  version) method. A new AdminClient can still generate older version of  ListTransactionsRequest when user sets durationFilter to 0 (or does not  set a durationFilter).`

# Rejected Alternatives

An alternative to enhancing these tools is to enable debug logging and
parse through coordinator logs to get information like completion time
and run duration for a transaction. Its better to enhance the tools (fix
in case of `DescribeTransactions` ) to provide a unified and convenient
user experience.

We considered adding `DurationFilter`  as a tagged field to
\`ListTransactionsRequest\` and not bump the API version. This approach
had a down side in terms of usability when a new client is talking to an
older broker. New client can send `durationFilter`  and assume that the
returned transactions are running longer than the specified duration. It
can further try to build follow up actions on these long running
transactions like aborting them. This is dangerous if the broker
supports older version of the API and does not recognize the new field.
Broker will simply return all transactions. Therefore client can not
build automated follow ups on the transactions returned by such
`ListTransactionsRequest` 
