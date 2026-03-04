FROM mysql:8.4.8-oracle@sha256:a2d126916bc2ba79a890a4bf62d305eb8b68fcbdd35c6e582d529df18faf5ebb

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
