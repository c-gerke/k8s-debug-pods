FROM mysql:9.7.0-oracle@sha256:c11782aa2a96624c1efc121768641d96954faa136d6aa82751b032d8c426ffbc

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
