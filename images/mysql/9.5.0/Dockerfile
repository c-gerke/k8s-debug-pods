FROM mysql:9.5.0-oracle@sha256:57c892ddeb17e6c7dddf219914db017b2cf3e2797806a75424444b83622cc8ef

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]

