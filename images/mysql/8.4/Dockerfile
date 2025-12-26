FROM mysql:8.4.7-oracle@sha256:1f5b0aca09cfa06d9a7b89b28d349c1e01ba0d31339a4786fbcb3d5927070130

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
