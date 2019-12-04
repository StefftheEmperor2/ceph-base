FROM alpine:3.10.3

ENV NODE_VERSION 10.16.3

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
		py3-pip \
		py2-pip \
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
	&& cp /tmp/gflags/APKBUILD.in /gflags/local/alpine \
	&& cp /tmp/gflags/make-apk /gflags/local \
	&& cp /tmp/gflags/make-dist /gflags/local \
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
	&& for key in \
		94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
		FD3A5288F042B6850C66B31F09FE44734EB7990E \
		71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
		DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
		C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
		B9AE9905FFD7803F25714661B63B535A4C206CA9 \
		77984A986EBC2AA786BC0F66B01FBB92821C587A \
		8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
		4ED778F539E3634C779C87C6D7062848A1AB005C \
		A48C2BEE680E841632CD4E44F07496B3EB3C1762 \
		B9E2F5981AA6E0CD28160D9FF13993A75599653C \
	; do \
		gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
		gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
		gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
	done \
	&& curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION.tar.xz" \
	&& curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
	&& gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
	&& grep " node-v$NODE_VERSION.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
	&& tar -xf "node-v$NODE_VERSION.tar.xz" \
	&& cd "node-v$NODE_VERSION" \
	&& ./configure \
	&& make -j$(getconf _NPROCESSORS_ONLN) V= \
	&& make install \
	&& cd .. \
	&& rm -Rf "node-v$NODE_VERSION" \
	&& rm "node-v$NODE_VERSION.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
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
	&& cd /  \
	&& rm -rf /ceph \
	&& apk add ceph \
	&& pip3 install influxdb \
	&& pip2 install influxdb \
	&& apk del .build-deps \
	&& mkdir -p /var/run/ceph \
	&& chown ceph:ceph /var/run/ceph
