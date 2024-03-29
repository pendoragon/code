FROM docker/for-desktop-kernel:5.10.104-ad41e9402fa6e51d2635fb92e4cb6b90107caa25 AS ksrc

FROM ubuntu:20.04 AS bpftrace
COPY --from=ksrc /kernel-dev.tar /
RUN tar xf kernel-dev.tar
# Use Alibaba Cloud mirror for ubuntu
RUN sed -i 's/archive.ubuntu.com/mirrors.aliyun.com/' /etc/apt/sources.list
# Install LLVM 10.0.1
RUN apt-get update && apt install -y wget lsb-release software-properties-common && wget https://apt.llvm.org/llvm.sh && chmod +x llvm.sh && ./llvm.sh 10 && ./llvm.sh 14
ENV PATH "$PATH:/usr/lib/llvm-14/bin"

# Build/Install bpftrace and perf tools
RUN apt-get install -y bpftrace

# WORKDIR
WORKDIR /root

# Install GO
RUN apt-get update && apt-get install -y gcc libc6-dev git vim lrzsz
RUN wget https://go.dev/dl/go1.18.linux-amd64.tar.gz && tar -C /usr/local -zxf go1.18.linux-amd64.tar.gz
ENV GOROOT=/usr/local/go
ENV PATH=$PATH:$GOROOT/bin
ENV GOPATH=/root/go
ENV PATH=$PATH:$GOPATH/bin

RUN apt-get install -y linux-tools-common linux-tools-generic
RUN ln -fs /usr/lib/linux-tools/*/perf /usr/bin/perf

# Build/Install bcc
RUN DEBIAN_FRONTEND="noninteractive" apt install -y kmod vim bison build-essential cmake flex git libedit-dev \
  libcap-dev zlib1g-dev libelf-dev libfl-dev python3.8 python3-pip python3.8-dev clang libclang-dev && \
  ln -s $(which python3) /usr/bin/python
RUN git clone https://github.com/iovisor/bcc.git && \
    cd bcc && git reset --hard v0.24.0 && \
    mkdir build && \
    cd build && \
    cmake .. && \
    make && \
    make install && \
    cmake -DPYTHON_CMD=python3 .. && \
    cd src/python/ && \
    make && \
    make install && \
    sed -i "s/self._syscall_prefixes\[0\]/self._syscall_prefixes\[1\]/g" /usr/lib/python3/dist-packages/bcc/__init__.py

# Install bpftool for debugging
RUN git clone --recurse-submodules https://github.com/libbpf/bpftool.git && \
    cd bpftool/src && \
    make install

CMD mount -t debugfs debugfs /sys/kernel/debug && echo 0 > /proc/sys/kernel/kptr_restrict && /bin/bash
