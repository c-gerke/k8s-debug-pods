FROM mysql:8.4.8-oracle@sha256:f1c7674d8dd3c94f439b0d607792187d55bd861f94e8c6d683b959b577152bdc

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
