FROM ruby:3.4-slim@sha256:384bff5c64f366283c55cb5d42fd339025d4d44741ed7e7c25923e5ab27f79e6

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
