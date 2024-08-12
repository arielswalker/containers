from ubuntu:20.04
ENV DEBIAN_FRONTEND=noninteractive

# Basic dependencies
run apt-get update && apt-get install -y git make curl

# rtems dependencies
# error: config error: gdb-common-1.cfg:123: "gdb: python: header file not found: python3.8/Python.
# sb-set-builder log says env needs gmp 4.2+ and mpfr 3.1.0+ 
run apt-get install -y flex bison texinfo python-dev g++ unzip libgmp-dev libmpfr-dev python3.8-dev

# QEMU dependencies
run apt-get install -y qemu-system-i386

# setup and compile RTEMS BSP
# When checking out/cloning the 5 rtems-source-builder branch, there is an error when using sb-set-builder
#2024-08-08T22:51:29.6122634Z #9 1492.8 _bset:   : 5/rtems-i386: 006: %error RTEMS version is not defined
# --target=i386-rtems5 and --with-target=i386-rtems5 did not fix the error 
run mkdir -p home/rtems-5 \
  && cd home\
  && git clone -b 5 https://gitlab.rtems.org/rtems/tools/rtems-source-builder.git \
  && cd rtems-source-builder/rtems \
  && ../source-builder/sb-check 

# Download gnu mpc before source builder will fail to download it
run cd home/rtems-source-builder/rtems \
  && mkdir sources \
  && curl https://ftp.gnu.org/gnu/mpc/mpc-1.0.3.tar.gz --output sources/mpc-1.0.3.tar.gz \
  && ../source-builder/sb-set-builder --prefix=$HOME/rtems-5 --target=i386-rtems5 5/rtems-i386 --trace --always-clean --log sb-set-builder.log 
# || true \ && cat sb-set-builder.log 

# clone and bootstrap RTEMS source tree
run cd home\
  && git clone -b 5 https://gitlab.rtems.org/rtems/rtos/rtems.git \
  && export PATH=$HOME/rtems-5/bin:$PATH \
  && cd rtems && ./bootstrap 

# Build and install RTEMS pc686 BSP
run cd home\
  && mkdir b-pc686 \
  && cd b-pc686 \
  && ../rtems/configure --target=i386-rtems5 \
    --enable-rtemsbsp=pc686 \
    --prefix=home/rtems-5 \
    --enable-networking \
    --enable-cxx \
    --disable-posix \
    --disable-deprecated \
  && make \
  && make install

# install git from source for updated version
run apt-get install -y libz-dev libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext cmake
run curl -o git.tar.gz https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.26.2.tar.gz
run tar -zxf git.tar.gz && rm git.tar.gz
run cd git-* && make prefix=/usr/local && make prefix=/usr/local install

# Delete unnecessary directories now that rtems is built
run cd home\
  && rm -rf rtems-source-builder b-pc686 rtems

# Additional dependencies
run apt-get install -y -f mtools parted udev bsdmainutils

# Install dosfstools from source release
run curl -o dosfstools.tar.gz http://deb.debian.org/debian/pool/main/d/dosfstools/dosfstools_4.2.orig.tar.gz
run tar -zxf dosfstools.tar.gz && rm dosfstools.tar.gz
run cd dosfstools-* && ./configure && make && make prefix=/usr/local install

# Remove unecessary dependencies now that rtems is built
run apt-get purge -y libz-dev libssl-dev libcurl4-gnutls-dev libexpat1-dev gettext unzip git flex bison texinfo curl
run apt-get autoremove -y
