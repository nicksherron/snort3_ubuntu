# Installing snort3 on ubuntu
[source "Snort3 on Ubuntu 18 & 19"](https://snort-org-site.s3.amazonaws.com/production/document_files/files/000/000/211/original/Snort3.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIXACIED2SPMSC7GA%2F20200329%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20200329T084137Z&X-Amz-Expires=172800&X-Amz-SignedHeaders=host&X-Amz-Signature=c09c75bd73b1274844a512e369d939fcffffa171dd651b878a4e116c00d66a17)

## prep
First, ensure your system is up to date and has the latest list of packages:
```
sudo apt-get update && sudo apt-get dist-upgrade -y
```

We will be downloading a number of source tarballs and other source files, we want to store them in
one folder:
```
mkdir ~/snort_src
cd ~/snort_src
```

## prequisites

```
sudo apt-get install -y build-essential autotools-dev libdumbnet-dev \
libluajit-5.1-dev libpcap-dev zlib1g-dev pkg-config libhwloc-dev \
cmake
```

optional (but highly recommended software)

```
sudo apt-get install -y liblzma-dev openssl libssl-dev cpputest \
libsqlite3-dev uuid-dev
```

If you want to build the latest documentation from the source tree including Snort++ Developers Guide,
install the following (purely optional) packages. These packages are nearly 800 MB in size and can be
skipped unless you specifically want the dev guide:
```
sudo apt-get install -y asciidoc dblatex source-highlight w3m
```

Since we will install Snort from the github repository, we need a few tools (not necessary on Ubuntu
19):
```
sudo apt-get install -y libtool git autoconf
```

The Snort DAQ (Data Acquisition library)has a few pre-requisites that need to be installed:
```
sudo apt-get install -y bison flex
```

If you want to run Snort in inline mode using NFQ, install the required packages (not required for IDS
mode or inline mode using afpacket). If you’re unsure, you should install this package.
```
sudo apt-get install -y libnetfilter-queue-dev libmnl-dev
```

Download and install safec for runtime bounds checks on certain legacy C-library calls (this is optional
but recommended):
```
cd ~/snort_src
wget https://github.com/rurban/safeclib/releases/download/v04062019/libsafec-04062019.0-ga99a05.tar.gz
tar -xzvf libsafec-04062019.0-ga99a05.tar.gz
cd libsafec-04062019.0-ga99a05/
./configure
make
sudo make install
```

Install PCRE: Perl Compatible Regular Expressions. We don’t use the Ubuntu repository because it has
an older version.
```
cd ~/snort_src/
wget https://ftp.pcre.org/pub/pcre/pcre-8.43.tar.gz
tar -xzvf pcre-8.43.tar.gz
cd pcre-8.43
./configure
make
sudo make install
```

Download and install gperftools 2.7, google’s thread-caching malloc (used in chrome). Tcmalloc is a
memory allocator that’s optimized for high concurrency situations which will provide better speed
for the trade-off of higher memory usage. We don’t want the version of tcmalloc from the Ubuntu
repository (version 2.5) as it is not compatible with Snort. Tcmalloc is optional but recommended:
```
cd ~/snort_src
wget https://github.com/gperftools/gperftools/releases/download/gperftools-2.7/gperftools-2.7.tar.gz
tar xzvf gperftools-2.7.tar.gz
cd gperftools-2.7
./configure
make
sudo make install
```

Snort 3 uses Hyperscan for fast pattern matching. Hyperscan requires Ragel and the Boost headers:
```
cd ~/snort_src
wget http://www.colm.net/files/ragel/ragel-6.10.tar.gz
tar -xzvf ragel-6.10.tar.gz
cd ragel-6.10
./configure
make
sudo make install
```

Hyperscan requires the Boost C++ Libraries. Note that we are not using the Ubuntu repository version
of the boost headers (libboost-all-dev) because Hyperscan requires boost libraries at or above version
number 1.58, and the Ubuntu repository version is too old. Download the Boost 1.71.0 libraries, but do
not install:
```
cd ~/snort_src
wget https://dl.bintray.com/boostorg/release/1.71.0/source/boost_1_71_0.tar.gz
tar -xvzf boost_1_71_0.tar.gz
```

Install Hyperscan 5.2 from source, referencing the location of the Boost headers source directory:
```
cd ~/snort_src
wget https://github.com/intel/hyperscan/archive/v5.2.0.tar.gz
tar -xvzf v5.2.0.tar.gz
mkdir ~/snort_src/hyperscan-5.2.0-build
cd hyperscan-5.2.0-build/
cmake -DCMAKE_INSTALL_PREFIX=/usr/local -DBOOST_ROOT=~/snort_src/boost_1_71_0/ ../hyperscan-5.2.0
make
sudo make install
```

If you want to test that Hyperscan works, from the build directory, run:
```
cd ~/snort_src/hyperscan-5.2.0-build/
./bin/unit-hyperscan
```

Snort has an optional requirement for flatbuers, A memory eicient serialization library:
```
cd ~/snort_src
wget https://github.com/google/flatbuffers/archive/v1.11.0.tar.gz \
  -O flatbuffers-v1.11.0.tar.gz
tar -xzvf flatbuffers-v1.11.0.tar.gz
mkdir flatbuffers-build
cd flatbuffers-build
cmake ../flatbuffers-1.11.0
make
sudo make install
```

Next, download and install Data AcQuisition library (DAQ) from the Snort website. Note that Snort 3
uses a different DAQ than the Snort 2.9.x.x series:
```
cd ~/snort_src
git clone https://github.com/snort3/libdaq.git
cd libdaq
./bootstrap
./configure
make
sudo make install
```

Update shared libraries:
```
sudo ldconfig
```
## install

Now we are ready to download, compile, and install Snort 3 from the github repository. If you are
interested in enabling additional compile-time functionality, such as the ability to process large (over 2
GB) PCAP files, or the new command line shell, you should run ./configure cmake.sh --help to
list all possible options. If you want to install to a different location, please see Appendix B.
Download and install, with default settings:

```
cd ~/snort_src
git clone git://github.com/snortadmin/snort3.git
cd snort3
./configure_cmake.sh --prefix=/usr/local --enable-tcmalloc
cd build
make
sudo make install
```

The last step ins the installation is to verify that the Snort installed and can run. To do this, we pass the
snort executable it the -V flag:
```
/usr/local/bin/snort -v
```
