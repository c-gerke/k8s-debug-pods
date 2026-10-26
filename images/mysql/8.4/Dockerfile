FROM mysql:8.4.13-oracle@sha256:76f0358884cb92d9a086d24e5671c38048d3fef40f2588703d73eb55392d6b69

RUN microdnf install -y \
    procps-ng \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
