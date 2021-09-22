FROM ubuntu:focal
MAINTAINER Hannu Kumpula <hannu.kumpula@geniem.com>
## Based from Onni Hakala's debian version

# Build arguments
ARG S6_OVERLAY_VERSION=v2.2.0.3
ARG S6_OVERLAY_SHA256=a7076cf205b331e9f8479bbb09d9df77dbb5cd8f7d12e9b74920902e0c16dd98
ARG GOSU_VERSION=1.11

# Finland is quite nice place to live.
# Instead of forking this you should move your living address here.
ENV TZ="Europe/Helsinki" \
    DEBIAN_FRONTEND=noninteractive

RUN set -x \
    # Add all deb-src addresses
    && echo "deb http://mirrors.nic.funet.fi/ubuntu/ focal main\n\
    deb-src http://mirrors.nic.funet.fi/ubuntu/ focal main\n\
    deb http://archive.ubuntu.com/ubuntu focal main restricted universe\n\
    deb http://mirrors.nic.funet.fi/ubuntu/ focal-updates main\n\
    deb-src http://mirrors.nic.funet.fi/ubuntu/ focal-updates main" > /etc/apt/sources.list \
    # Update sources and upgrade all packages
    && chown -R _apt:root /var/lib/apt/lists \
    && apt-get update -y \
    && apt-get upgrade -y \
    && apt-get -y install dpkg-dev \
    
    && apt-get -y install apt-utils apt-transport-https \
    ##
    # Rebuild pam with because pam_env buffersize too low in focal repo version

    
    # Install ca-certificates
    && apt-get install -y --no-install-recommends curl ca-certificates gpg dirmngr gpg-agent \
    # This folder is in $PATH by default but isn't created by default
    && mkdir -p /usr/local/sbin \
    && cd /usr/local/sbin \
    # Install sha256sum validator to check that we download the right files
    && curl -L -o validate_sha256sum https://gist.githubusercontent.com/Hooace/94f42d2d107f4e4cf13bb721c5c63218/raw/d3ef37ab4a653e1b7655df55dfeadd54e0bacf84/validate_sha256sum \
    # This is semi meta but validate that our validator is valid
    && sha256sum validate_sha256sum | grep 0f7b790036f7cd00610cbe9e79c5b6b42d5b0e02beaff9549bdc43fc99910709 \
    && echo "Success: validate_sha256sum matches provided sha256sum" || exit 1 \
    && chmod +x validate_sha256sum \
    # Give execution rights to all scripts which we downloaded
    && chmod a+x * 
    ##
    # Add S6-overlay to use S6 process manager
    # source: https://github.com/just-containers/s6-overlay/#the-docker-way
    ##
ADD https://github.com/just-containers/s6-overlay/releases/download/$S6_OVERLAY_VERSION/s6-overlay-amd64.tar.gz /tmp/
RUN validate_sha256sum /tmp/s6-overlay-amd64.tar.gz ${S6_OVERLAY_SHA256}
RUN tar xzf /tmp/s6-overlay-amd64.tar.gz -C / --exclude='./bin' && tar xzf /tmp/s6-overlay-amd64.tar.gz -C /usr ./bin


    # Add default timezone
    ## TODO: could not get env working so tz is now hardcoded
RUN apt-get install -y tzdata \
    &&  /bin/ln -fs /usr/share/zoneinfo/${TZ} /etc/localtime \

    # Install envsubst command for replacing config files in system startup
    # - it needs libintl package
    # - only weights 100KB combined with it's libraries
    && apt-get install -y gettext \
    && mv /usr/bin/envsubst /usr/local/sbin/envsubst \

    ##
    # Install gosu to fix misbehaving su in some environments
    ##
    && dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')" \
    && curl -L -o /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch" \
    && curl -L -o /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --keyserver keyserver.ubuntu.com --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
    && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
    && rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc \
    && chmod +x /usr/local/bin/gosu \
    && gosu nobody true \

    ##
    # Create a few aliases
    # - I didn't figure out how to load aliases into sh shell with docker so we add scripts instead
    ##

    # ll
    && echo "#!/bin/sh \nls -lah \"\$@\"" > /usr/local/bin/ll \
    # la
    && echo "#!/bin/sh \nls -A \"\$@\"" > /usr/local/bin/la \
    # l
    && echo "#!/bin/sh \nls -CF \"\$@\"" > /usr/local/bin/l \

    && chmod a+x /usr/local/bin/* \

    # Cleanup
    && apt-get remove --purge -y $(apt-mark showauto) \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/log/apt/*

ENTRYPOINT ["/init"]
