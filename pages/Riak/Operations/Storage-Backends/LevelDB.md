# LevelDB

<div id="toc"></div>

## Overview

[eLevelDB](https://github.com/basho/eleveldb) is an Erlang application that
encapsulates [LevelDB](http://code.google.com/p/leveldb/), an open source
on-disk key-value store written by Google Fellows Jeffrey Dean and Sanjay
Ghemawat. LevelDB is a relatively new entrant into the growing list of
key/value database libraries but it has some very interesting qualities that we
believe make it an ideal candidate for use in Riak.  LevelDB’s storage
architecture is more like [BigTable’s](http://en.wikipedia.org/wiki/BigTable)
memtable/sstable model than it is like either Bitcask or InnoDB.  This design
and implementation brings the possibility of a storage engine without Bitcask’s
RAM limitation and without any of the drawbacks of InnoDB.

### Strengths:

  * License

    The LevelDB and eLevelDB licenses are the [New BSD
    License](http://www.opensource.org/licenses/bsd-license.php) and the
    [Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0.html).
    We'd like to thank Google and the authors of LevelDB at Google for choosing
    a completely F/LOSS license so that everyone can benefit from this
    innovative storage engine.  Due to this commercial users of eLevelDB can
    choose eLevelDB over the Innostore backend if they consider the GPL to be
    an issue.

### Weaknesses:

  * Read access can slow when there are many levels to search

    LevelDB may have to do a few disk seeks to satisfy a read; one disk seek
    per level and, if 10% of the database fits in memory, one seek for the last
    level (since all of the earlier levels should end up cached in the OS
    buffer cache for most file systems) whereas if 1% fits in memory, LevelDB
    will need two seeks.

## Installing eLevelDB

Riak ships with eLevelDB included within the distribution, so there is no
separate installation required.

The default configuration values found in your `app.config` for eLevelDB are as
follows:
```erlang
 %% LevelDB Config
 {eleveldb, [
             {data_root, "/var/lib/riak/leveldb"}
           ]},
```

## Configuring eLevelDB

Modify the default behavior by adding these settings into the `eleveldb` section
in your [app.config](Configuration Files).

### Write Buffer Size

  Amount of data to build up in memory (backed by an unsorted log on disk)
  before converting to a sorted on-disk file.

  Larger values increase performance, especially during bulk loads.  Up to two
  write buffers may be held in memory at the same time, so you may wish to
  adjust this parameter to control memory usage.  Also, a larger write buffer
  will result in a longer recovery time the next time the database is opened.
 
  Default is: 4MB

```erlang
{eleveldb, [
	    ...,
            {write_buffer_size, 4194304}, %% 4MB in bytes
	    ...
]}
```

### Max Open Files

  Number of open files that can be used by the DB.  You may need to increase
  this if your database has a large working set (budget one open file per 2MB
  of working set divided by `ring_creation_size`).
  
  Default: 20

  Minimum: 20

```erlang
{eleveldb, [
	    ...,
            {max_open_files, 20}, %% Maximum number of files open at once per partition
	    ...
]}
```

<div class="note"><div class="title">Check your system's open files limits</div>
<p>Due to the large number of open files used by this storage engine is it
imperitive that you review and properly set your system's open files limits.  If
you are seeing an error that contains `emfile` then it is highly likely that
you've exceeded the limits on your system for open files, read more about this
later in the Tips & Tricks section to see how to fix this issue.</p>
</div>

### Block Size

  Approximate size of user data packed per block. For very large databases
  bigger block sizes are likely to perform better so increasing the block size
  to 256k (or another power of 2) may be a good idea.  Keep in mind that
  LevelDB's default internal block cache is only 8MB so if you increase the
  block size you will want to resize `cache_size` as well.
  
  Default: 4K

```erlang
{eleveldb, [
	    ...,
            {block_size, 4096}, %% 4K blocks
	    ...
]}
```

### Block Restart Interval

  Number of keys between restart points for delta encoding of keys.
  Most clients should leave this parameter alone.

  Default: 16

```erlang
{eleveldb, [
	    ...,
            {block_restart_interval, 16}, %% # of keys before restarting delta encoding
	    ...
]}
```

### Cache Size

   The `cache_size` determines how much data LevelDB caches in memory. The more
   of your data set that can fit in-memory, the better LevelDB will perform.
   The LevelDB cache works in conjunction with your operating system and file
   system caches, do not disable or under-size them.  If you are running a
   64-bit Erlang VM, `cache_size` can safely be set above 2G assuming you have
   enough memory available.  Unlike Bitcask, LevelDB keeps keys and values in a
   block cache, this allows for management of key spaces that are larger than
   available memory.  Unlike Innostore, eLevelDB creates a separate LevelDB
   instance for each partition of the cluster and so each partition will have
   it's own cache.  For comparison sake, Innostore manages all keys and values
   across all partitions in a single database with a single cache.  The cache
   uses a least-recently-used eviction policy.

   We recommend that you set this to be 60-80% of available RAM (available
   means after subtracting RAM consumed by other services including the
   file system cache overhead from physical memory).  For example, on a 12GB
   machine managing a cluster with 64 partitions you might want to divide up
   8GB across the LevelDB's managing each partition.  Set the `cache_size` to
   1/64th of 8GB in bytes (read: `(8 * (1024 ^ 3)) / 64`) 134217728 bytes
   (aka 128 MB).

   Default: 8MB

```erlang
{eleveldb, [
	    ...,
            {cache_size, 8388608}, %% 8MB default cache size per-partition
	    ...
]}
```

### Sync

  If true, the write will be flushed from the operating system buffer cache
  before the write is considered complete.  Writes will be slower but data more
  durable.

  If this flag is false, and the machine crashes, some recent writes may be
  lost.  Note that if it is just the process that crashes (i.e., the machine
  does not reboot), no writes will be lost even if sync is set to false.

  In other words, a DB write with sync is false has similar crash semantics as
  the "write()" system call.  A DB write with when sync is true has similar
  crash semantics to a "write()" system call followed by "fsync()".

  One other consideration is that the hard disk itself may be buffering the
  write in its memory (a write ahead cache) and responding before the data has
  been written to the platter. This may or may not be safe based on whether or
  not the hard disk has enough power to save its memory in the event of a power
  failure.  If data durability is absolutely necessary then make sure you
  disable this feature on your drive or provide battery backup and a proper
  shutdown procedure during power loss.

  Default: false


```erlang
{eleveldb, [
	    ...,
            {sync, true}, %% do the write()/fsync() every time
	    ...
]}
```

### Verify Checksums

  If true, all data read from underlying storage will be
  verified against corresponding checksums.

  Default: false

```erlang
{eleveldb, [
	    ...,
            {verify_checksums, true}, %% make sure data is what we expected it to be
	    ...
]}
```

## Tuning LevelDB

While eLevelDB can be extremely fast for a durable store, its performance
varies based on how you tune it.  All the configuration is exposed via
application variables in the `eleveldb` application scope.

### Tips & Tricks:

  * __Be aware of file handle limits__

    You can control the number of file descriptors eLevelDB will use with
    `max_open_files`.  eLevelDB configuration is set to 20 per partition (which
    is both the default and minimum allowed value) which means that in a
    cluster with 64 partitions you'll have at most 1280 file handles in use at
    a given time.  This can cause problems on some platforms (e.g. OS X has a
    default limit of 256 handles).  The solution is to increase the number of
    file handles available.  Review the (open files
    limitations)(Open-Files-Limit) information.

  * __Avoid extra disk head seeks by turning off `noatime`__

    eLevelDB is very aggressive at reading and writing files.  As such, you
    can get a big speed boost by adding the `noatime` mounting option to
    `/etc/fstab`.  This will disable the recording of the "last accessed time"
    for all files.  If you need last access times but you'd like some of the
    benefits of this optimization you can try `relatime`.

```bash
/dev/sda5    /data           ext3    noatime  1 1
/dev/sdb1    /data/inno-log  ext3    noatime  1 2
```

## FAQ

  * [Has Basho run any performance benchmarks comparing LevelDB and InnoDB?](http://blog.basho.com/2011/07/01/Leveling-the-Field/)
  * As of Riak 1.0 use of secondary indexes (2I) requires that the bucket
    being indexed be configured to use eLevelDB.

## LevelDB Implementation Details

[LevelDB](http://leveldb.googlecode.com/svn/trunk/doc/impl.html) is a Google
sponsored open source project that has been incorporated into an Erlang
application and integrated into Riak for storage of key/value information on
disk. The implementation of LevelDB is similar in spirit to the representation
of a single Bigtable tablet (section 5.3).

### How "Levels" Are Managed

LevelDB is a memtable/sstable design. The set of sorted tables are organized
into a sequence of levels. Each level stores approximately ten times as much
data as the level before it.  The sorted table generated from a flush is placed
in a special young level (also called level-0). When the number of young files
exceeds a certain threshold (currently four), all of the young files are merged
together with all of the overlapping level-1 files to produce a sequence of new
level-1 files (a new level-1 file is created for every 2MB of data.)

Files in the young level may contain overlapping keys. However files in other
levels have distinct non-overlapping key ranges. Consider level number L where
L >= 1. When the combined size of files in level-L exceeds (10^L) MB (i.e.
10MB for level-1, 100MB for level-2, ...), one file in level-L, and all of the
overlapping files in level-(L+1) are merged to form a set of new files for
level-(L+1). These merges have the effect of gradually migrating new updates
from the young level to the largest level using only bulk reads and writes
(i.e., minimizing expensive disk seeks).

When the size of level L exceeds its limit, LevelDB will compact it in a
background thread. The compaction picks a file from level L and all overlapping
files from the next level L+1. Note that if a level-L file overlaps only part
of a level-(L+1) file, the entire file at level-(L+1) is used as an input to
the compaction and will be discarded after the compaction. Compactions from
level-0 to level-1 are treated specially because level-0 is special (files in
it may overlap each other).  A level-0 compaction may pick more than one
level-0 file in case some of these files overlap each other.

A compaction merges the contents of the picked files to produce a sequence of
level-(L+1) files. LevelDB will switch to producing a new level-(L+1) file
after the current output file has reached the target file size (2MB). LevelDB
will also switch to a new output file when the key range of the current output
file has grown enough to overlap more then ten level-(L+2) files. This last
rule ensures that a later compaction of a level-(L+1) file will not pick up too
much data from level-(L+2).

Compactions for a particular level rotate through the key space. In more
detail, for each level L, LevelDB remembers the ending key of the last
compaction at level L. The next compaction for level L will pick the first file
that starts after this key (wrapping around to the beginning of the key space
if there is no such file).

Level-0 compactions will read up to four 1MB files from level-0, and at worst
all the level-1 files (10MB) (i.e., LevelDB will read 14MB and write 14MB in
that case).

Other than the special level-0 compactions, LevelDB will pick one 2MB file from
level L. In the worst case, this will overlap with approximately 12 files from
level L+1 (10 because level-(L+1) is ten times the size of level-L, and another
two at the boundaries since the file ranges at level-L will usually not be
aligned with the file ranges at level-L+1). The compaction will therefore read
26MB, write 26MB. Assuming a disk IO rate of 100MB/s, the worst compaction cost
will be approximately 0.5 second.

If we throttle the background writing to a reasonably slow rate, for instance
10% of the full 100MB/s speed, a compaction may take up to 5 seconds. If the
user is writing at 10MB/s, LevelDB might build up lots of level-0 files (~50 to
hold the 5*10MB). This may significantly increase the cost of reads due to the
overhead of merging more files together on every read.

### Compaction

Levels are compacted into ordered data files over time.  Compaction first
computes a score for each level as the ratio of bytes in that level to desired
bytes. For level 0, it computes files / desired files instead.  The level with
the highest score is compacted.

When compacting L0 the only special case to consider is that after picking the
primary L0 file to compact, it will check other L0 files to determine the
degree to which they overlap. This is an attempt to avoid some I/O, we can
expect L0 compactions to usually if not always be "all L0 files".

See the PickCompaction routine in
[1](http://www.google.com/codesearch#mHLldehqYMA/trunk/db/version_set.cc) for
all the details.

### Comparison of eLevelDB and Bitcask

LevelDB is a persistent ordered map; Bitcask is a persistent hash table (no
ordered iteration).  Bitcask stores keys in memory, so for databases with large
number of keys it may exhaust available physical memory and then swap into
virtual memory causing a severe slow down in performance.  Bitcask guarantees
at most one disk seek per look-up.  LevelDB may have to do a small number of
disk seeks.  For instance, a read needs one disk seek per level. If 10% of
the database fits in memory, LevelDB will need to do one seek (for the last
level since all of the earlier levels should end up cached in the OS buffer
cache). If 1% fits in memory, LevelDB will need two seeks.

### Comparison of eLevelDB/LevelDB and Innostore/InnoDB

Fundamentally the major differences fall out of the different operational
characteristics of btrees and LSM tables.  InnoDB uses btrees for ordered
storage and LevelDB uses log structured merge trees with some cache oblivious
features as well.  Random write performance (which is generally the case with
data storage in Riak) is significantly better in LevelDB.  Random read can
suffer if there are many levels to explore before finding the data.

## Recovery

LevelDB never writes in place: it always appends to a log file, or merges
existing files together to produce new ones. So an OS crash will cause a
partially written log record (or a few partially written log records). LevelDB
recovery code uses checksums to detect this and will skip the incomplete
records.

### eLevelDB Database Files

Below there are two directory listings showing what you would expect to find on
disk when using eLevelDB.  In this example we use a 64 partition ring which
results in 64 separate directories, each with their own LevelDB database.

```bash
leveldb/
|-- 0
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1004782375664995756265033322492444576013453623296
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1027618338748291114361965898003636498195577569280
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004

... etc ...

`-- 981946412581700398168100746981252653831329677312
    |-- 000005.log
    |-- CURRENT
    |-- LOCK
    |-- LOG
    |-- LOG.old
    `-- MANIFEST-000004

64 directories, 378 files
```

After performing a large number "put" (write) operations the Riak cluster
running eLevelDB will look something like this.

```bash
gburd@toe:~/Projects/riak/dev/dev1/data$ tree leveldb/
leveldb/
|-- 0
|   |-- 000118.sst
|   |-- 000119.sst
|   |-- 000120.sst
|   |-- 000121.sst
|   |-- 000123.sst
|   |-- 000126.sst
|   |-- 000127.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000125
|-- 1004782375664995756265033322492444576013453623296
|   |-- 000120.sst
|   |-- 000121.sst
|   |-- 000122.sst
|   |-- 000123.sst
|   |-- 000125.sst
|   |-- 000128.sst
|   |-- 000129.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000127
|-- 1027618338748291114361965898003636498195577569280
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002

... etc ...

`-- 981946412581700398168100746981252653831329677312
    |-- 000003.log
    |-- CURRENT
    |-- LOCK
    |-- LOG
    `-- MANIFEST-000002

64 directories, 433 files
```

## References

* [LevelDB documentation](http://leveldb.googlecode.com/svn/trunk/doc/index.html)
* [Cache Oblivious BTree](http://supertech.csail.mit.edu/cacheObliviousBTree.html)
* [LevelDB benchmarks](http://leveldb.googlecode.com/svn/trunk/doc/benchmark.html) run by Goolge
* [LevelDB](http://en.wikipedia.org/wiki/LevelDB) information on Wikipedia
* [LSM trees](http://nosqlsummer.org/paper/lsm-tree)
* [Cache Conscious Indexing for Decision-Support in Main Memory](http://www.cs.columbia.edu/~library/TR-repository/reports/.../cucs-019-98.pdf)