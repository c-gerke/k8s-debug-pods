FROM ruby:3.4-slim@sha256:09125b2be429d65151802d2a04a60e0817fb8f380195fedbc3e3d8b51b12c049

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
