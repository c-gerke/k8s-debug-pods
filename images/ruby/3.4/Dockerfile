FROM ruby:3.4-slim@sha256:dcc535e858eeeac1f70eaf8c39ef7ee257b183a9e495cc5a9ef3e340eadf2d3d

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
