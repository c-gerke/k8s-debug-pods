FROM ruby:3.3-slim@sha256:a67a4b6c46fa28f64473336e5624d2443e0651b6a8e680cb7f17001b17846ffa

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    ca-certificates \
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
