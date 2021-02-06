FROM debian:bullseye-slim AS build

SHELL ["/bin/bash", "-c"]

RUN apt-get -qq update \
 && apt-get -qq -y install build-essential git &>/dev/null \
 && apt-get -qq clean \
 && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# let Docker cache this between Git revision and testnet version changes
RUN cd /root \
 && git clone https://github.com/kidkal/nimbus-eth2.git \
 && cd nimbus-eth2 \
 && make -j$(nproc) update

# Please note that the commands above have the goal of caching the
# compilation of Nim, but don't depend on the current git revision.
# This means that the cache can become outdated over time and you'll
# start seeing Nim being compiled on every run. If this happens, just
# prune your docker cache to get a fresh up-to-date version of Nim.
ARG NETWORK_NIM_FLAGS
ARG MARCH_NIM_FLAGS

RUN cd /root/nimbus-eth2 \
 && git fetch \
 && git reset --hard origin/v1.0.7-rocketpool \
 && make -j$(nproc) update \
 && make LOG_LEVEL=TRACE NIMFLAGS="-d:insecure -d:testnet_servers_image ${NETWORK_NIM_FLAGS} ${MARCH_NIM_FLAGS}"

# --------------------------------- #
# Starting new image to reduce size #
# --------------------------------- #
FROM debian:bullseye-slim as deploy

SHELL ["/bin/bash", "-c"]

RUN apt-get -qq update \
 && apt-get -qq -y install psmisc &>/dev/null \
 && apt-get -qq clean \
 && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Likely to match the first regular user:group created on the host.
RUN addgroup --gid 1000 user; \
 	adduser --disabled-password --gecos '' --uid 1000 --gid 1000 user;

USER user

# "COPY" creates new image layers, so we cram all we can into one command
COPY --from=build "/root/nimbus-eth2/build/nimbus_*" "/home/user/nimbus-eth2/build/"

MAINTAINER Zahary Karadjov <zahary@status.im>
LABEL description="Nimbus installation that can act as an ETH2 network bootstrap node."

STOPSIGNAL SIGINT

WORKDIR "/home/user/nimbus-eth2/"
ENTRYPOINT ["/home/user/nimbus-eth2/build/nimbus_beacon_node"]
