FROM mysql:8.4.7-oracle@sha256:b306273d4d36bc1a7f265130c3dd93c0462253af7634203e569add0403a7b273

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
