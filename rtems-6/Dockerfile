from ubuntu:20.04
ENV DEBIAN_FRONTEND=noninteractive

# Basic dependencies
run apt-get update && apt-get install -y git make curl

# rtems dependencies
run apt-get install -y flex bison texinfo python-dev g++ unzip

# Added: Python development package installation for RTEMS build dependencies
run apt-get install -y python3-dev libpython3-dev

# QEMU dependencies
run apt-get install -y qemu-system-i386

# Copy config.ini file - Note that the environment variable HOME will not work here
copy rtems-6/config.ini /config.ini

# Temporary fix for legacy network stack
copy rtems-6/testbusy.c /testbusy.c

run mkdir -p home/rtems-6 \
  && cd home \
  && git clone https://gitlab.rtems.org/rtems/tools/rtems-source-builder.git \
  && cd rtems-source-builder/rtems \
  && ../source-builder/sb-check \

# ERROR RUNNING sb-set-builder
# Build Set: devel/gmp-internal.bset
# config: devel/gmp-6.3.0.cfg
# gmp-6.3.0-x86_64-linux-gnu-1
# https://gmplib.org/download/gmp/gmp-6.3.0.tar.bz2 -> sources/gmp-6.3.0.tar.bz2
# download: https://gmplib.org/download/gmp/gmp-6.3.0.tar.bz2: error: <urlopen error [Errno 110] Connection timed out>
# try to manually download gmp but the smallest version file which is tar.lz not tar.bz2 
#run cd home/rtems-source-builder/rtems \
#  && mkdir sources \
#  && curl https://gmplib.org/download/gmp/gmp-6.3.0.tar.lz --output sources/gmp-6.3.0.tar.lz \
  && ../source-builder/sb-set-builder --prefix=$HOME/rtems-6 6/rtems-i386 --trace --always-clean --log sb-set-builder.log 
  
# clone RTEMS source tree
run cd home \
  && git clone https://gitlab.rtems.org/rtems/rtos/rtems.git \
  && export PATH=$HOME/rtems-6/bin:$PATH 
  
# Build RTEMS BSP 
run cd rtems \
  && git checkout 3ccfb583 \
  && mv /config.ini . \
  && ./waf configure --prefix=home/rtems-6 \
  && ./waf \
  && ./waf install 
  
# Get and build the legacy network stack
run cd home \
  && git clone git://git.rtems.org/rtems-net-legacy.git \
  && cd rtems-net-legacy \
  && git checkout 1d492241 \
  && git submodule update --init \
  && mv /testbusy.c libtest/testbusy.c \
  && ./waf configure --prefix=$HOME/rtems-6 \
  && ./waf build install 
 
# install git from source for updated version
run apt-get install -y libz-dev libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext cmake
run curl -o git.tar.gz https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.26.2.tar.gz
run tar -zxf git.tar.gz && rm git.tar.gz
run cd git-* && make prefix=/usr/local && make prefix=/usr/local install

# Delete unnecessary directories now that rtems is built
run cd home \
  && rm -rf rtems-source-builder rtems rtems-net-legacy

# Additional dependencies
run apt-get install -y -f mtools parted udev bsdmainutils

# Install dosfstools from source release
run curl -o dosfstools.tar.gz http://deb.debian.org/debian/pool/main/d/dosfstools/dosfstools_4.2.orig.tar.gz
run tar -zxf dosfstools.tar.gz && rm dosfstools.tar.gz
run cd dosfstools-* && ./configure && make && make prefix=/usr/local install

# Remove unecessary dependencies now that rtems is built
run apt-get purge -y libz-dev libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext unzip git flex bison texinfo curl
run apt-get autoremove -y
