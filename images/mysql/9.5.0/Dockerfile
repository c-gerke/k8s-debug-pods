FROM mysql:26.7.0-oracle@sha256:66aec17cd21a956029b83f083b813073859e8355dc1a00e55df6ba02f0e32345

RUN microdnf install -y \
    procps-ng \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
