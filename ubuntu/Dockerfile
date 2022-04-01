FROM ubuntu:focal
MAINTAINER Hannu Kumpula <hannu.kumpula@geniem.com>

ARG S6_VERSION="v1.21.2.0"

# adds file from the shinsenter/s6-overlay image
COPY --from=shinsenter/s6-overlay / /
ENTRYPOINT ["/init"]

# Finland is quite nice place to live.
# Instead of forking this you should move your living address here.
ENV TZ="Europe/Helsinki" \
    DEBIAN_FRONTEND=noninteractive

RUN set -x \
    # Update sources and upgrade all packages
    && apt-get update -y \
    && apt-get upgrade -y \
    && apt-get -y install apt-utils \
    # Install ca-certificates
    && apt-get install -y --no-install-recommends curl ca-certificates \
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
    && chmod a+x * \
    # Add default timezone
    ## TODO: could not get env working so tz is now hardcoded
    && apt-get install -y tzdata \
    &&  /bin/ln -fs /usr/share/zoneinfo/${TZ} /etc/localtime \
    ##
    # Install envsubst command for replacing config files in system startup
    # - it needs libintl package
    # - only weights 100KB combined with it's libraries
    ##
    && apt-get install -y gettext \
    && mv /usr/bin/envsubst /usr/local/sbin/envsubst \
    ##
    # Install gosu to fix misbehaving su in some environments
    ##
    && apt-get install gosu \
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
    ##
    # Cleanup
    ##
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/log/apt/*

ENTRYPOINT ["/init"]
