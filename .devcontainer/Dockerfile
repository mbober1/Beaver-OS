FROM ubuntu:24.04@sha256:c4a8d5503dfb2a3eb8ab5f807da5bc69a85730fb49b5cfca2330194ebcc41c7b

ARG USERNAME="ubuntu"
ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-c"]

RUN apt-get -y update && \
	apt-get --no-install-recommends -y install \
    build-essential chrpath cpio debianutils diffstat \
    file gawk gcc git iputils-ping libacl1 liblz4-tool \
    locales python3 python3-git python3-jinja2 python3-pexpect \
    python3-pip python3-subunit socat texinfo unzip wget \
    xz-utils zstd screen ssh \
		&& \
	apt-get clean -y && \
	apt-get autoremove --purge -y && \
	rm -rf /var/lib/apt/lists/*

# Initialise system locale
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US:en
ENV LC_ALL=en_US.UTF-8
RUN locale-gen $LANG

ADD --chmod=a+x+r https://storage.googleapis.com/git-repo-downloads/repo /bin/repo
USER ${USERNAME}
WORKDIR /home/${USERNAME}/workspace
