ARG VCS_REF=master
ARG BUILD_DATE=""
ARG REGISTRY_PATH=redspot

FROM ${REGISTRY_PATH}/base:latest


WORKDIR /builds

ENV SHELL /bin/bash
ENV DEBIAN_FRONTEND=noninteractive

# install tools and dependencies
RUN set -eux; \
	apt-get -y update && \
	apt-get install -y --no-install-recommends \
		zlib1g-dev npm wabt llvm-dev && \
	npm install --ignore-scripts -g yarn && \
# `binaryen` is needed by `cargo-contract` for optimizing Wasm files.
# We fetch the latest release which contains a Linux binary. There is
# currently an issue with not all releases containing a Linux binary
# (https://github.com/WebAssembly/binaryen/issues/4148). Once that one
# is fixed we can always use the latest release again.
	curl --silent https://api.github.com/repos/WebAssembly/binaryen/releases | \
		egrep --only-matching 'https://github.com/WebAssembly/binaryen/releases/download/version_[0-9]+/binaryen-version_[0-9]+-x86_64-linux.tar.gz' | \
		head -n1 | \
		xargs curl -L -O && \
	tar -xvzf binaryen-version_*-x86_64-linux.tar.gz  && \
	rm binaryen-version_*-x86_64-linux.tar.gz && \
	chmod +x binaryen-version_*/bin/wasm-opt && \
	mv binaryen-version_*/bin/wasm-opt /usr/local/bin/ && \
	rm -rf binaryen-version_*/ && \
# Installs the latest common nightly for the listed components,
# adds those components, wasm target and sets the profile to minimal
	rustup toolchain install nightly --target wasm32-unknown-unknown \
		--profile minimal --component rustfmt clippy rust-src && \
	rustup default nightly && \
	cargo install pwasm-utils-cli --bin wasm-prune && \
	cargo install cargo-contract && \
# tried v0.1.5 and the latest master - both fail with https://github.com/hyperledger-labs/solang/issues/314
	cargo install --git https://github.com/hyperledger-labs/solang --tag v0.1.2 && \
# download the latest `substrate-contracts-node` binary
	curl -L "https://gitlab.parity.io/parity/substrate-contracts-node/-/jobs/artifacts/master/raw/artifacts/substrate-contracts-node-linux/substrate-contracts-node?job=build-linux" \
		-o /usr/local/cargo/bin/substrate-contracts-node && \
	chmod +x /usr/local/cargo/bin/substrate-contracts-node && \
# versions
	yarn --version && \
	rustup show && \
	cargo --version && \
	solang --version && \
	substrate-contracts-node --version && \
# cargo clean up
# removes compilation artifacts cargo install creates (>250M)
	rm -rf "${CARGO_HOME}/registry" "${CARGO_HOME}/git" /root/.cache/sccache && \
# apt clean up
	apt-get autoremove -y && \
	apt-get clean && \
	rm -rf /var/lib/apt/lists/*