FROM ubuntu:22.04

LABEL maintainer="@k33g"

# Set environment variables for Go version and user
ARG GO_ARCH=${GO_ARCH}
ARG GO_VERSION=${GO_VERSION}

ARG NODE_DISTRO=${NODE_DISTRO}
ARG NODE_VERSION=${NODE_VERSION}

ARG ZELLIJ_VERSION=${ZELLIJ_VERSION}
ARG ZELLIJ_ARCH=${ZELLIJ_ARCH}

ARG HELM_VERSION=${HELM_VERSION}
ARG HELM_ARCH=${HELM_ARCH}

ARG K9S_VERSION=${K9S_VERSION}
ARG K9S_ARCH=${K9S_ARCH}

ARG CODER_VERSION=${CODER_VERSION}
ARG CODER_ARCH=${CODER_ARCH}

ARG USER_NAME=${USER_NAME}

ARG DEBIAN_FRONTEND=noninteractive

ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8
ENV LC_COLLATE=C
ENV LC_CTYPE=en_US.UTF-8

# ------------------------------------
# Install Tools
# ------------------------------------
RUN <<EOF
apt-get update 
apt-get install -y curl wget git build-essential xz-utils software-properties-common htop openssh-server sudo gopls delve pkg-config libssl-dev sshpass gettext
apt-get install -y nano mc bat exa
ln -s /usr/bin/batcat /usr/bin/bat
apt-get -y install hey
add-apt-repository ppa:maveonair/helix-editor
apt update
apt install helix
EOF

# ------------------------------------
# Install Coder
# ------------------------------------
RUN <<EOF
curl -fOL https://github.com/coder/code-server/releases/download/v$CODER_VERSION/code-server_${CODER_VERSION}_${CODER_ARCH}.deb
dpkg -i code-server_${CODER_VERSION}_${CODER_ARCH}.deb
EOF

# ------------------------------------
# Install Docker
# ------------------------------------
RUN <<EOF
# Add Docker's official GPG key:
apt-get update
apt-get install -y ca-certificates curl gnupg
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
EOF

# ------------------------------------
# Install Go
# ------------------------------------
RUN <<EOF
wget https://golang.org/dl/go${GO_VERSION}.linux-${GO_ARCH}.tar.gz
tar -xvf go${GO_VERSION}.linux-${GO_ARCH}.tar.gz
mv go /usr/local
rm go${GO_VERSION}.linux-${GO_ARCH}.tar.gz
EOF

# ------------------------------------
# Set Environment Variables for Go
# ------------------------------------
ENV PATH="/usr/local/go/bin:${PATH}"
ENV GOPATH="/home/${USER_NAME}/go"
ENV GOROOT="/usr/local/go"

RUN <<EOF
go version
go install -v golang.org/x/tools/gopls@latest
go install -v github.com/ramya-rao-a/go-outline@latest
go install -v github.com/stamblerre/gocode@v1.0.0
EOF

# ------------------------------------
# Install Zellij
# ------------------------------------
RUN <<EOF
wget https://github.com/zellij-org/zellij/releases/download/v${ZELLIJ_VERSION}/zellij-${ZELLIJ_ARCH}-unknown-linux-musl.tar.gz
tar -xvf zellij*.tar.gz
chmod +x zellij
cp zellij /usr/bin
rm zellij*.tar.gz
rm zellij
EOF

# ------------------------------------
# Install NodeJS
# ------------------------------------
RUN <<EOF
wget https://nodejs.org/dist/${NODE_VERSION}/node-${NODE_VERSION}-${NODE_DISTRO}.tar.xz
mkdir -p /usr/local/lib/nodejs
tar -xJvf node-$NODE_VERSION-$NODE_DISTRO.tar.xz -C /usr/local/lib/nodejs
rm node-$NODE_VERSION-$NODE_DISTRO.tar.xz
EOF

ENV VERSION="${NODE_VERSION}"
ENV DISTRO="${NODE_DISTRO}"
ENV PATH=/usr/local/lib/nodejs/node-$VERSION-$DISTRO/bin:$PATH

# ------------------------------------
# Install Helm
# ------------------------------------
RUN <<EOF
wget https://get.helm.sh/helm-v${HELM_VERSION}-linux-${HELM_ARCH}.tar.gz
tar xvf helm-v${HELM_VERSION}-linux-${HELM_ARCH}.tar.gz
mv linux-${HELM_ARCH}/helm /usr/local/bin
EOF

# ------------------------------------
# Install K8S tools: kubectl & k9s
# ------------------------------------
RUN <<EOF
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

chmod +x kubectl
mv ./kubectl /usr/local/bin/kubectl

wget https://github.com/derailed/k9s/releases/download/v${K9S_VERSION}/k9s_${K9S_ARCH}.tar.gz
tar xvf k9s_${K9S_ARCH}.tar.gz -C /usr/local/bin
EOF


# ------------------------------------
# Create a new user
# ------------------------------------
RUN adduser --disabled-password --gecos '' ${USER_NAME}

#  Add new user `${USER_NAME}` to sudo and docker group
RUN adduser ${USER_NAME} sudo
RUN adduser ${USER_NAME} docker

# Ensure sudo group users are not asked for a password when using 
# sudo command by ammending sudoers file
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Set the working directory
WORKDIR /home/${USER_NAME}

# Set the user as the owner of the working directory
RUN chown -R ${USER_NAME}:${USER_NAME} /home/${USER_NAME}

# Switch to the regular user
USER ${USER_NAME}

# Avoid the message about sudo
RUN touch ~/.sudo_as_admin_successful

# ------------------------------------
# Install OhMyBash
# ------------------------------------
RUN <<EOF
bash -c "$(curl -fsSL https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh)"
EOF

CMD ["/bin/bash"]
