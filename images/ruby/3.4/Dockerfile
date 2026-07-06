FROM ruby:3.4-slim@sha256:634fb60b00d033da01c2062f2f067cd29f4d16a26ed0c0aa37e4e24ae2374628

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
