FROM mysql:8.4.7-oracle@sha256:5cdee9be17b6b7c804980be29d1bb0ba1536c7afaaed679fe0c1578ea0e3c233

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
