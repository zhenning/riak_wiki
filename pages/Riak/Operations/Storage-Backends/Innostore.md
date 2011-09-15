# Innostore

<div id="toc"/></div>

## Overview
[Innostore](https://github.com/basho/innostore) is an Erlang application that provides an API for storing and retrieving key/value data using the [InnoDB](http://www.innodb.com/products/innodb/) storage system. This storage system is the same one used by [MySQL](http://www.mysql.com) for reliable, transactional data storage. It’s a proven, fast system and perfect for use with Riak if you have a large amount of data to store. Let’s take a look at how you can use Innostore as a backend for Riak.

## Installing Innostore

Unlike the other storage engines, Innostore is distributed separately from Riak.
We recommend using a pre-packaged binary distribution but you can also build it
from source code.

### Installing from pre-built packages

1. Determine your system and architecture (such as "Ubuntu" and "64bit Intel") and
   then find the latest download package.
   [http://downloads.basho.com/](http://downloads.basho.com/innostore/CURRENT)
   Use your browser or a tool such as `wget` or `curl` to download the package
   and then install the package using the proper package manager.

```bash
# Example: Ubuntu x64 or other Debian-based Linux distributions:
$ wget http://downloads.basho.com/innostore/innostore-1.0.3/innostore_1.0.3-2_amd64.deb
$ sudo dpkg -i innostore_1.0.3-2_amd64.deb

# Example: RedHat x64 or other RedHat-based Linux distributions:
$ wget http://downloads.basho.com/innostore/innostore-1.0.3/innostore-1.0.3-2.el5.x86_64.rpm
$ sudo rpm -Uvh innostore-1.0.3-2.el5.x86_64.rpm

# Example: Fedora Core x64 and related Linux distributions:
$ wget http://downloads.basho.com/innostore/innostore-1.0.3/innostore-1.0.3-2.fc12.x86_64.rpm
$ sudo rpm -Uvh innostore-1.0.3-2.fc12.x86_64.rpm

# etc...
```

### Building and installing from source code

1. You will need to have [Erlang installed](Installing Erlang) to
   compile Innostore.

2. Obtain the source code.
   - You can either obtain the source code [as a package|http://downloads.basho.com/innostore/innostore-1.0.3/innostore-1.0.3.tar.gz].
```bash
# Example: downloading the code
$ wget http://downloads.basho.com/innostore/innostore-1.0.3/innostore-1.0.3.tar.gz
$ tar -xzf innostore-1.0.3.tar.gz
$ cd innostore-1.0.3
```
   - or directly from the source code repository [http://github.com/basho/innostore].
```bash
# Example: clone the Git repository
$ git clone http://github.com/basho/innostore
$ cd innostore
$ git checkout innostore-1.0.3
```

3. Now that you have the code, let's build it:
```bash
# NOTE: on a Singe CPU system you'll need to enable Erlang to run SMP before building
$ export ERL_FLAGS="-smp enable"
$ make
```

4. Install innostore
If your compile passed all the tests you are now ready to install Innostore into your Riak distribution.

   - Option 1: into your Erlang distribution:
   If you are using a copy of Riak you compiled yourself you can install Innostore by issuing the following command replacing $RIAK with the location of your Riak install:
   ```bash
   $ make install
   ```

   - Option 2: into an existing Riak build:
   If you are using a copy of Riak you compiled yourself you can install Innostore by issuing the following command replacing $RIAK with the location of your Riak install:
   ```bash
   $ ./rebar install target=$RIAK/lib
   ```

## Configuring Riak to use Innostore

There are two steps to configure Riak to use Innostore.

1. Edit your Riak installation's `app.config` file
   Change the the `storage_backend` setting to `riak_kv_innostore_backend`.
```erlang
{storage_backend, riak_kv_innostore_backend}
```

2. While the defaults should work just fine you may want to modify the
   location of the Innostore database files.  To do that you first add an
   `innostore` application section to the `riak/etc/app.config` file.
   Modify your `data_home_dir` and `log_group_home_dir` paths as needed.
```erlang
{innostore, [
    {data_home_dir, "/var/lib/riak/innodb"}, %% Where data files go
    {log_group_home_dir, "/var/lib/riak/innodb"}, %% Where log files go
    {buffer_pool_size, 2147483648} %% 2GB of buffer
]}
```

3. Now that you have configured Innostore you can test your install by
   connecting to the Riak console and watching for messages about Innostore
   `sudo /usr/sbin/riak console`. If the install was successful you will
   see something similar to:
```bash
100220 16:36:58 InnoDB: highest supported file format is Barracuda.
100220 16:36:58 Embedded InnoDB 1.0.3.5325 started; log sequence number 45764
```

## Tuning Innostore

While InnoDB can be extremely fast for a durable store, its performance
is highly dependent on tuning the configuration to match the hardware and
dataset. All the configuration is available as application variables in
the `innostore` application scope.

* Store data and logs files on separate spindles/drives

  In general, only the first three parameters (`data_home_dir`,
  `log_group_home_dir` and `buffer_pool_size`) will need to be
  changed. It is _strongly recommended_ that the data home dir and log
  home dir are kept on _separate spindles/drives_.  For best performance,
  we recommend putting Innostore's log data on a separate hard disk from
  the actual data.  This will also make its data more resilient to
  hardware failure -- corrupted writes to the data files can often be
  recovered from the information in the transaction logs.
``` erlang
{innostore, [
	    ...,
             {data_home_dir,            "/innodb"}, %% Where data files go
             {log_group_home_dir,       "/innodb-log"}, %% Where log files go
	     ...
]}
```

* Size the transaction log appropriately

  When storing binary objects or working with a cluster the number
  of log files as well as their size should be increased to handle
  the additional amount of data being stored in Innostore. The
  following error exemplifies the log messages that will be seen if
  your log setup can't cope with the amount of data.
``` bash
InnoDB: ERROR: the age of the last checkpoint is 30209814,
InnoDB: which exceeds the log group capacity 30195303.
InnoDB: If you are using big BLOB or TEXT rows, you must set the
InnoDB: combined size of log files at least 10 times bigger than the
InnoDB: largest such row.
```
  To fix this issue the following log settings will work for most environments:
``` erlang
{innostore, [
	    ...,
{log_files_in_group, 6},  %% How many files you need, usually 3 < x < 6
{log_file_size, 268435456},  %% No bigger than 256MB, otherwise recovery takes too long
	     ...
]}
```

* Maximize the buffer pool size

  The `buffer_pool_size` determines how much data InnoDB tries to keep in
  memory at once. Obviously, the more of your dataset that can fit in
  RAM, the better InnoDB will perform. If you are running a 64-bit Erlang
  VM, the `buffer_pool_size` can safely be set above 2G.  Unlike Bitcask
  InnoDB keeps keys and values in the buffer pool cache, this allows for
  management of key spaces that are larger than available memory.
  We recommend that you set this to be 60-80% of available RAM (after
  subtracting RAM consumed by other services).  For example, on a 12GB
  machine you might set it to 8GB:
```erlang
{innostore, [
	    ...,
	    {buffer_pool_size, 8589934592} %% 8 * 1024 * 1024 * 1024
	    ...
]}
```

* Double buffering only wastes RAM when using InnoDB
  On Linux and other Unix-like platforms, setting this to "O_DIRECT"
  will bypass a layer of filesystem buffering provided by the operating
  system.  It is generally not necessary since Innostore does its own
  buffering.
```erlang
{innostore, [
	    ...,
	    {flush_method, "O_DIRECT"}
	    ...
]}
```

* Be aware of file handle limits

  You can control the number of file descriptors InnoDB will use with
  `open_files`.  InnoDB defaults to 300 which can cause problems on some
  platforms (e.g. OS X has a default limit of 256 handles).
  Innostore opens a file for each partition/bucket combination, plus
  several for its transaction log files. Each of these count against
  the total number of files any one program may have open.  As a result,
  you may need to adjust this number up or down from its default to
  accommodate a lower limit, or more open buckets.  The default is `300`.
  The other option, really the preferred option for production, is to
  increase the number of file handles available.
```erlang
{innostore, [
	    ...,
            {open_files, 100} %% accommodate a lower limit
	    ...
]}
```

* Avoid extra disk head seeks by turning off `noatime`

  Innostore is very aggressive at reading and writing files.  As such,
  you can get a big speed boost by adding the `noatime` mounting option
  to `/etc/fstab`.  This will disable the recording of the "last accessed
  time" for all files.  If you need last access times but you'd like
  some of the benefits of this optimization you can try `relatime`.
```bash
/dev/sda5    /data           ext3    noatime  1 1
/dev/sdb1    /data/inno-log  ext3    noatime  1 2
```

* ZFS

  When running innostore on ZFS, make sure to set the
  `recordsize=16k` on the pool where `data_home_dir` lives (prior to
  starting innostore). You may also find that setting
  `primarycache=metadata` will positively influence performance.


* InnoDB Table Format

  Embedded InnoDb offers several table formats: `compact`, `dynamic` and
  `compressed`.

  - `compact` format stores the first 768 bytes of the value with the
    key and any extra data on extension pages.  This is a good option
    for small values.
  - `dynamic` format stores the value outside separately from the
    index.  For larger values (>600 bytes) this will make the index
    pages denser and may provide a performance increase.
  - `compressed` format is like dynamic format, except it compresses
    the key and data pages.

  The default format is `compact` to match previous innostore
  releases. To configure, set the `format` option in your innostore
  config.  The setting is system wide and will be used for all buckets.

``` erlang
{innostore, [
	    ...,
	    {format, dynamic} %% Use dynamic format tables.
	    ...
]}
```

If you wish to use compressed tables the `page_size` must be set to 0
(changing this is not recommended for any other case).

``` erlang
{innostore, [
	    ...,
	    {format, compressed}, %% Use compressed, dynamic format tables.
            {page_size, 0} %% Compressed format requires page_size be set to 0
	    ...
]}
```

## Miscellaneous

* The maximum key size for Innostore is 255 bytes.
