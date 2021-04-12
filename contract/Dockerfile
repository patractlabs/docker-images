ARG VCS_REF=master
ARG BUILD_DATE=""
ARG REGISTRY_PATH=redspot

FROM ${REGISTRY_PATH}/base:latest


WORKDIR /builds

ENV SHELL /bin/bash
ENV DEBIAN_FRONTEND=noninteractive

# install tools and dependencies
RUN set -eux; \
# binaryen is needed by cargo-contract for optimizing Wasm files
# we fetch the latest version number from the GitHub API and use that release
	curl --silent https://api.github.com/repos/WebAssembly/binaryen/releases/latest | \
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
# versions
	yarn --version && \
	rustup show && \
	cargo --version && \
	cargo contract --version && \
# cargo clean up
# removes compilation artifacts cargo install creates (>250M)
	rm -rf "${CARGO_HOME}/registry" "${CARGO_HOME}/git" /root/.cache/sccache && \
# apt clean up
	apt-get autoremove -y && \
	apt-get clean && \
	rm -rf /var/lib/apt/lists/*