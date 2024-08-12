from ubuntu:20.04
ENV DEBIAN_FRONTEND=noninteractive

# Basic dependencies
run apt-get update && apt-get install -y git make curl iputils-ping

# rtems dependencies
# sb-set-builder throws ascii character if python3 isnt used
run apt-get install -y flex bison texinfo python-dev python3 g++ unzip

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
  && ../source-builder/sb-check 

# cannot access https://gmplib.org/download/gmp/gmp-6.3.0.tar.lz - curl: (28) Failed to connect to gmplib.org port 443: Connection timed out
# that file is downloaded during sb-set-builder
# try same file from different website
 
run cd home/rtems-source-builder/rtems \
  && mkdir sources \ 
  && cd sources \
  && curl -O https://ftp.gnu.org/gnu/gmp/gmp-6.3.0.tar.bz2

  
run cd home/rtems-source-builder/rtems \  
  && python3 ../source-builder/sb-set-builder --prefix=home/rtems-6 6/rtems-i386 --trace --always-clean --log sb-set-builder.log 

# clone RTEMS source tree
run cd home \
  && git clone https://gitlab.rtems.org/rtems/rtos/rtems.git \
  && export PATH=home/rtems-6/bin:$PATH 
  
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
