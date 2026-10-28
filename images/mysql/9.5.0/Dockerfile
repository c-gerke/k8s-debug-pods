FROM mysql:9.7.4-oracle@sha256:86b390ef642f89a0157d6215dee33ddc928b8b0ecab173117f56f6c40cb24eab

RUN microdnf install -y \
    procps-ng \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
