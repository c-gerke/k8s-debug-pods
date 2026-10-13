FROM mysql:8.4.11-oracle@sha256:24989179f58856db763f418010c1169fecc944e3ec1ea7e921572d83eb070f70

RUN microdnf install -y \
    procps-ng \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
