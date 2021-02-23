FROM ubuntu:18.04

RUN apt-get update && \
    apt-get -y install build-essential strace ltrace && \
    apt-get -y install xz-utils wget pkg-config libncurses-dev kmod tldr && \
    apt-get clean

ADD ./hw.c ./var/

WORKDIR /var

RUN gcc -g -o hw hw.c

RUN ./hw
