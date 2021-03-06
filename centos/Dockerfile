# Dockerfile to create an environment that contains the Nix package manager.

FROM centos:7.6.1810



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
  && echo 'yum can find best mirror automatically' \
  && echo 'yum update also do upgrade' && yum -y update \
  && yum install -y epel-release && yum -y update \
  && yum -y install aria2 bzip2 ca-certificates curl git gnupg2 httpie jq redhat-lsb-core lzma nano openssh-clients openssl sudo tar unzip vim wget xz zip \
  && yum -y install bc bind-utils gawk iproute iproute-doc net-tools perf socat strace telnet tcpdump \
  && yum -y autoremove \
  && yum -y clean all \
  && echo ===== Create user ===== \
  && groupadd --gid 1000 centos \
  && echo 'create home directory manually will prevent os copying any file from skel directory into it' \
  && useradd -c "centos user" --home-dir /home/centos --shell /bin/bash -g centos -m --uid 1000 centos && chown -R centos:centos /home/centos \
  && usermod -a -G root centos \
  && echo "centos ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/centos \
  && chmod 0440 /etc/sudoers.d/centos


WORKDIR /home/centos
USER    centos
ENV HOME=/home/centos \
	SHELL=/bin/bash \
	USER=centos


COPY --chown=centos:centos docker /

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
  && mkdir -p /home/centos/.config/nix && echo 'sandbox = false' | tee /home/centos/.config/nix/nix.fonf \
  && sudo mkdir -p /root/.config/nix && echo 'sandbox = false' | sudo tee /root/.config/nix/nix.fonf \
  && sudo mkdir -m 0755 /nix && sudo chown centos:centos /nix \
  && nix-*-x86_64-linux/install \
  && sudo mv nix-*-x86_64-linux /root/ && sudo bash -c "cd /root && nix-*-x86_64-linux/install" \
  && sudo chown -R centos:centos /nix && sudo chown -R centos:centos /home/centos \
  && sudo chown root:root /home /etc \
  && sudo ln -s /nix/var/nix/profiles/default/etc/profile.d/nix.sh /etc/profile.d/ \
  && echo ". /home/centos/.nix-profile/etc/profile.d/nix.sh" | tee -a /home/centos/.bashrc \
  && echo ". /home/centos/.nix-profile/etc/profile.d/nix.sh" | tee -a /home/centos/.profile \
  && USER=centos . /home/centos/.profile \
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
       sudo chown -h centos:centos /nix/var/nix/profiles/default/etc/ssl; \
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
    USER=centos \
    PATH=/nix/var/nix/profiles/default/bin:/nix/var/nix/profiles/default/sbin:/bin:/sbin:/usr/bin:/usr/sbin \
    GIT_SSL_CAINFO=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt \
    NIX_SSL_CERT_FILE=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt

ENV \
    ENV=/etc/profile \
    USER=centos \
    PATH=/nix/var/nix/profiles/default/bin:/nix/var/nix/profiles/default/sbin:/bin:/sbin:/usr/bin:/usr/sbin \
    GIT_SSL_CAINFO=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt \
    NIX_SSL_CERT_FILE=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt \
    NIX_PATH=/nix/var/nix/profiles/per-user/centos/channels
