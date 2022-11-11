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

RUN rustup target add wasm32-unknown-unknown
RUN cargo install cargo-watch soroban-cli sccache

ARG DEBIAN_FRONTEND=noninteractive
RUN sudo apt-get update && sudo apt-get install -y binaryen

RUN mkdir -p .local/bin
COPY --link --from=squirtle /app/sq /app/sqweb .local/bin/
COPY --from=stellar /usr/local/bin/soroban-rpc /usr/local/bin/

ENV PATH="/workspace/.local/bin:${PATH}" \
    RUSTC_WRAPPER=sccache \
    SCCACHE_CACHE_SIZE=5G \
    SCCACHE_DIR=/workspace/.sccache

ENV SOROBAN_RPC_URL: "http://localhost:8000/soroban/rpc" \
    SOROBAN_NETWORK_PASSPHRASE: "Test SDF Future Network ; October 2022"
