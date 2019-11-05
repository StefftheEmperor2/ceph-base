FROM alpine:3.10.3

RUN addgroup -g 1000 node \
	&& adduser -u 1000 -G node -s /bin/sh -D node \
	&& addgroup -g 167 -S ceph \
	&& adduser -u 167 -G ceph -S -s /bin/sh -D ceph \
	&& addgroup ceph abuild \
	&& apk --update add busybox \
		bash \
		coreutils \
		grep \
		sudo \
		git \
		tar \
		perl \
		py3-virtualenv \
		abuild \
		xz \
		curl \
		libstdc++ \
		leveldb \
		leveldb-dev \
		lz4 lz4-libs \ 
		oath-toolkit \
		oath-toolkit-liboath \
		bison \
		flex \
		flex-libs \
		elfutils \
		libdwarf \
		popt \
		libnl \
		libnl3 \
		libnl3-cli \
		libcap-ng \
		gperf \
		rabbitmq-c \
		libzip \
		zlib \
		snappy \
		bison \
		libbz2 \
		libssl1.1 \
		lz4 \
		zstd \
	&& apk add --no-cache --virtual .build-deps \
		binutils-gold \
		curl \
		g++ \
		gcc \
		gnupg \
		libgcc \
		linux-headers \
		make \
		cmake \
		py3-gflags \
		python \
		lz4-dev \
		oath-toolkit-dev \
                autoconf \
                automake \
		glib \
		glib-dev \
		libtool \
		flex-dev \
		elfutils-dev \
		libdwarf-dev \ 
		popt-dev \
		libnl-dev \
		libnl3-dev \
		libcap-ng-dev \
		rabbitmq-c-dev \
		libzip-dev \
		zlib-dev \
		snappy-dev \
		libaio-dev \
		openssl-dev \
		lz4-dev \ 
		zstd-dev \
		boost-dev

RUN mkdir -p /abuild \
	&& cd /abuild \
	&& openssl genrsa -out abuild.rsa 2048 \
	&& chgrp abuild abuild.rsa \
	&& chmod g+r abuild.rsa \
	&& openssl rsa -in abuild.rsa -pubout -out /etc/apk/keys/abuild.rsa.pub \
	&& mkdir -p "/localrepo/local/$(abuild -A)" \
	&& cd /localrepo/local \
	&& apk index -vU -o "/localrepo/local/$(abuild -A)/APKINDEX.tar.gz" \
	&& abuild-sign -k /abuild/abuild.rsa /localrepo/local/$(abuild -A)/APKINDEX.tar.gz \
	&& chgrp -R abuild /localrepo \
	&& chmod -R g+rw /localrepo \
	&& echo "/localrepo/local" >> /etc/apk/repositories
	

COPY install-apk /usr/bin
RUN chmod o+x /usr/bin/install-apk

RUN git clone --branch=master --single-branch https://github.com/gflags/gflags.git /gflags/local \
	&& mkdir -p /gflags/local/alpine \
	&& chmod o+w /usr/include \
	&& chmod o+w /usr/lib

COPY gflags/APKBUILD.in /gflags/local/alpine
COPY gflags/make-apk /gflags/local
COPY gflags/make-dist /gflags/local

RUN chmod u+x /gflags/local/make-apk \
	&& chmod u+x /gflags/local/make-dist

RUN chown -R ceph:ceph /gflags/local
USER 167:167
WORKDIR /gflags/local 
RUN cd /gflags/local \
	&& ./make-dist

USER root

RUN su ceph - -c ./make-apk

RUN apk add gflags \
	&& rm -rf /gflags

RUN cd / \
	&& git clone --single-branch --branch=master https://github.com/facebook/rocksdb.git /rocksdb/local \
	&& mkdir -p /rocksdb/local/alpine

COPY rocksdb/APKBUILD.in /rocksdb/local/alpine
COPY rocksdb/make-apk /rocksdb/local
COPY rocksdb/make-dist /rocksdb/local
	
RUN chmod u+x /rocksdb/local/make-apk \
	&& chmod u+x /rocksdb/local/make-dist

RUN chown -R ceph:ceph /rocksdb/local
USER 167:167
WORKDIR /rocksdb/local
RUN cd /rocksdb/local \
	&& ./make-dist

USER root

RUN su ceph - -c ./make-apk

RUN apk add rocksdb \
	&& rm -rf /rocksdb

RUN cd / \
	&& git clone --single-branch --branch=master https://github.com/ceph/ceph /ceph/local

COPY MYAPKBUILD.in /ceph/local/alpine
COPY make-myapk /ceph/local
COPY make-mydist /ceph/local
COPY 10-client.patch /ceph/local/alpine
COPY 20-indirect_intrusive_heap.patch /ceph/local/alpine
COPY 40-bluestore.patch /ceph/local/alpine

RUN chmod u+x /ceph/local/make-myapk \
	&& chmod u+x /ceph/local/make-mydist

RUN chown -R ceph:ceph /ceph/local
USER 167:167
WORKDIR /ceph/local
RUN cd /ceph/local \
	&& ./make-mydist \
	&& cp alpine/MYAPKBUILD alpine/APKBUILD
USER root
RUN rm alpine/ceph-common.pre-install
RUN su ceph - -c ./make-myapk \
	&& cd ../..  \
	&& rm -rf /ceph
WORKDIR /
