FROM mysql:8.4.9-oracle@sha256:c36050afdca850f23cef85703f84c7531a5ae155a11b5ee1c60acb09937c4084

RUN microdnf install -y \
    watch \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
