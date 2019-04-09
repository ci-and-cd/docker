# Dockerfile to create an environment that contains the Nix package manager.

FROM fedora:29



ARG IMAGE_ARG_NIX_VERSION
ARG IMAGE_ARG_NIX_DIGEST


#${ARIA2C_DOWNLOAD} -d ${out_dir} -o ${out_file} ${url}
# wget -q -O ${out_dir}/${out_file} ${url}
# curl -o ${out_dir}/${out_file} ${url}
ENV ARIA2C_DOWNLOAD aria2c --file-allocation=none -c -x 10 -s 10 -m 0 --console-log-level=notice --log-level=notice --summary-interval=0


# Create a user with id 1000 to avoid premission problem when mount host's directory into container
# Install openssl to enable HTTPS support in wget.
RUN set -ex \
  && echo ===== Install libs and tools ===== \
  && echo 'dnf can find best mirror automatically' \
  && echo 'dnf update also do upgrade' \
  && dnf -y update \
  && dnf -y install aria2 bzip2 ca-certificates curl git gnupg2 httpie jq redhat-lsb-core lzma nano openssh-clients openssl sudo tar unzip vim wget xz zip \
  && dnf -y install bc bind-utils gawk iproute iproute-doc net-tools perf procps socat strace telnet tcpdump \
  && dnf -y autoremove \
  && dnf -y clean all \
  && echo ===== Create user ===== \
  && groupadd --gid 1000 fedora \
  && echo 'create home directory manually will prevent os copying any file from skel directory into it' \
  && useradd -c "fedora user" --home-dir /home/fedora --shell /bin/bash -g fedora -m --uid 1000 fedora && chown -R fedora:fedora /home/fedora \
  && usermod -a -G root fedora \
  && echo "fedora ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/fedora \
  && chmod 0440 /etc/sudoers.d/fedora


WORKDIR /home/fedora
USER    fedora
ENV HOME=/home/fedora \
	SHELL=/bin/bash \
	USER=fedora


COPY --chown=fedora:fedora docker /

# see: https://github.com/NixOS/docker/blob/master/Dockerfile
# see: https://github.com/nix-community/docker-nix/blob/master/Dockerfile

