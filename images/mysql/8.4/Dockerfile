FROM mysql:8.4.11-oracle@sha256:b3b90af2a6552ae30c266fdb7d5dd55f3afb72404bb78d37fe8a23eb857fd3fb

RUN microdnf install -y \
    procps-ng \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
