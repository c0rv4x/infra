ARG NODE_ARCH_RELEASE
ARG PYCA_RELEASE
FROM ghcr.io/pyca/static-nodejs-${NODE_ARCH_RELEASE} as staticnodejs
FROM quay.io/pypa/${PYCA_RELEASE}
LABEL org.opencontainers.image.authors="Python Cryptographic Authority"

WORKDIR /root

# Combine installation commands for different architectures in a single RUN block
RUN \
  if stat /etc/redhat-release 1>&2 2>/dev/null; then \
    if [ $(uname -m) = "x86_64" ]; then \
      yum -y install binutils perl-IPC-Cmd --setopt=tsflags=nodocs && \
      yum -y clean all && \
      rm -rf /var/cache/yum; \
    elif [ $(uname -m) = "aarch64" ]; then \
      yum -y install perl-IPC-Cmd --setopt=tsflags=nodocs && \
      yum -y clean all && \
      rm -rf /var/cache/yum; \
    fi; \
  fi

# Copy static nodejs binary from the first stage
COPY --from=staticnodejs /out/ /staticnode/

# Add scripts and patches
ADD install_openssl.sh /root/install_openssl.sh
ADD openssl-version.sh /root/openssl-version.sh
ADD list-util-pairs-25367.patch /root/list-util-pairs-25367.patch

# Install OpenSSL
RUN ./install_openssl.sh

# Install Rust with a minimal profile
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y --default-toolchain stable --profile minimal

# Set PATH environment variable for Cargo (Rust)
ENV PATH="/root/.cargo/bin:$PATH"
