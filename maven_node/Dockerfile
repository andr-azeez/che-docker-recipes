#FROM ubuntu:18.04
FROM quay.io/eclipse/che-java11-maven:nightly
USER 0
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates curl netbase wget && rm -rf /var/lib/apt/lists/*

RUN set -ex; if ! command -v gpg > /dev/null; then apt-get update; apt-get install -y --no-install-recommends gnupg dirmngr ; rm -rf /var/lib/apt/lists/*; fi

RUN apt-get update && apt-get install -y --no-install-recommends bzr git mercurial openssh-client subversion procps && rm -rf /var/lib/apt/lists/*


RUN set -ex; apt-get update; apt-get install -y --no-install-recommends autoconf automake bzip2 dpkg-dev file g++ gcc imagemagick libbz2-dev libc6-dev libcurl4-openssl-dev libdb-dev libevent-dev libffi-dev libgdbm-dev libglib2.0-dev libgmp-dev libjpeg-dev libkrb5-dev liblzma-dev libmagickcore-dev libmagickwand-dev libmaxminddb-dev libncurses5-dev libncursesw5-dev libpng-dev libpq-dev libreadline-dev libsqlite3-dev libssl-dev libtool libwebp-dev libxml2-dev libxslt-dev libyaml-dev make patch unzip xz-utils zlib1g-dev $( if apt-cache show 'default-libmysqlclient-dev' 2>/dev/null | grep -q '^Version:'; then echo 'default-libmysqlclient-dev'; else echo 'libmysqlclient-dev'; fi ) ; rm -rf /var/lib/apt/lists/*


RUN groupadd --gid 1000 node && useradd --uid 1000 --gid node --shell /bin/bash --create-home node

ENV NODE_VERSION=14.2.0

RUN ARCH= && dpkgArch="$(dpkg --print-architecture)" && case "${dpkgArch##*-}" in amd64) ARCH='x64';; ppc64el) ARCH='ppc64le';; s390x) ARCH='s390x';; arm64) ARCH='arm64';; armhf) ARCH='armv7l';; i386) ARCH='x86';; *) echo "unsupported architecture"; exit 1 ;; esac && set -ex && for key in 94AE36675C464D64BAFA68DD7434390BDBE9B9C5 FD3A5288F042B6850C66B31F09FE44734EB7990E 71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 DD8F2338BAE7501E3DD5AC78C273792F7D83545D C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 B9AE9905FFD7803F25714661B63B535A4C206CA9 77984A986EBC2AA786BC0F66B01FBB92821C587A 8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 4ED778F539E3634C779C87C6D7062848A1AB005C A48C2BEE680E841632CD4E44F07496B3EB3C1762 B9E2F5981AA6E0CD28160D9FF13993A75599653C ; do gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; done && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH.tar.xz" && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc && grep " node-v$NODE_VERSION-linux-$ARCH.tar.xz\$" SHASUMS256.txt | sha256sum -c - && tar -xJf "node-v$NODE_VERSION-linux-$ARCH.tar.xz" -C /usr/local --strip-components=1 --no-same-owner && rm "node-v$NODE_VERSION-linux-$ARCH.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt && ln -s /usr/local/bin/node /usr/local/bin/nodejs


ENV YARN_VERSION=1.22.4


RUN set -ex && for key in 6A010C5166006599AA17F08146C2130DFD2497F5 ; do gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; done && curl -fsSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz" && curl -fsSLO --compressed "https://yarnpkg.com/downloads/$YARN_VERSION/yarn-v$YARN_VERSION.tar.gz.asc" && gpg --batch --verify yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz && mkdir -p /opt && tar -xzf yarn-v$YARN_VERSION.tar.gz -C /opt/ && ln -s /opt/yarn-v$YARN_VERSION/bin/yarn /usr/local/bin/yarn && ln -s /opt/yarn-v$YARN_VERSION/bin/yarnpkg /usr/local/bin/yarnpkg && rm yarn-v$YARN_VERSION.tar.gz.asc yarn-v$YARN_VERSION.tar.gz

RUN echo '#!/bin/sh \n\
    set -e \n\
if [ "${1#-}" != "${1}" ] || [ -z "$(command -v "${1}")" ]; then \n\
  set -- node "$@" \n\
fi \n\
exec "$@"'>/usr/local/bin/docker-entrypoint.sh


ENTRYPOINT ["docker-entrypoint.sh"]

CMD ["node"]




RUN echo '#!/bin/bash \n\
#
# Copyright (c) 2019-2021 Red Hat, Inc.
# This program and the accompanying materials are made
# available under the terms of the Eclipse Public License 2.0
# which is available at https://www.eclipse.org/legal/epl-2.0/
#
# SPDX-License-Identifier: EPL-2.0
#

# Ensure $HOME exists when starting \n\
if [ ! -d "${HOME}" ]; then \n\
  mkdir -p "${HOME}" \n\
fi \n\
# Setup $PS1 for a consistent and reasonable prompt \n\
if [ -w "${HOME}" ] && [ ! -f "${HOME}"/.bashrc ]; then \n\
  echo "PS1='\s-\v \w \$ '" > "${HOME}"/.bashrc \n\
fi \n\
# Add current (arbitrary) user to /etc/passwd and /etc/group \n\
if ! whoami &> /dev/null; then \n\
  if [ -w /etc/passwd ]; then \n\
    echo "${USER_NAME:-user}:x:$(id -u):0:${USER_NAME:-user} user:${HOME}:/bin/bash" >> /etc/passwd \n\
    echo "${USER_NAME:-user}:x:$(id -u):" >> /etc/group \n\
  fi \n\
fi \n\
exec "$@"'>/entrypoint.sh


 
RUN chown 0:0 /entrypoint.sh

USER 0

RUN mkdir -p /home/user && chgrp -R 0 /home && chmod -R g=u /etc/passwd /etc/group /home && chmod +x /entrypoint.sh

USER root

RUN echo '#!/bin/sh \n\
#
# Copyright (c) 2020-2021 Red Hat, Inc.
# This program and the accompanying materials are made
# available under the terms of the Eclipse Public License 2.0
# which is available at https://www.eclipse.org/legal/epl-2.0/
#
# SPDX-License-Identifier: EPL-2.0
#
set -e \n\
TOOLING="vim nano" \n\
# to avoid shellcheck complaints, but also not break if using "${TOOLING}" because of
# ERROR: 'vim nano' is not a valid world dependency, format is name(@tag)([<>~=]version)
# shellcheck disable=SC2086
if command -v dnf 2> /dev/null; then \n\
  dnf install -y ${TOOLING} \n\
  dnf -y clean all \n\
elif command -v yum 2> /dev/null; then \n\
  yum install -y ${TOOLING} \n\
  yum -y clean all \n\
elif command -v apt-get 2> /dev/null; then \n\
  apt-get update \n\
  apt-get install -y ${TOOLING} \n\
  apt-get clean \n\
  rm -rf /var/lib/apt/lists/* \n\
elif command -v apk 2> /dev/null; then \n\
  apk add --no-cache ${TOOLING} \n\
fi'>/tmp/install-editor-tooling.sh

RUN chown 0:0 /tmp/install-editor-tooling.sh

USER 0 
RUN chmod +x /tmp/install-editor-tooling.sh
RUN /tmp/install-editor-tooling.sh && rm -f /tmp/install-editor-tooling.sh


CMD ["tail" "-f" "/dev/null"]
ENTRYPOINT ["/entrypoint.sh"]
WORKDIR /projects
ENV HOME=/home/user
USER 10001
