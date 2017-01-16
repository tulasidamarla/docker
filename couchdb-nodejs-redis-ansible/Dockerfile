# Pull base image.
FROM opensuse:13.2

MAINTAINER Tulasi Ram ram.damarla@gmail.com

RUN zypper update -y

#
# Python
#
RUN zypper install -y python python-devel python-pip python-virtualenv

#
# Node.js and NPM
#
RUN zypper install -y nodejs npm git
RUN ln -s /dev/null

#
# Grunt, Bower, Forever
#

RUN npm install -g grunt \
        && npm install -g bower \
        && npm install -g forever

#
#CouchDB
#
RUN useradd -m couchdb \
        && zypper update -y \
        && zypper install -y curl which \
        && mkdir -p /var/lib/couchdb
VOLUME /var/lib/couchdb
RUN zypper install -y couchdb
RUN sed -i'' 's/bind_address = 127.0.0.1/bind_address = 0.0.0.0/' /etc/couchdb/default.ini
EXPOSE 5984

#CMD ["/usr/bin/couchdb"]

#
# Ansible
#

RUN zypper update -y && zypper install -y ansible

#
# Install Redis
#

RUN zypper install -y wget

ENV REDIS_VERSION 3.2.4
ENV REDIS_DOWNLOAD_URL http://download.redis.io/releases/redis-3.2.4.tar.gz
ENV REDIS_DOWNLOAD_SHA1 f0fe685cbfdb8c2d8c74613ad8a5a5f33fba40c9

RUN zypper install -y tar make gcc

# for redis-sentinel see: http://redis.io/topics/sentinel
RUN wget -O redis.tar.gz "$REDIS_DOWNLOAD_URL" \
        && echo "$REDIS_DOWNLOAD_SHA1 *redis.tar.gz" | sha1sum -c - \
        && mkdir -p /usr/src/redis \
        && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
        && rm redis.tar.gz \
        \
# Disable Redis protected mode [1] as it is unnecessary in context
# of Docker. Ports are not automatically exposed when running inside
# Docker, but rather explicitely by specifying -p / -P.
# [1] https://github.com/antirez/redis/commit/edd4d555df57dc84265fdfb4ef59a4678832f6da
        && grep -q '^#define CONFIG_DEFAULT_PROTECTED_MODE 1$' /usr/src/redis/src/server.h \
        && sed -ri 's!^(#define CONFIG_DEFAULT_PROTECTED_MODE) 1$!\1 0!' /usr/src/redis/src/server.h \
        && grep -q '^#define CONFIG_DEFAULT_PROTECTED_MODE 0$' /usr/src/redis/src/server.h \
# for future reference, we modify this directly in the source instead of just supplying a default configuration flag because apparently "if you specify any argument to redis-server, [it assumes] you are going to specify everything"
# see also https://github.com/docker-library/redis/issues/4#issuecomment-50780840
# (more exactly, this makes sure the default behavior of "save on SIGTERM" stays functional by default)
        \
        && cd /usr/src/redis/deps \
        && make distclean \
        && make hiredis lua jemalloc linenoise \
        && make -C /usr/src/redis \
        && make -C /usr/src/redis install \
        \
        && rm -r /usr/src/redis

EXPOSE 6379
