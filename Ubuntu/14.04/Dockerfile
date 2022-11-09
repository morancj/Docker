FROM ubuntu:14.04

# Install outdated software
ENV UNVERSIONED_PACKAGES \
  openssh-client

# Install software and cleanup
# Tell apt not to use interactive prompts
RUN export DEBIAN_FRONTEND=noninteractive && \
# update the package database
  apt-get update && \
# Install packages
  apt-get install -y --no-install-recommends \
  ${UNVERSIONED_PACKAGES} \
  && \
# Clean up package cache in this layer
# Remove dependencies which are no longer required
  apt-get --purge autoremove -y && \
# Clean package cache
  apt-get clean -y && \
# Restore interactive prompts
  unset DEBIAN_FRONTEND && \
# Remove cache files
  rm -rf \
  /tmp/* \
  /var/cache/* \
  /var/log/* \
  /var/lib/apt/lists/*

ENTRYPOINT ["/usr/bin/ssh"]
