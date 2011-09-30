# eLevelDB

<div id="toc"></div>

## Overview

[eLevelDB](https://github.com/basho/eleveldb) is an Erlang application that
encapsulates [LevelDB](http://code.google.com/p/leveldb/), an open source
on-disk key-value store written by Google Fellows Jeffrey Dean and Sanjay
Ghemawat. LevelDB is a relatively new entrant into the growing list of
key/value database libraries but it has some very interesting qualities that we
believe make it an ideal candidate for use in Riak.  LevelDB’s storage
architecture is more like [BigTable’s](http://en.wikipedia.org/wiki/BigTable)
memtable/sstable model than it is like either Bitcask or InnoDB. LevelDB
includes support for compression of data using another Google open source
project, [Snappy](http://code.google.com/p/snappy/).  This design and
implementation brings the possibility of a storage engine without Bitcask’s RAM
limitation and without any of the drawbacks of InnoDB.

## Installing eLevelDB

Riak ships with eLevelDB included within the distribution, there is no separate
installation required.

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
            {max_open_files, 1000}, %% Maximum number of files open at once
	    ...
]}
```

### Block Size

  Approximate size of user data packed per block.  Note that the block size
  specified here corresponds to uncompressed data.  The actual size of the unit
  read from disk may be smaller if compression is enabled.  For very large
  databases bigger block sizes are likely to perform better, increase the block
  size to 256k (or another power of 2).  Keep in mind that LevelDB's default
  internal block cache is only 8MB so if you increase the block size you will
  want to re-size it setting the `cache_size` option as well.
  
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

    The `cache_size` determines how much data LevelDB caches in memory. The
    more of your dataset that can fit in-memory, the better LevelDB will
    perform.  The LevelDB cache works in conjunction with your operating system
    and filesystem caches, do not disable or undersize them.  If you are
    running a 64-bit Erlang VM, `cache_size` can safely be set above 2G
    assuming you have enough memory available.  Unlike Bitcask LevelDB keeps
    keys and values in a block cache, this allows for management of key spaces
    that are larger than available memory.  Unlike Innostore eLevelDB creates a
    separate LevelDB instance with for each partition of the cluster and so
    each partition will have it's own cache.  For comparison sake, Innostore
    manages all keys and values across all partitions in a single database with
    a single cache.  The cache uses a least-recently-used eviction policy.

    We recommend that you set this to be 60-80% of available RAM (available
    means after subtracting RAM consumed by other services including the
    filesystem cache overhead from physical memory).  For example, on a 12GB
    machine managing a cluster with 64 partitions you might want to divide up
    8GB across the LevelDB's managing each partition.  Set the `cache_size` to
    1/64th of 8GB in bytes (read: `(8 * (1024 ** 3)) / 64`) 134217728 bytes
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
  before the write is considered complete.  If this flag is true, writes will
  be slower but data more durable.

  If this flag is false, and the machine crashes, some recent
  writes may be lost.  Note that if it is just the process that
  crashes (i.e., the machine does not reboot), no writes will be
  lost even if sync==false.

  In other words, a DB write with sync==false has similar
  crash semantics as the "write()" system call.  A DB write
  with sync==true has similar crash semantics to a "write()"
  system call followed by "fsync()".

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

While eLevelDB can be extremely fast for a durable store, its performance can
vary based on how you tune it.  All the configuration is available as
application variables in the `eleveldb` application scope.

### Strengths:

  * Data is compressed

    All data stored into eLevelDB is compressed using the
    [Snappy](http://code.google.com/p/snappy/) compression algorithm.

### Weaknesses:

  * Read access can slow when there are many levels to search

    LevelDB may have to do a few disk seeks to satisfy a read; one disk seek
    per level and, if 10% of the database fits in memory, one seek for the last
    level (since all of the earlier levels should end up cached in the OS
    buffer cache for most filesystems) whereas if 1% fits in memory, leveldb
    will need two seeks.

### Tips & Tricks:

  * __Consider uncompressed data volumes when sizing the cache__

    To avoid repeated decompression of blocks read from disk you should try to
    supply enough cache so that the uncompressed blocks for your working set
    can be held in memory.

  * __Be aware of file handle limits__

    You can control the number of file descriptors eLevelDB will use with
    `max_open_files`.  eLevelDB configuration is set to 20 per partition (which
    is both the default and minimum allowed value) which means that in a
    cluster with 64 partitions you'll have at most 1280 file handles in use at
    a given time.  This can cause problems on some platforms (e.g. OS X has a
    default limit of 256 handles).  The solution is to increase the number of
    file handles available.

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
  * As of Riak 1.0 use of secondary indicies (2i) requires that the bucket
    being indexed be configured to use eLevelDB.

## LevelDB Implementation Details

[LevelDB](http://leveldb.googlecode.com/svn/trunk/doc/impl.html) is a Google
sponsored open source project that has been incorporated into an Erlang
application and integrated into Riakfor storage of key/value information on
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
L >= 1. When the combined size of files in level-L exceeds (10^L) MB (i.e.,
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
level L. In the worst case, this will overlap with ~ 12 files from level L+1
(10 because level-(L+1) is ten times the size of level-L, and another two at
the boundaries since the file ranges at level-L will usually not be aligned
with the file ranges at level-L+1). The compaction will therefore read 26MB and
write 26MB. Assuming a disk IO rate of 100MB/s, the worst compaction cost will
be approximately 0.5 second.

If we throttle the background writing to a resonably slow rate, for instance
10% of the full 100MB/s speed, a compaction may take up to 5 seconds. If the
user is writing at 10MB/s, LevelDB might build up lots of level-0 files (~50 to
hold the 5*10MB). This may signficantly increase the cost of reads due to the
overhead of merging more files together on every read.

### Compaction

Levels are compacted into ordered data files over time.  Compaction first
computes a score for each level as the ratio of bytes in that level to desired
bytes. For level 0, it computes files / desired files instead.  The level with
the highest score is compacted.

When compacting L0 the only special case to consider is that after picking the
primary L0 file to compact, it will check other L0 files for to determine the
degree to which they overlap. This is an attempt to avoid some I/O, we can
expect L0 compactions to usually if not always be "all L0 files".

See the PickCompaction routine in
[1](http://www.google.com/codesearch#mHLldehqYMA/trunk/db/version_set.cc) for
all the details.

### Compression

LevelDB uses Snappy to compress data on the fly.  Snappy gives lightweight but
fast compression.  Typical speeds of the Snappy compression algorithm on an
Intel(R) Core(TM)2 2.4GHz:
 * ~200-500MB/s compression
 * ~400-800MB/s decompression
Note that these speeds are significantly faster than most persistent storage
(disk drives and SSDs) speeds, and therefore it is typically worth the CPU time
to compress data.  If the input data is incompressible, the Snappy compression
implementation will efficiently detect that and will switch to uncompressed
mode.

### Comparison of eLevelDB and Bitcask

LevelDB is a persistent ordered map; Bitcask is a persistent hash table (no
ordered iteration).  Bitcask stores keys in memory, so for databases with large
number of keys it may exhaust available physical memory and then swap into
virtual memory causing a severe slow down in performance.  Bitcask guarantees
at most one disk seek per lookup.  LevelDB may have to do a small number of
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
partially written log record (or a few partially written log records). Leveldb
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
|-- 1050454301831586472458898473514828420377701515264
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1073290264914881830555831049026020342559825461248
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1096126227998177188652763624537212264741949407232
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1118962191081472546749696200048404186924073353216
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 114179815416476790484662877555959610910619729920
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1141798154164767904846628775559596109106197299200
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1164634117248063262943561351070788031288321245184
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1187470080331358621040493926581979953470445191168
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1210306043414653979137426502093171875652569137152
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1233142006497949337234359077604363797834693083136
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1255977969581244695331291653115555720016817029120
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1278813932664540053428224228626747642198940975104
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1301649895747835411525156804137939564381064921088
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1324485858831130769622089379649131486563188867072
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1347321821914426127719021955160323408745312813056
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 137015778499772148581595453067151533092743675904
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1370157784997721485815954530671515330927436759040
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1392993748081016843912887106182707253109560705024
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1415829711164312202009819681693899175291684651008
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 1438665674247607560106752257205091097473808596992
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 159851741583067506678528028578343455274867621888
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 182687704666362864775460604089535377456991567872
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 205523667749658222872393179600727299639115513856
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 22835963083295358096932575511191922182123945984
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 228359630832953580969325755111919221821239459840
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 251195593916248939066258330623111144003363405824
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 274031556999544297163190906134303066185487351808
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 296867520082839655260123481645494988367611297792
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 319703483166135013357056057156686910549735243776
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 342539446249430371453988632667878832731859189760
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 365375409332725729550921208179070754913983135744
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 388211372416021087647853783690262677096107081728
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 411047335499316445744786359201454599278231027712
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 433883298582611803841718934712646521460354973696
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 45671926166590716193865151022383844364247891968
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 456719261665907161938651510223838443642478919680
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 479555224749202520035584085735030365824602865664
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 502391187832497878132516661246222288006726811648
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 525227150915793236229449236757414210188850757632
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 548063113999088594326381812268606132370974703616
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 570899077082383952423314387779798054553098649600
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 593735040165679310520246963290989976735222595584
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 616571003248974668617179538802181898917346541568
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 639406966332270026714112114313373821099470487552
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 662242929415565384811044689824565743281594433536
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 68507889249886074290797726533575766546371837952
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 685078892498860742907977265335757665463718379520
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 707914855582156101004909840846949587645842325504
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 730750818665451459101842416358141509827966271488
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 753586781748746817198774991869333432010090217472
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 776422744832042175295707567380525354192214163456
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 799258707915337533392640142891717276374338109440
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 822094670998632891489572718402909198556462055424
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 844930634081928249586505293914101120738586001408
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 867766597165223607683437869425293042920709947392
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 890602560248518965780370444936484965102833893376
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 91343852333181432387730302044767688728495783936
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 913438523331814323877303020447676887284957839360
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 936274486415109681974235595958868809467081785344
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
|-- 959110449498405040071168171470060731649205731328
|   |-- 000005.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000004
`-- 981946412581700398168100746981252653831329677312
    |-- 000005.log
    |-- CURRENT
    |-- LOCK
    |-- LOG
    |-- LOG.old
    `-- MANIFEST-000004

64 directories, 378 files
```

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
|-- 1050454301831586472458898473514828420377701515264
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1073290264914881830555831049026020342559825461248
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1096126227998177188652763624537212264741949407232
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
|-- 1118962191081472546749696200048404186924073353216
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 114179815416476790484662877555959610910619729920
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1141798154164767904846628775559596109106197299200
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1164634117248063262943561351070788031288321245184
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1187470080331358621040493926581979953470445191168
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
|-- 1210306043414653979137426502093171875652569137152
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1233142006497949337234359077604363797834693083136
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1255977969581244695331291653115555720016817029120
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1278813932664540053428224228626747642198940975104
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
|-- 1301649895747835411525156804137939564381064921088
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1324485858831130769622089379649131486563188867072
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1347321821914426127719021955160323408745312813056
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 137015778499772148581595453067151533092743675904
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1370157784997721485815954530671515330927436759040
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
|-- 1392993748081016843912887106182707253109560705024
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1415829711164312202009819681693899175291684651008
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1438665674247607560106752257205091097473808596992
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 159851741583067506678528028578343455274867621888
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 182687704666362864775460604089535377456991567872
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
|-- 205523667749658222872393179600727299639115513856
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 22835963083295358096932575511191922182123945984
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 228359630832953580969325755111919221821239459840
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 251195593916248939066258330623111144003363405824
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 274031556999544297163190906134303066185487351808
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
|-- 296867520082839655260123481645494988367611297792
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 319703483166135013357056057156686910549735243776
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 342539446249430371453988632667878832731859189760
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 365375409332725729550921208179070754913983135744
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
|-- 388211372416021087647853783690262677096107081728
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 411047335499316445744786359201454599278231027712
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 433883298582611803841718934712646521460354973696
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 45671926166590716193865151022383844364247891968
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 456719261665907161938651510223838443642478919680
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
|-- 479555224749202520035584085735030365824602865664
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 502391187832497878132516661246222288006726811648
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 525227150915793236229449236757414210188850757632
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 548063113999088594326381812268606132370974703616
|   |-- 000121.sst
|   |-- 000122.sst
|   |-- 000123.sst
|   |-- 000124.sst
|   |-- 000125.sst
|   |-- 000127.sst
|   |-- 000130.sst
|   |-- 000131.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000129
|-- 570899077082383952423314387779798054553098649600
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 593735040165679310520246963290989976735222595584
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 616571003248974668617179538802181898917346541568
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 639406966332270026714112114313373821099470487552
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
|-- 662242929415565384811044689824565743281594433536
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 68507889249886074290797726533575766546371837952
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 685078892498860742907977265335757665463718379520
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 707914855582156101004909840846949587645842325504
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 730750818665451459101842416358141509827966271488
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
|-- 753586781748746817198774991869333432010090217472
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 776422744832042175295707567380525354192214163456
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 799258707915337533392640142891717276374338109440
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 822094670998632891489572718402909198556462055424
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
|-- 844930634081928249586505293914101120738586001408
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 867766597165223607683437869425293042920709947392
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 890602560248518965780370444936484965102833893376
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 91343852333181432387730302044767688728495783936
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
|-- 913438523331814323877303020447676887284957839360
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
|-- 936274486415109681974235595958868809467081785344
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 959110449498405040071168171470060731649205731328
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
`-- 981946412581700398168100746981252653831329677312
    |-- 000003.log
    |-- CURRENT
    |-- LOCK
    |-- LOG
    `-- MANIFEST-000002

64 directories, 433 files
```

```bash
...
|-- 0
|   |-- 000003.log
|   |-- 000120.sst
|   |-- 000121.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1004782375664995756265033322492444576013453623296
...
```


```bash
leveldb/
|-- 0
|   |-- 000134.sst
|   |-- 000135.sst
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.log
|   |-- 000140.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000131
|-- 1004782375664995756265033322492444576013453623296
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.sst
|   |-- 000140.sst
|   |-- 000141.log
|   |-- 000142.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000133
|-- 1027618338748291114361965898003636498195577569280
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1050454301831586472458898473514828420377701515264
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1073290264914881830555831049026020342559825461248
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1096126227998177188652763624537212264741949407232
|   |-- 000134.sst
|   |-- 000135.sst
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.log
|   |-- 000140.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000131
|-- 1118962191081472546749696200048404186924073353216
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 114179815416476790484662877555959610910619729920
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1141798154164767904846628775559596109106197299200
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1164634117248063262943561351070788031288321245184
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1187470080331358621040493926581979953470445191168
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.sst
|   |-- 000140.sst
|   |-- 000141.log
|   |-- 000142.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000133
|-- 1210306043414653979137426502093171875652569137152
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1233142006497949337234359077604363797834693083136
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1255977969581244695331291653115555720016817029120
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1278813932664540053428224228626747642198940975104
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.sst
|   |-- 000140.sst
|   |-- 000141.log
|   |-- 000142.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000133
|-- 1301649895747835411525156804137939564381064921088
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1324485858831130769622089379649131486563188867072
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1347321821914426127719021955160323408745312813056
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 137015778499772148581595453067151533092743675904
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1370157784997721485815954530671515330927436759040
|   |-- 000134.sst
|   |-- 000135.sst
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.log
|   |-- 000140.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000131
|-- 1392993748081016843912887106182707253109560705024
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1415829711164312202009819681693899175291684651008
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 1438665674247607560106752257205091097473808596992
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 159851741583067506678528028578343455274867621888
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 182687704666362864775460604089535377456991567872
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.sst
|   |-- 000140.sst
|   |-- 000141.log
|   |-- 000142.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000133
|-- 205523667749658222872393179600727299639115513856
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 22835963083295358096932575511191922182123945984
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 228359630832953580969325755111919221821239459840
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 251195593916248939066258330623111144003363405824
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 274031556999544297163190906134303066185487351808
|   |-- 000134.sst
|   |-- 000135.sst
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.log
|   |-- 000140.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000131
|-- 296867520082839655260123481645494988367611297792
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 319703483166135013357056057156686910549735243776
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 342539446249430371453988632667878832731859189760
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 365375409332725729550921208179070754913983135744
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.sst
|   |-- 000140.sst
|   |-- 000141.log
|   |-- 000142.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000133
|-- 388211372416021087647853783690262677096107081728
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 411047335499316445744786359201454599278231027712
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 433883298582611803841718934712646521460354973696
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 45671926166590716193865151022383844364247891968
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 456719261665907161938651510223838443642478919680
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.sst
|   |-- 000140.sst
|   |-- 000141.log
|   |-- 000142.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000133
|-- 479555224749202520035584085735030365824602865664
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 502391187832497878132516661246222288006726811648
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 525227150915793236229449236757414210188850757632
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 548063113999088594326381812268606132370974703616
|   |-- 000125.sst
|   |-- 000138.sst
|   |-- 000139.sst
|   |-- 000140.sst
|   |-- 000141.sst
|   |-- 000142.sst
|   |-- 000143.log
|   |-- 000144.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000135
|-- 570899077082383952423314387779798054553098649600
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 593735040165679310520246963290989976735222595584
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 616571003248974668617179538802181898917346541568
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 639406966332270026714112114313373821099470487552
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.sst
|   |-- 000140.sst
|   |-- 000141.log
|   |-- 000142.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000133
|-- 662242929415565384811044689824565743281594433536
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 68507889249886074290797726533575766546371837952
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 685078892498860742907977265335757665463718379520
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 707914855582156101004909840846949587645842325504
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 730750818665451459101842416358141509827966271488
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.sst
|   |-- 000140.sst
|   |-- 000141.log
|   |-- 000142.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000133
|-- 753586781748746817198774991869333432010090217472
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 776422744832042175295707567380525354192214163456
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 799258707915337533392640142891717276374338109440
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 822094670998632891489572718402909198556462055424
|   |-- 000134.sst
|   |-- 000135.sst
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.log
|   |-- 000140.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000131
|-- 844930634081928249586505293914101120738586001408
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 867766597165223607683437869425293042920709947392
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 890602560248518965780370444936484965102833893376
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 91343852333181432387730302044767688728495783936
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.sst
|   |-- 000140.sst
|   |-- 000142.sst
|   |-- 000143.log
|   |-- 000144.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000133
|-- 913438523331814323877303020447676887284957839360
|   |-- 000136.sst
|   |-- 000137.sst
|   |-- 000138.sst
|   |-- 000139.sst
|   |-- 000140.sst
|   |-- 000141.log
|   |-- 000142.sst
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   |-- LOG.old
|   `-- MANIFEST-000133
|-- 936274486415109681974235595958868809467081785344
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
|-- 959110449498405040071168171470060731649205731328
|   |-- 000003.log
|   |-- CURRENT
|   |-- LOCK
|   |-- LOG
|   `-- MANIFEST-000002
`-- 981946412581700398168100746981252653831329677312
    |-- 000003.log
    |-- CURRENT
    |-- LOCK
    |-- LOG
    `-- MANIFEST-000002

64 directories, 434 files
```

## References

* [LevelDB documentation](http://leveldb.googlecode.com/svn/trunk/doc/index.html)
* [Cache Oblivious BTree](http://supertech.csail.mit.edu/cacheObliviousBTree.html)
* [LevelDB benchmarks](http://leveldb.googlecode.com/svn/trunk/doc/benchmark.html) run by Goolge
* [LevelDB](http://en.wikipedia.org/wiki/LevelDB) information on Wikipedia
* [LSM trees](http://nosqlsummer.org/paper/lsm-tree)
* [Cache Conscious Indexing for Decision-Support in Main Memory](http://www.cs.columbia.edu/~library/TR-repository/reports/.../cucs-019-98.pdf)