FROM ubuntu:20.04
ENV DEBIAN_FRONTEND=noninteractive

# Install essential dependencies required to build GCC
RUN apt-get update && apt-get install -y \
    wget \
    make \
    build-essential \
    libgmp-dev \
    libmpfr-dev \
    libmpc-dev \
    zlib1g-dev \
    flex \
    bison \
    libc6-dev \
    autoconf \
    automake \
    libtool \
    pkg-config \
    libncurses-dev \
    libtinfo-dev \
    libssl-dev


RUN gcc --version

RUN which gcc

# Install GCC 14 from source if it's not available in PPA
RUN wget http://ftp.gnu.org/gnu/gcc/gcc-14.1.0/gcc-14.1.0.tar.gz && \
    tar -xf gcc-14.1.0.tar.gz && \
    cd gcc-14.1.0 && \
    ./contrib/download_prerequisites && \
    ./configure --prefix=/usr/local/gcc-14.1.0 --enable-languages=c,c++ --disable-multilib && \
    make -j$(nproc) && \
    make install && \
    ln -s /usr/local/gcc-14.1.0/bin/gcc /usr/bin/gcc && \
    ln -s /usr/local/gcc-14.1.0/bin/g++ /usr/bin/g++ && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /gcc-14.1.0

# Add newly installed GCC to the PATH
RUN echo "export PATH=/usr/local/gcc-14.1.0/bin:$PATH" >> ~/.bashrc

RUN gcc --version

RUN which gcc
