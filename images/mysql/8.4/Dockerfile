FROM mysql:8.4.8-oracle@sha256:cf6d1c8f2d31c73c878c6f5f60c2bec46701c41d5baf99100888bef5b6578b86

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
