FROM debian:bullseye-slim
LABEL maintainer "The Infrastructure Company GmbH <info@servercow.de>"

ARG DEBIAN_FRONTEND=noninteractive
ARG SOGO_DEBIAN_REPOSITORY=http://packages.sogo.nu/nightly/5/debian/
# renovate: datasource=github-releases depName=tianon/gosu versioning=semver-coerced extractVersion=^v(?<version>.*)$
ARG GOSU_VERSION=1.16
ENV LC_ALL C

# Prerequisites
RUN echo "Building from repository $SOGO_DEBIAN_REPOSITORY" \
  && apt-get update && apt-get install -y --no-install-recommends \
  apt-transport-https \
  ca-certificates \
  gettext \
  gnupg \
  mariadb-client \
  rsync \
  supervisor \
  syslog-ng \
  syslog-ng-core \
  syslog-ng-mod-redis \
  dirmngr \
  netcat \
  psmisc \
  wget \
  patch \
  && dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')" \
  && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch" \
  && chmod +x /usr/local/bin/gosu \
  && gosu nobody true \
  && mkdir /usr/share/doc/sogo \
  && touch /usr/share/doc/sogo/empty.sh \
  && apt-key adv --keyserver keys.openpgp.org --recv-key 74FFC6D72B925A34B5D356BDF8A27B36A6E2EAE9 \
  && echo "deb ${SOGO_DEBIAN_REPOSITORY} bullseye bullseye" > /etc/apt/sources.list.d/sogo.list \
  && apt-get update && apt-get install -y --no-install-recommends \
    sogo \
    sogo-activesync \
  && apt-get autoclean \
  && rm -rf /var/lib/apt/lists/* /etc/apt/sources.list.d/sogo.list \
  && touch /etc/default/locale

COPY ./bootstrap-sogo.sh /bootstrap-sogo.sh
COPY syslog-ng.conf /etc/syslog-ng/syslog-ng.conf
COPY syslog-ng-redis_slave.conf /etc/syslog-ng/syslog-ng-redis_slave.conf
COPY supervisord.conf /etc/supervisor/supervisord.conf
COPY acl.diff /acl.diff
COPY stop-supervisor.sh /usr/local/sbin/stop-supervisor.sh
COPY docker-entrypoint.sh /

RUN chmod +x /bootstrap-sogo.sh \
  /usr/local/sbin/stop-supervisor.sh

ENTRYPOINT ["/docker-entrypoint.sh"]

CMD exec /usr/bin/supervisord -c /etc/supervisor/supervisord.conf