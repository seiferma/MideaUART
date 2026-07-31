FROM alpine AS go
ARG GO_VERSION=1.26.3
ARG TARGETARCH
RUN wget -O /tmp/go.tar.gz https://go.dev/dl/go${GO_VERSION}.linux-${TARGETARCH}.tar.gz && \
    tar -xvzf /tmp/go.tar.gz -C /tmp
ENV PATH="$PATH:/tmp/go/bin"
RUN go install golang.org/x/tools/gopls@latest && \
    go install github.com/go-delve/delve/cmd/dlv@latest && \
    go install honnef.co/go/tools/cmd/staticcheck@2026.1 && \
    go install golang.org/x/tools/cmd/goimports@latest && \
    go install github.com/jandelgado/gcov2lcov@latest

FROM alpine AS rtk
ARG TARGETARCH
RUN if [[ "${TARGETARCH}" == "arm64" ]]; then export URL_PART="aarch64"; elif [[ "${TARGETARCH}" == "amd64" ]]; then export URL_PART="x86_64"; else export URL_PART="UNKNOWN"; fi && \
    wget -O /tmp/rtk.tar.gz https://github.com/rtk-ai/rtk/releases/latest/download/rtk-$URL_PART-unknown-linux-gnu.tar.gz && \
    tar -xvzf /tmp/rtk.tar.gz -C /tmp && \
    chmod 755 /tmp/rtk

FROM alpine AS opencode
ARG TARGETARCH
RUN if [[ "${TARGETARCH}" == "arm64" ]]; then export URL_PART="arm64"; elif [[ "${TARGETARCH}" == "amd64" ]]; then export URL_PART="x64"; else export URL_PART="UNKNOWN"; fi && \
    wget -O /tmp/oc.tar.gz https://github.com/anomalyco/opencode/releases/latest/download/opencode-linux-$URL_PART.tar.gz && \
    tar -xvzf /tmp/oc.tar.gz -C /tmp && \
    chmod 755 /tmp/opencode



FROM python:3-trixie

# install dependencies from os
RUN apt update && \
    apt install -y openssh-client make ca-certificates gcc g++ docker-cli && \
    apt install -y ripgrep grep && \
    rm -rf /var/lib/apt/lists/*

# add vscode user + sudo
RUN adduser --disabled-password vscode && \
    mkdir /home/vscode/.local && \
    chown vscode:vscode /home/vscode/.local && \
    echo 'vscode ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# give vscode user access to docker socket
RUN --mount=type=bind,src=docker_gid,dst=/tmp/docker_gid \
    export DOCKER_GID=$(head -n 1 /tmp/docker_gid | grep -o '[0-9]\+') && \
    addgroup --gid $DOCKER_GID dockerhostgroup || true && \
    adduser vscode $(getent group $DOCKER_GID | cut -d: -f1)

# install platform.io
RUN pip3 install -U platformio  && \
    pio upgrade  && \
    pio remote update || true

# Install AI stuff
COPY --from=opencode /tmp/opencode /usr/local/bin/
COPY --from=rtk /tmp/rtk /usr/local/bin/
ENV RTK_TELEMETRY_DISABLED=1

ENTRYPOINT ["/bin/bash"]