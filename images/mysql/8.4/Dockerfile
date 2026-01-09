FROM mysql:8.4.7-oracle@sha256:90544b3775490579867a30988d48f0215fc3b88d78d8d62b2c0d96ee9226a2b7

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
