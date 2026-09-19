FROM ruby:3.4-slim@sha256:9d50d98e61ccbe4f1ef436349911e09b53c42a00364bcd3bda6ac107abc29528

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    ca-certificates \
    procps \
    curl \
    wget \
    git \
    vim \
    build-essential \
    libssl-dev \
    libreadline-dev \
    zlib1g-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* \
           /tmp/* \
           /var/tmp/* \
           /var/cache/debconf/* \
           /var/lib/dpkg/status-old \
           /var/cache/apt/*.bin

CMD ["/bin/bash"]
