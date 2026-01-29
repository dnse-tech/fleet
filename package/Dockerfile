ARG BUILD_ENV=dapper
ARG ARCH

FROM --platform=linux/$ARCH registry.suse.com/bci/bci-base:15.7 AS base
COPY package/log.sh /usr/bin/
RUN zypper rm -y container-suseconnect && \
    zypper -n update && \
    zypper -n install --no-recommends openssh-clients git-core curl && \
    zypper -n clean -a && \
    rm -fr /var/log/zypp* /usr/share/doc && \
    # Install tini from GitHub (not available in zypper for s390x)
    TINI_ARCH=$(uname -m | sed 's/x86_64/amd64/' | sed 's/aarch64/arm64/') && \
    curl -fsSL "https://github.com/krallin/tini/releases/download/v0.19.0/tini-static-${TINI_ARCH}" -o /sbin/tini && \
    chmod +x /sbin/tini

FROM base AS copy_dapper
ONBUILD ARG ARCH
ONBUILD COPY bin/fleetcontroller-linux-$ARCH /usr/bin/fleetcontroller
ONBUILD COPY bin/fleet-linux-$ARCH /usr/bin/fleet

FROM base AS copy_buildx
ONBUILD ARG TARGETARCH
ONBUILD COPY bin/fleetcontroller-linux-$TARGETARCH /usr/bin/fleetcontroller
ONBUILD COPY bin/fleet-linux-$TARGETARCH /usr/bin/fleet

FROM base AS copy_goreleaser
ONBUILD ARG ARCH
ONBUILD COPY fleetcontroller-linux-$ARCH /usr/bin/fleetcontroller
ONBUILD COPY fleet-linux-$ARCH /usr/bin/fleet

FROM copy_${BUILD_ENV}
RUN useradd -u 1000 user
USER 1000
ENTRYPOINT ["tini", "--"]
CMD ["fleetcontroller"]
