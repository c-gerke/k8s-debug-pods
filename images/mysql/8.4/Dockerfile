FROM mysql:8.4.8-oracle@sha256:c4678fed620278d29a6ef031b6aba9a31b1bc8f48e46bd56e9706943db2bc0c1

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
