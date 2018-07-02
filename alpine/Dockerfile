# Dockerfile to create an environment that contains the Nix package manager.

FROM alpine:3.7


ARG IMAGE_ARG_ALPINE_MIRROR


#${ARIA2C_DOWNLOAD} -d ${out_dir} -o ${out_file} ${url}
# wget -q -O ${out_dir}/${out_file} ${url}
# curl -o ${out_dir}/${out_file} ${url}
ENV ARIA2C_DOWNLOAD aria2c --file-allocation=none -c -x 10 -s 10 -m 0 --console-log-level=notice --log-level=notice --summary-interval=0


# Create a user with id 1000 to avoid premission problem when mount host's directory into container
# Install openssl to enable HTTPS support in wget.
RUN set -ex \
  && echo ===== Install libs and tools ===== \
  && echo "http://${IMAGE_ARG_ALPINE_MIRROR:-dl-cdn.alpinelinux.org}/alpine/v3.7/main" > /etc/apk/repositories \
  && echo "http://${IMAGE_ARG_ALPINE_MIRROR:-dl-cdn.alpinelinux.org}/alpine/v3.7/community" >> /etc/apk/repositories \
  && echo "http://${IMAGE_ARG_ALPINE_MIRROR:-dl-cdn.alpinelinux.org}/alpine/edge/testing/" >> /etc/apk/repositories \
  && apk add --update aria2 bash ca-certificates curl openssl shadow sudo tar unzip wget xz \
  && rm -r /var/cache/apk/* \
  && echo ===== Create user ===== \
  && addgroup -g 1000 alpine \
  && adduser -h /home/alpine -s /bin/bash -G alpine -D -u 1000 alpine \
  && usermod -a -G root alpine \
  && echo "alpine ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/alpine \
  && chmod 0440 /etc/sudoers.d/alpine

WORKDIR /home/alpine
USER    alpine
ENV HOME=/home/alpine \
	SHELL=/bin/bash \
	USER=alpine

#RUN ${ARIA2C_DOWNLOAD} https://nixos.org/releases/nix/nix-2.0.4/nix-2.0.4-x86_64-linux.tar.bz2
# Download Nix and install it into the system.
RUN set -ex \
  && if [ ! -f nix-2.0.4-x86_64-linux.tar.bz2 ]; then ${ARIA2C_DOWNLOAD} https://nixos.org/releases/nix/nix-2.0.4/nix-2.0.4-x86_64-linux.tar.bz2; fi \
  && echo "d6db178007014ed47ad5460c1bd5b2cb9403b1ec543a0d6507cb27e15358341f  nix-2.0.4-x86_64-linux.tar.bz2" | sha256sum -c \
  && tar xjf nix-*-x86_64-linux.tar.bz2 \
  && sudo addgroup -g 30000 -S nixbld \
  && for i in $(seq 1 30); do sudo adduser -S -D -h /var/empty -g "Nix build user $i" -u $((30000 + i)) -G nixbld nixbld$i ; done \
  && sudo mkdir -m 0755 /nix && sudo chown alpine:alpine /nix \
  && USER=alpine sudo -E sh nix-*-x86_64-linux/install && sudo chown -R alpine:alpine /nix && sudo chown -R alpine:alpine /home/alpine \
  && sudo ln -s /nix/var/nix/profiles/default/etc/profile.d/nix.sh /etc/profile.d/ \
  && echo ". /home/alpine/.nix-profile/etc/profile.d/nix.sh" > /home/alpine/.profile \
  && USER=alpine . /home/alpine/.profile \
  && rm -r nix-*-x86_64-linux*
#USER=root sh nix-*-x86_64-linux/install

ONBUILD ENV \
    ENV=/etc/profile \
    USER=alpine \
    PATH=/nix/var/nix/profiles/default/bin:/nix/var/nix/profiles/default/sbin:/bin:/sbin:/usr/bin:/usr/sbin \
    GIT_SSL_CAINFO=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt \
    NIX_SSL_CERT_FILE=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt

ENV \
    ENV=/etc/profile \
    USER=alpine \
    PATH=/nix/var/nix/profiles/default/bin:/nix/var/nix/profiles/default/sbin:/bin:/sbin:/usr/bin:/usr/sbin \
    GIT_SSL_CAINFO=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt \
    NIX_SSL_CERT_FILE=/nix/var/nix/profiles/default/etc/ssl/certs/ca-bundle.crt \
    NIX_PATH=/nix/var/nix/profiles/per-user/alpine/channels
