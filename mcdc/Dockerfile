FROM ubuntu:20.04
ENV DEBIAN_FRONTEND=noninteractive

# Step 1: Install dependencies needed for building GCC
RUN apt-get update && apt-get install -y \
    build-essential \
    libgmp-dev \
    libmpfr-dev \
    libmpc-dev \
    wget \
    tar \
    gcc \
    g++ \
    make \
    && rm -rf /var/lib/apt/lists/*

# Step 2: Configure dynamic linker to include /usr/local/lib (for GCC libraries)
RUN echo "/usr/local/lib" > /etc/ld.so.conf.d/gcc.conf && ldconfig

RUN gcc --version

RUN which gcc

# Step 3: Download and build GCC
RUN wget http://ftp.gnu.org/gnu/gcc/gcc-14.1.0/gcc-14.1.0.tar.gz && \
    tar -xf gcc-14.1.0.tar.gz && \
    cd gcc-14.1.0 && \
    ./contrib/download_prerequisites && \
    ./configure --prefix=/usr/local/gcc-14.1.0 --enable-languages=c,c++ --disable-multilib && \
    make -j$(nproc) && \
    make install && \
    ln -sf /usr/local/gcc-14.1.0/bin/gcc /usr/bin/gcc && \
    ln -sf /usr/local/gcc-14.1.0/bin/g++ /usr/bin/g++ && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /gcc-14.1.0

# Add newly installed GCC to the PATH
RUN echo "export PATH=/usr/local/gcc-14.1.0/bin:$PATH" >> ~/.bashrc

RUN gcc --version

RUN which gcc
