FROM mysql:8.4.11-oracle@sha256:0744ee5ef89ce6ccfa13de3e579fe6b9e27f93dd70da9c06d2c908b1b193fb8d

RUN microdnf install -y \
    procps-ng \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
