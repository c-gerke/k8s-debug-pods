FROM ruby:3.4-slim@sha256:65680693964528b522a38d85ec5065538c7ab9e75268802aab9770f39ee56de0

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
