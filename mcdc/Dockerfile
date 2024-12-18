from ubuntu:20.04
ENV DEBIAN_FRONTEND=noninteractive

# Basic dependencies
run apt-get update && apt-get install -y git make curl

# Install required packages and a compatible version of GCC before GCC 14
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    $buildtest_pkg $dev_pkg $debug_pkg $doxy_pkg $ci_pkg $gs_pkg \
    && apt-get install -y software-properties-common \
    && apt-get install -y bzip2 \
    && add-apt-repository -y ppa:ubuntu-toolchain-r/test \
    && apt-get update \
    && apt-get install -y gcc g++ \
    && apt-get install -y gcc-14 g++-14 \
    || (echo "GCC 14 not found in PPA, attempting to install from source..." && \
    wget http://ftp.gnu.org/gnu/gcc/gcc-14.1.0/gcc-14.1.0.tar.gz && \
    tar -xf gcc-14.1.0.tar.gz && \
    cd gcc-14.1.0 && \
    ./contrib/download_prerequisites && \
    ./configure --prefix=/usr/local/gcc-14.1.0 --enable-languages=c,c++ --disable-multilib && \
    make && make install) \
    && rm -rf /var/lib/apt/lists/*

# Set the PATH to prioritize GCC 14.1.0, ensuring it is used for compilation and gcov analysis.
RUN export PATH=/usr/local/gcc-14.1.0/bin:$PATH 
