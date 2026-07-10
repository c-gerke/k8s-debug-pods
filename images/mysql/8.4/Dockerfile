FROM mysql:8.4.10-oracle@sha256:c831a0f11348d402b43d77453e17d770be2eef356615a2823fe0f5a0d6c8b9af

RUN microdnf install -y \
    procps-ng \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
