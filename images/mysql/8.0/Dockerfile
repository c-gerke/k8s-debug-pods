FROM mysql:8.0.45-debian@sha256:1ae7ba07e714856c61aa0d559a8f1a788eb8f25bd0b5b71738a1bdd6798bc022

RUN rm -f /etc/apt/sources.list.d/mysql.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* \
           /tmp/* \
           /var/tmp/* \
           /var/cache/debconf/* \
           /var/lib/dpkg/status-old \
           /var/cache/apt/*.bin

CMD ["/bin/bash"]
