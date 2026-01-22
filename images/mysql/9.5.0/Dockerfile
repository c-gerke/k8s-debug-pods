FROM mysql:9.6.0-oracle@sha256:c5467c2e0e1b0363d9b67633dff98d3e0ab5be2fa70f90e12fc4b63dc79a5f73

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]

