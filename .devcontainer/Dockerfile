FROM ubuntu:22.04

ARG DEBIAN_FRONTEND=noninteractive

ARG SOLANA_CLI="v2.1.0"
ARG ANCHOR_CLI="v0.31.1"
ARG NODE_VERSION="v21.7.0"

ENV HOME="/root"
ENV PATH="${HOME}/.cargo/bin:${PATH}"
ENV PATH="${HOME}/.local/share/solana/install/active_release/bin:${PATH}"
ENV PATH="${HOME}/.nvm/versions/node/${NODE_VERSION}/bin:${PATH}"

# Install base utilities.
RUN mkdir -p /workdir/anchor-devbox && mkdir -p /tmp && \
    apt-get update -qq && apt-get upgrade -qq && apt-get install -qq \
    build-essential git curl wget jq pkg-config python3-pip \
    libssl-dev libudev-dev

# Install rust.
RUN curl "https://sh.rustup.rs" -sfo rustup.sh && \
    sh rustup.sh -y && \
    rustup component add rustfmt clippy

# Install node / npm / yarn.
RUN curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
ENV NVM_DIR="${HOME}/.nvm"
RUN . $NVM_DIR/nvm.sh && \
    nvm install ${NODE_VERSION} && \
    nvm use ${NODE_VERSION} && \
    nvm alias default node && \
    npm install -g yarn

# Install Solana tools.
RUN sh -c "$(curl -sSfL https://release.anza.xyz/${SOLANA_CLI}/install)"

# Install anchor.
RUN cargo install --git https://github.com/coral-xyz/anchor --tag ${ANCHOR_CLI} anchor-cli --locked

# Build a dummy program to bootstrap the BPF SDK (doing this speeds up builds).
RUN mkdir -p /tmp && cd /tmp && anchor init dummy && cd dummy && anchor build

# Generate a default Solana keypair
RUN solana-keygen new --no-bip39-passphrase 

# Initialize the anchor project in /workdir/anchor-devbox/
WORKDIR /workdir/anchor-devbox
RUN anchor init myproject && \
    mv myproject/* . && \
    mv myproject/.* . 2>/dev/null || true && \
    rmdir myproject

# Create an entrypoint script
RUN echo '#!/bin/bash\nexec "$@"' > /usr/local/bin/entrypoint.sh && \
    chmod +x /usr/local/bin/entrypoint.sh

# Set the working directory to the anchor project
WORKDIR /workdir/anchor-devbox

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
CMD ["bash"]