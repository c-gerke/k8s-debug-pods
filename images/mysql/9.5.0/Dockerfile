FROM mysql:9.5.0-oracle@sha256:5ca0a273ed28c73acaef91da8bf1eca3711bee94bce4c378d42846375e645a72

RUN microdnf install -y \
    curl \
    wget && \
    microdnf clean all

CMD ["/bin/bash"]

