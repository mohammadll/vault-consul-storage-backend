FROM ubuntu:20.04

ENV VAULT_VERSION=1.12.2
ENV HASHICORP_RELEASES=https://releases.hashicorp.com
ENV VAULT_ARCH=amd64

RUN apt-get update && apt-get install -y unzip curl jq wget iputils-ping iproute2 && \
    rm -rf /var/lib/apt/lists/*

RUN useradd --system --home /etc/vault.d --shell /bin/false vault

RUN mkdir -p /etc/vault.d && \
    chown vault:vault /etc/vault.d

COPY vault /bin
RUN vault version
EXPOSE 8200 8201

