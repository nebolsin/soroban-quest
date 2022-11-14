# syntax=docker/dockerfile-upstream:master-labs
ARG VARIANT="bullseye"

FROM stellar/quickstart:soroban-dev as stellar

FROM denoland/deno:1.27.2 as squirtle
WORKDIR /app/
ADD https://github.com/tyvdh/soroban-quest--pioneer.git#main ./
RUN deno compile --allow-env --allow-write --allow-read --allow-run --allow-net --unstable --output sq _squirtle/mod.ts
RUN deno compile --allow-net --allow-read --allow-run --unstable --output sqweb _client/webserver.ts


FROM mcr.microsoft.com/devcontainers/rust:1-${VARIANT}
LABEL version="1.0.0"

SHELL ["/bin/bash", "-o", "errexit", "-o", "nounset", "-o", "pipefail", "-c"]

RUN <<-EOF
    case "$(dpkg --print-architecture)" in
    amd64)
        download_architecture="x86_64"
        ;;
    arm64)
        download_architecture="aarch64"
        ;;
    *) echo "(!) Architecture ${architecture} not supported."
        exit 1
        ;;
    esac

    echo "Installing sccache..."
    sccache_version="0.3.0"
    sccache_dist="sccache-v${sccache_version}-${download_architecture}-unknown-linux-musl"

    curl -sSL "https://github.com/mozilla/sccache/releases/download/v${sccache_version}/${sccache_dist}.tar.gz" -o /tmp/sccache.tar.gz
    tar --file="/tmp/sccache.tar.gz" --directory="$CARGO_HOME/bin" --auto-compress --extract --strip-components 1 "${sccache_dist}/sccache"
    rm -rf /tmp/sccache.tar.gz
    chmod +x "$CARGO_HOME/bin/sccache"

    echo "Installing cargo-binstall..."
    curl -sSL "https://github.com/cargo-bins/cargo-binstall/releases/latest/download/cargo-binstall-${download_architecture}-unknown-linux-musl.tgz" -o /tmp/cargo-binstall.tgz
    tar --file="/tmp/cargo-binstall.tgz" --directory="$CARGO_HOME/bin" --auto-compress --extract "cargo-binstall"
    chmod +x "$CARGO_HOME/bin/cargo-binstall"


    echo "Installing binaryen (wasm-opt) ..."
    apt-get update && apt-get install -y binaryen

EOF

ENV RUSTC_WRAPPER=sccache \
    SCCACHE_CACHE_SIZE=5G \
    SCCACHE_DIR=/workspaces/.sccache

RUN <<-EOF
    echo "Installing wasm32-unknown-unknown target..."
    rustup target add wasm32-unknown-unknown

    echo "Installing common Rust dependencies..."
    rustup component add rls rust-analysis rust-src rustfmt clippy 2>&1

    echo "Installing Cargo tools..."
    cargo binstall --no-confirm cargo-edit cargo-watch cargo-workspace

    echo "Installing Soroban CLI..."
    cargo binstall --no-confirm soroban-cli
EOF

COPY --from=squirtle /app/sq /app/sqweb /workspaces/.local/bin/
COPY --from=stellar /usr/local/bin/soroban-rpc /workspaces/.local/bin

ENV PATH="/workspaces/.local/bin:${PATH}" \
    SOROBAN_RPC_URL="http://localhost:8000/soroban/rpc" \
    SOROBAN_NETWORK_PASSPHRASE="Test SDF Future Network ; October 2022"
