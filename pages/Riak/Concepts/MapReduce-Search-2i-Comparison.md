# Comparing MapReduce, RiakSearch, and Secondary Indexes

Riak supports a variety of data retrieval methods in addition to basic GET
operations. Each of these methods provide different abilities and tradeoffs. As
such, it is important to understand the strength and limitations of each, so you
can pick the right tool for the job.

## MapReduce

[[MapReduce]] is used to perform ad-hoc queries and transformations on known
sets of key. While full bucket MapReduce is supported, it's expensive and should
be used sparingly in production environments. It should be noted that
[[Link-walking|Links#Link-walking]] is powered by MapReduce.

## Riak Search

[[Riak Search]] is used to perform SOLR style queries. This means that a rich
query syntax is available for free-text queries. Riak Search is primarily meant
as a way to index prosaic data along with its relevant metadata.

## Secondary Indexes

[[Secondary Indexes]] are a new feature supported in Riak 1.0 that allow users
to create indexes by adding metadata to Riak objects. They are best used to
allow easy retrieval of objects tags. With Secondary Indexes, it's used to model
one-to-many relationships. For example, a blog post could be tagged with all of
it categories, and be then retrieved by those categories.

## Comparison

<table>
    <tr>
        <th>&nbsp;</th>
        <th>MapReduce</th>
        <th>Riak Search</th>
        <th>Secondary Indexes</th>
    </tr>
    <tr>
        <td>Query Support</td>
        <td>Ad-hoc queries composed of an arbitrary number Map phases and Reduce
            phases</td>
        <td>SOLR style queries supporting free-text, wildcards, and boolean
            operators</td>
        <td>Equality and range query support</td>
    </tr>
    <tr>
        <td>Coverage Query</td>
        <td><!-- Need info --></td>
        <td>1 search vnode per a term queried</td>
        <td>1/N of all KV vnodes per a request</td>
    </tr>
    <tr>
        <td>Supported Data Types</td>
        <td>Any datatype with Erlang MapReduce functions, valid UTF-8 JSON with
            Javascript functions. [[Links]] per the specification</td>
        <td>Integer, Date, and Text</td>
        <td>Binary and Integer</td>
    </tr>
    <tr>
        <td>Extraction</td>
        <td>Map phases can be used to extract data for later Map and Reduce
            phases</td>
        <td>Either one of the provided analyzers (Whitespace, Standard, Integer,
            and No-Op) or a Custom Analyzer</td>
        <td>Indexed values are submitted as metadata on the objeect.</td>
    </tr>
    <tr>
        <td>Anti-Entropy</td>
        <td>N/A</td>
        <td>No anti-entropy features. If a search partition is lost, the entire
            search index should be rebuilt</td>
        <td>Anti-entropy is carried over from kv; if a partition is lost,
            secondary indexes will be rebuilt along side the kv data</td>
    </tr>
    <tr>
        <td>Limitations</td>
        <td>MapReduce operations are performed in memory</td>
        <td>A covering query could hit all nodes when many of terms are
            used</td>
        <td>Only available with the LevelDB backend</td>
    </tr>
    <tr>
        <td>Good Use Cases</td>
        <td>Performing calculations based on a known set of bucket-key pairs</td>
        <td>Searching objects with full-text data</td>
        <td>Retrieving all objects tagged with a particular term</td>
    </tr>
    <tr>
        <td>Poor Use Cases</td>
        <td>Analyzing every object in a bucket</td>
        <td>Searching for common (low cardinality) terms in documents</td>
        <td>Searching prosiac text</td>
    </tr>
</table>