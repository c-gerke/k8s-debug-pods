FROM mysql:8.4.8-oracle@sha256:7fcf7bcd3fa703ff23a2b998e4ed9077e5db570bcc4468459902e33b23d2841d

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
