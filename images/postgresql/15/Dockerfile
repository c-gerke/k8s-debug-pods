FROM debian:bookworm-slim@sha256:56ff6d36d4eb3db13a741b342ec466f121480b5edded42e4b7ee850ce7a418ee

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    wget \
    gnupg \
    lsb-release && \
    sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && \
    curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
    postgresql-client-15 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* \
           /tmp/* \
           /var/tmp/* \
           /var/cache/debconf/* \
           /var/lib/dpkg/status-old \
           /var/cache/apt/*.bin

CMD ["/bin/bash"]
