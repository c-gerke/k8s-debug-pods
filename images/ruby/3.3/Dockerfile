FROM ruby:3.3-slim@sha256:37d45ed5edf175280231e7361ac53bdccdcd476147a74d70cd44b8a7704ef85e

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
