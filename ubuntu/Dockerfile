# Dockerfile to create an environment that contains the Nix package manager.

FROM ubuntu:18.04


ARG IMAGE_ARG_APT_MIRROR


#${ARIA2C_DOWNLOAD} -d ${out_dir} -o ${out_file} ${url}
# wget -q -O ${out_dir}/${out_file} ${url}
# curl -o ${out_dir}/${out_file} ${url}
ENV ARIA2C_DOWNLOAD aria2c --file-allocation=none -c -x 10 -s 10 -m 0 --console-log-level=notice --log-level=notice --summary-interval=0


# Create a user with id 1000 to avoid premission problem when mount host's directory into container
# Install openssl to enable HTTPS support in wget.
RUN set -ex \
  && echo ===== Install libs and tools ===== \
  && sed -i "s/http:\/\/archive.ubuntu.com\/ubuntu\//http:\/\/${IMAGE_ARG_APT_MIRROR:-archive.ubuntu.com}\/ubuntu\//g" /etc/apt/sources.list \
  && apt -y update \
  && apt -y upgrade \
  && apt -y install apt-transport-https apt-utils aria2 bzip2 ca-certificates curl git gnupg2 httpie jq lsb-release lzma nano openssh-client openssl software-properties-common sudo tar unzip vim wget xz-utils \
  && apt -q -y autoremove \
  && apt -q -y clean && rm -rf /var/lib/apt/lists/* && rm -f /var/cache/apt/*.bin \
  && echo ===== Create user ===== \
  && groupadd --gid 1000 ubuntu \
  && mkdir -p /home/ubuntu && useradd --home-dir /home/ubuntu --shell /bin/bash -g ubuntu --uid 1000 ubuntu && chown -R ubuntu:ubuntu /home/ubuntu \
  && usermod -a -G root ubuntu \
  && echo "ubuntu ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/ubuntu \
  && chmod 0440 /etc/sudoers.d/ubuntu


WORKDIR /home/ubuntu
USER    ubuntu
ENV HOME=/home/ubuntu \
	SHELL=/bin/bash \
	USER=ubuntu


COPY --chown=ubuntu:ubuntu docker /


#RUN ${ARIA2C_DOWNLOAD} https://nixos.org/releases/nix/nix-2.0.4/nix-2.0.4-x86_64-linux.tar.bz2
# Download Nix and install it into the system.
RUN set -ex \
  && sudo chown root:root /etc/profile && sudo chmod 0644 /etc/profile \
  && if [ ! -f nix-2.0.4-x86_64-linux.tar.bz2 ]; then ${ARIA2C_DOWNLOAD} https://nixos.org/releases/nix/nix-2.0.4/nix-2.0.4-x86_64-linux.tar.bz2; fi \
  && echo "d6db178007014ed47ad5460c1bd5b2cb9403b1ec543a0d6507cb27e15358341f  nix-2.0.4-x86_64-linux.tar.bz2" | sha256sum -c \
  && tar xjf nix-*-x86_64-linux.tar.bz2 \
  && sudo groupadd --gid 30000 --system nixbld \
  && for i in $(seq 1 30); do sudo useradd --system --home-dir /var/empty --comment "Nix build user $i" --uid $((30000 + i)) --groups nixbld nixbld$i ; done \
  && sudo mkdir -m 0755 /nix && sudo chown ubuntu:ubuntu /nix \
  && USER=ubuntu sudo -E sh nix-*-x86_64-linux/install && sudo chown -R ubuntu:ubuntu /nix && sudo chown -R ubuntu:ubuntu /home/ubuntu \
  && sudo ln -s /nix/var/nix/profiles/default/etc/profile.d/nix.sh /etc/profile.d/ \
  && echo ". /home/ubuntu/.nix-profile/etc/profile.d/nix.sh" > /home/ubuntu/.bashrc \
  && echo ". /home/ubuntu/.nix-profile/etc/profile.d/nix.sh" > /home/ubuntu/.profile \
  && USER=ubuntu . /home/ubuntu/.profile \
  && rm -r nix-*-x86_64-linux*
#USER=root sh nix-*-x86_64-linux/install

ONBUILD ENV \
    ENV=/etc/profile \
    USER=ubuntu \
    PATH=/nix/var/nix/profiles/default/bin:/nix/var/nix/profiles/default/sbin:/bin:/sbin:/usr/bin:/usr/sbin \
    GIT_SSL_CAINFO=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt \
    NIX_SSL_CERT_FILE=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt

ENV \
    ENV=/etc/profile \
    USER=ubuntu \
    PATH=/nix/var/nix/profiles/default/bin:/nix/var/nix/profiles/default/sbin:/bin:/sbin:/usr/bin:/usr/sbin \
    GIT_SSL_CAINFO=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt \
    NIX_SSL_CERT_FILE=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt \
    NIX_PATH=/nix/var/nix/profiles/per-user/ubuntu/channels
