FROM debian:bookworm-slim@sha256:56ff6d36d4eb3db13a741b342ec466f121480b5edded42e4b7ee850ce7a418ee

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    wget \
    dnsutils \
    iputils-ping \
    net-tools \
    iproute2 \
    telnet \
    netcat-traditional && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* \
           /tmp/* \
           /var/tmp/* \
           /var/cache/debconf/* \
           /var/lib/dpkg/status-old \
           /var/cache/apt/*.bin

CMD ["/bin/bash"]

