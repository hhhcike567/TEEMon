# 使用 debian:unstable-slim 作为基础镜像
FROM reme3180/golang-packr2:latest as builder

# 添加 Ubuntu 20.04 源
RUN echo "deb http://archive.ubuntu.com/ubuntu focal main universe" > /etc/apt/sources.list && \
    echo "deb http://archive.ubuntu.com/ubuntu focal-updates main universe" >> /etc/apt/sources.list && \
    echo "deb http://archive.ubuntu.com/ubuntu focal-security main universe" >> /etc/apt/sources.list

# 添加 apt-key
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3B4FE6ACC0B21F32 || true && \
    apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 91B31D79C30E5EE6 || true && \
    apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 648A4A16A23015EEF4A66B8E4052245BD4284CDD || \
    apt-key adv --keyserver hkp://keyserver.keyserver.ubuntu.com:80 --recv-keys 648A4A16A23015EEF4A66B8E4052245BD4284CDD

# 安装 apt-transport-https
RUN apt-get update && apt-get install -y apt-transport-https

# 添加 iovisor 源
RUN echo "deb https://repo.iovisor.org/apt/xenial xenial main" > /etc/apt/sources.list.d/iovisor.list

RUN apt-get update && apt-get install -y bpfcc-tools linux-headers-generic
       
#RUN apt-get update && apt-get install -y libtinfo5
RUN apt-get install -y libtinfo5 libbcc

# 复制项目文件
COPY ./ /go/ebpf_exporter

# 编译项目
RUN cd /go/ebpf_exporter && GOPATH="/go" GOPROXY="off" GOFLAGS="-mod=vendor" CGO_CFLAGS="-Wno-macro-redefined" go install -v ./... && \
    test -f /go/bin/ebpf_exporter

# 使用 debian:unstable-slim 作为最终镜像
FROM  ziyedm/bcc:ubuntu20.04

# 添加 Ubuntu 20.04 源
RUN echo "deb http://archive.ubuntu.com/ubuntu focal main universe" > /etc/apt/sources.list && \
    echo "deb http://archive.ubuntu.com/ubuntu focal-updates main universe" >> /etc/apt/sources.list && \
    echo "deb http://archive.ubuntu.com/ubuntu focal-security main universe" >> /etc/apt/sources.list

# 安装 gnupg 和 ca-certificates
RUN apt-get update && apt-get install -y gnupg apt-transport-https libtinfo5

# 添加 apt-key
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3B4FE6ACC0B21F32 || true && \
    apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 91B31D79C30E5EE6 || true

# 添加 iovisor 源
RUN echo "deb https://repo.iovisor.org/apt/xenial xenial main" > /etc/apt/sources.list.d/iovisor.list

RUN apt-get update && apt-get install -y linux-headers-generic

# 从 builder 阶段复制编译好的二进制文件
COPY --from=builder /go/bin/ebpf_exporter /usr/bin/ebpf_exporter

# 复制配置文件
COPY ./configfile/sgx.yaml /etc/ebpf_exporter/

# 设置 ENTRYPOINT 和 CMD
ENTRYPOINT [ "ebpf_exporter" ]
CMD [ "--config.file=/etc/ebpf_exporter/sgx.yaml" ]
