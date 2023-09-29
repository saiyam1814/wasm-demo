FROM scratch
COPY spin.toml /spin.toml
COPY target/wasm32-wasi/release/wasmdemo.wasm /target/wasm32-wasi/release/wasmdemo.wasm
ENTRYPOINT ["/spin.toml"]
