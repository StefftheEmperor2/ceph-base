FROM alpine:3.10.3

RUN mkdir -p /tmp/gflags \
	&& mkdir -p /tmp/rocksdb \
	&& mkdir -p /tmp/ceph \
	&& mkdir -p /tmp/cmake

COPY cmake/APKBUILD.in /tmp/cmake
COPY cmake/make-apk /tmp/cmake
COPY cmake/make-dist /tmp/cmake

COPY gflags/APKBUILD.in /tmp/gflags
COPY gflags/make-apk /tmp/gflags
COPY gflags/make-dist /tmp/gflags

COPY rocksdb/APKBUILD.in /tmp/rocksdb
COPY rocksdb/make-apk /tmp/rocksdb
COPY rocksdb/make-dist /tmp/rocksdb

COPY ceph/APKBUILD.in /tmp/ceph
COPY ceph/make-apk /tmp/ceph
COPY ceph/make-dist /tmp/ceph
COPY ceph/10-client.patch /tmp/ceph
COPY ceph/20-indirect_intrusive_heap.patch /tmp/ceph
COPY ceph/40-bluestore.patch /tmp/ceph
COPY ceph/50-rados.patch /tmp/ceph

RUN addgroup -g 1000 node \
	&& adduser -u 1000 -G node -s /bin/sh -D node \
	&& addgroup -g 167 -S ceph \
	&& adduser -u 167 -G ceph -S -s /bin/sh -D ceph \
	&& addgroup ceph abuild \
	&& addgroup ceph disk \
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
		boost-dev \
	&& mkdir -p /abuild \
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
	&& echo "/localrepo/local" >> /etc/apk/repositories \
	&& cd / \
	&& git clone --single-branch --branch=master https://gitlab.kitware.com/cmake/cmake.git /cmake/local \
	&& mkdir -p /cmake/local/alpine \
	&& cp /tmp/cmake/APKBUILD.in /cmake/local/alpine \
	&& cp /tmp/cmake/make-apk /cmake/local \
	&& cp /tmp/cmake/make-dist /cmake/local \
	&& rm -rf /tmp/cmake \
	&& chmod u+x /cmake/local/make-apk \
	&& chmod u+x /cmake/local/make-dist \
	&& chown -R ceph:ceph /cmake/local \
	&& cd /cmake/local \
	&& su ceph - -c ./make-dist \
	&& su ceph - -c ./make-apk \
	&& cd / \
	&& git clone --branch=master --single-branch https://github.com/gflags/gflags.git /gflags/local \
	&& mkdir -p /gflags/local/alpine \
	&& chmod o+w /usr/include \
	&& chmod o+w /usr/lib \
	&& mv /tmp/gflags/APKBUILD.in /gflags/local/alpine \
	&& mv /tmp/gflags/make-apk /gflags/local \
	&& mv /tmp/gflags/make-dist /gflags/local \
	&& rm -rf /tmp/gflags \
	&& chmod u+x /gflags/local/make-apk \
	&& chmod u+x /gflags/local/make-dist \
	&& chown -R ceph:ceph /gflags/local \
	&& cd /gflags/local \
	&& su ceph - -c ./make-dist \
	&& su ceph - -c ./make-apk \
	&& apk add gflags \
	&& rm -rf /gflags \
	&& cd / \
	&& git clone --single-branch --branch=master https://github.com/facebook/rocksdb.git /rocksdb/local \
	&& mkdir -p /rocksdb/local/alpine \
	&& cp /tmp/rocksdb/APKBUILD.in /rocksdb/local/alpine \
	&& cp /tmp/rocksdb/make-apk /rocksdb/local \
	&& cp /tmp/rocksdb/make-dist /rocksdb/local \
	&& rm -rf /tmp/rocksdb \
	&& chmod u+x /rocksdb/local/make-apk \
	&& chmod u+x /rocksdb/local/make-dist \
	&& chown -R ceph:ceph /rocksdb/local \
	&& cd /rocksdb/local \
	&& su ceph - -c ./make-dist \
	&& su ceph - -c ./make-apk \
	&& apk add rocksdb \
	&& rm -rf /rocksdb \
	&& cd / \
	&& git clone --single-branch --branch=master https://github.com/ceph/ceph /ceph/local \
	&& cp /tmp/ceph/APKBUILD.in /ceph/local/alpine \
	&& cp /tmp/ceph/make-apk /ceph/local \
	&& cp /tmp/ceph/make-dist /ceph/local \
	&& cp /tmp/ceph/10-client.patch /ceph/local/alpine \
	&& cp /tmp/ceph/20-indirect_intrusive_heap.patch /ceph/local/alpine \
	&& cp /tmp/ceph/40-bluestore.patch /ceph/local/alpine \
	&& cp /tmp/ceph/50-rados.patch /ceph/local/alpine \
	&& rm -rf /tmp/ceph \
	&& chmod u+x /ceph/local/make-apk \
	&& chmod u+x /ceph/local/make-dist \
	&& chown -R ceph:ceph /ceph/local \
	&& cd /ceph/local \
	&& su ceph - -c ./make-dist \
	&& su ceph - -c ./make-apk \
	&& cd / \
	&& rm -rf /ceph \
	&& apk add ceph \
	&& apk del .build-deps
