FROM mysql:8.4.7-oracle@sha256:0426ec38c7a10aa45ba383887df7878f74ee70e2fd589c7b69207f3577901903

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]
