Riak 1.0 requires [[Erlang|http://erlang.org]] R14B03 or later.  Riak versions
prior to 1.0 will not function on the R14B02 or later. Riak versions prior to
0.12 will not function on the R14 series of Erlang. For Erlang to build and
install, you must have a GNU-compatible build system, and the development
bindings of `ncurses` and `openssl`.

<div class="info">
<p>The Riak binary packages for Debian and Ubuntu, Mac OS X,  and RHEL and
CentOS do not require that you build Erlang from source. <strong>You will have
to download and install Erlang, however, if you're planning on completing
[[The Riak Fast Track]].</strong></p>
</div>

<div id="toc"></div>

## Install using kerl

kerl is a simple shell script that allows installing different Erlang versions
with only two commands. It's probably the easiest way to install Erlang from
source on a system.  Installing kerl is as simple as running the following
command

```bash
$ curl -O https://raw.github.com/spawngrid/kerl/master/kerl; chmod a+x kerl
```

In order for kerl to compile erlang as 64-bit on Mac OS X, you'll need to 
configure kerl to pass the correct flags to the ```configure``` command. 
The easiest way to do it is by creating a file ```~/.kerlrc``` with the
following contents:

```
KERL_CONFIGURE_OPTIONS="--enable-hipe --enable-smp-support --enable-threads 
                        --enable-kernel-poll  --enable-darwin-64bit"
```

Building with kerl on GNU/Linux has the same prerequisites that 
[[building from source|Installing-Erlang#Installing-on-GNU-Linux]] does. 



Then you can just build the Erlang release of your choice, as of current, you'll
need r14b03:

```bash
$ ./kerl build R14B03 r14b03
```

This installs Erlang and does all the steps required to manually install Erlang
for you.

When successfully built you can install it using

```bash
$ ./kerl install r14b03 /opt/erlang/r14b03
$ . /opt/erlang/r14b03/activate
```

The last line activates the Erlang build that was just installed into
/opt/erlang/r14b03.  See the [[kerl readme|https://github.com/spawngrid/kerl]]
for more details on the available commands.

If you prefer to install completely manually though, we've got you covered too.

## Installing on GNU/Linux

Most distributions *do not* have the most recent Erlang release available, **so
you will need to install from source**.

First, make sure you have a compatible build system and the `ncurses` and
`openssl` development libraries installed.  On Debian/Ubuntu use this command:

```bash
$ sudo apt-get install build-essential libncurses5-dev openssl libssl-dev
```

On RHEL/CentOS use this command:

```bash
$ sudo yum install gcc glibc-devel make ncurses-devel openssl-devel autoconf
```

Next, download, build and install Erlang:

```bash
$ wget http://erlang.org/download/otp_src_R14B03.tar.gz
$ tar zxvf otp_src_R14B03.tar.gz
$ cd otp_src_R14B03
$ ./configure && make && sudo make install
```

## Installing on Mac OS/X

You can install Erlang in several ways on OS/X: from source, with Homebrew, or
with MacPorts.

### Source

To build from source, you must have XCode tools installed from the CD that came
with your Mac or from Apple's [[Developer website|http://developer.apple.com/]].

First, download and unpack the source:

```bash
$ curl -O http://erlang.org/download/otp_src_R14B03.tar.gz
$ tar zxvf otp_src_R14B03.tar.gz
$ cd otp_src_R14B03
```

Next, configure Erlang.  

If you're on Lion (OS/X 10.7) you can use LLVM, the default, or GCC to compile
Erlang.

Using LLVM:

```bash
$ CFLAGS=-O0 ./configure --enable-hipe --enable-smp-support --enable-threads \
--enable-kernel-poll --enable-darwin-64bit
```

If you prefer GCC:

```bash
$ CC=gcc-4.2 CPPFLAGS='-DNDEBUG' MAKEFLAGS='-j 3' \
./configure --enable-hipe --enable-smp-support --enable-threads \
--enable-kernel-poll --enable-darwin-64bit
```

If you're on Snow Leopard (OS/X 10.6) or Leopard (OS/X 10.5) with an Intel
processor:

```bash
$ ./configure --enable-hipe --enable-smp-support --enable-threads \
--enable-kernel-poll  --enable-darwin-64bit
```

If you're on a non-Intel processor or older version of OS/X:

```bash
$ ./configure --enable-hipe --enable-smp-support --enable-threads \
--enable-kernel-poll
```

Now build and install:

```bash
$ make && sudo make install
```

You will be prompted for your sudo password.

###  Homebrew

If you want to install Riak with Homebrew, simply follow these instructions
[[here|Installing on Mac OS X]] and Erlang will be installed automatically. To
install it separately:

```bash
$ brew install erlang
```

###  MacPorts

Installing with MacPorts is easy:


```bash
$ port install erlang +ssl
```
