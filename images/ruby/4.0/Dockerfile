FROM ruby:4.0-slim@sha256:3d2d07ec3c267107a2a253327c108c1e5b74ea84cfa5ab125f127beb86dccd86

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    ca-certificates \
    watch \
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

