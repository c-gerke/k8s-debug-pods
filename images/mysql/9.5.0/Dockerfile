FROM mysql:9.5.0-oracle@sha256:c9f0c66a87356518d4b30dbc065eb4567e4a04aff4d0ff194dea0973e5985b57

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]