#${ARIA2C_DOWNLOAD} https://nixos.org/releases/nix/nix-${IMAGE_ARG_NIX_VERSION:-2.0.4}/nix-${IMAGE_ARG_NIX_VERSION:-2.0.4}-x86_64-linux.tar.bz2
#echo "${IMAGE_ARG_NIX_DIGEST:-d6db178007014ed47ad5460c1bd5b2cb9403b1ec543a0d6507cb27e15358341f}  nix-${IMAGE_ARG_NIX_VERSION:-2.0.4}-x86_64-linux.tar.bz2" | sha256sum -c
# Download Nix and install it into the system.
RUN set -ex \
  && sudo chown root:root /etc/profile && sudo chmod 0644 /etc/profile \
  && if [ ! -f nix-${IMAGE_ARG_NIX_VERSION:-2.2.1}-x86_64-linux.tar.bz2 ]; then ${ARIA2C_DOWNLOAD} https://nixos.org/releases/nix/nix-${IMAGE_ARG_NIX_VERSION:-2.2.1}/nix-${IMAGE_ARG_NIX_VERSION:-2.2.1}-x86_64-linux.tar.bz2; fi \
  && echo "${IMAGE_ARG_NIX_DIGEST:-e229e28f250cad684c278c9007b07a24eb4ead239280c237ed2245871eca79e0}  nix-${IMAGE_ARG_NIX_VERSION:-2.2.1}-x86_64-linux.tar.bz2" | sha256sum -c \
  && tar xjf nix-${IMAGE_ARG_NIX_VERSION:-2.2.1}-x86_64-linux.tar.bz2 && rm -f nix-${IMAGE_ARG_NIX_VERSION:-2.2.1}-x86_64-linux.tar.bz2 \
  && sudo groupadd --gid 30000 --system nixbld \
  && for i in $(seq 1 30); do sudo useradd --system --home-dir /var/empty --comment "Nix build user $i" --uid $((30000 + i)) --groups nixbld nixbld$i ; done \
  && sudo mkdir -m 0755 /etc/nix \
  && echo 'sandbox = false' | sudo tee /etc/nix/nix.conf \
  && mkdir -p /home/fedora/.config/nix && echo 'sandbox = false' | tee /home/fedora/.config/nix/nix.fonf \
  && sudo mkdir -p /root/.config/nix && echo 'sandbox = false' | sudo tee /root/.config/nix/nix.fonf \
  && sudo mkdir -m 0755 /nix && sudo chown fedora:fedora /nix \
  && nix-*-x86_64-linux/install \
  && sudo mv nix-*-x86_64-linux /root/ && sudo bash -c "cd /root && nix-*-x86_64-linux/install" \
  && sudo chown -R fedora:fedora /nix && sudo chown -R fedora:fedora /home/fedora \
  && sudo chown root:root /home /etc \
  && sudo ln -s /nix/var/nix/profiles/default/etc/profile.d/nix.sh /etc/profile.d/ \
  && echo ". /home/fedora/.nix-profile/etc/profile.d/nix.sh" | tee -a /home/fedora/.bashrc \
  && echo ". /home/fedora/.nix-profile/etc/profile.d/nix.sh" | tee -a /home/fedora/.profile \
  && USER=fedora . /home/fedora/.profile \
  && nix-env -iA \
       nixpkgs.bashInteractive \
       nixpkgs.cacert \
       nixpkgs.coreutils \
       nixpkgs.gitMinimal \
       nixpkgs.gnutar \
       nixpkgs.gzip \
       nixpkgs.iana-etc \
       nixpkgs.xz \
  && true \
  && sudo rm -fr /root/nix-*-x86_64-linux* \
  && sudo chown 0 /nix/var/nix/profiles/per-user/root \
  && sudo chown 0 /nix/var/nix/gcroots/per-user/root \
  && sudo su - root sh -c 'exit' \
  && /nix/var/nix/profiles/default/bin/nix-collect-garbage --delete-old \
  && /nix/var/nix/profiles/default/bin/nix-store --optimise \
  && /nix/var/nix/profiles/default/bin/nix-store --verify --check-contents \
  && if [ ! -L /nix/var/nix/profiles/default/etc/ssl ]; then \
       sudo ln -s $(find /nix/store -type d -name "*nss-cacert*")/etc/ssl /nix/var/nix/profiles/default/etc/ssl; \
       sudo chown -h fedora:fedora /nix/var/nix/profiles/default/etc/ssl; \
     fi

# nixpkgs.cacert fixes following issue
# fatal: unable to access 'https://github.com/owner/repo.git/': error setting certificate verify locations:
#   CAfile: /nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt
#   CApath: none
RUN set -ex \
  && sudo git config --system http.sslcainfo /nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt \
  && sudo git config --global http.sslverify "false"

ONBUILD ENV \
    ENV=/etc/profile \
    USER=fedora \
    PATH=/nix/var/nix/profiles/default/bin:/nix/var/nix/profiles/default/sbin:/bin:/sbin:/usr/bin:/usr/sbin \
    GIT_SSL_CAINFO=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt \
    NIX_SSL_CERT_FILE=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt

ENV \
    ENV=/etc/profile \
    USER=fedora \
    PATH=/nix/var/nix/profiles/default/bin:/nix/var/nix/profiles/default/sbin:/bin:/sbin:/usr/bin:/usr/sbin \
    GIT_SSL_CAINFO=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt \
    NIX_SSL_CERT_FILE=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt \
    NIX_PATH=/nix/var/nix/profiles/per-user/fedora/channels
