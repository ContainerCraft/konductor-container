###############################################################################
# Use:
# - docker build --progress plain --tag docker.io/testing/devcontainer:nvim -f Dockerfile .
# - docker run --rm -d --name devcontainer --hostname devcontainer docker.io/testing/devcontainer:nvim
###############################################################################
# Base Devcontainer Image
FROM ghcr.io/containercraft/devcontainer:dind
LABEL tag="neovim"
ENV DEVCONTAINER="neovim"

#################################################################################
# Base package and user configuration
#################################################################################
USER root

# Apt Packages
ARG APT_PKGS="\
  tar \
  gcc \
  curl \
  locales \
  ripgrep \
  fd-find \
  xclip \
  perl \
  ruby \
  ruby-dev \
  libyaml-dev \
  python3 \
  python3-pip \
  luarocks \
  libmsgpack-dev \
  "

# Install Base Packages
RUN echo \
  && export TEST="echo" \
  && ${apt_update} \
  && bash -c "${apt_install} software-properties-common ca-certificates" \
  && bash -c "${apt_install} ${APT_PKGS}" \
  && ln -s $(which fdfind) /usr/local/bin/fd \
  && sudo apt-get remove -y --purge nano \
  && curl -L https://cpanmin.us | perl - --notest App::cpanminus \
  && bash -c "${apt_clean}" \
  && ${dir_clean} \
  && ${TEST} \
  && echo

# Install lazygit
RUN echo \
  && export NAME="lazygit" \
  && export TEST="${NAME} --version" \
  && export REPOSITORY="jesseduffield/lazygit" \
  && export VERSION="$(${curl} https://api.github.com/repos/${REPOSITORY}/releases/latest | jq --raw-output .tag_name)" \
  && export ARCH=$(uname -m | awk '{ if ($1 == "x86_64") print "x86_64"; else if ($1 == "aarch64" || $1 == "arm64") print "arm64"; else print "unknown" }') \
  && export PKG="${NAME}_${VERSION#v}_Linux_${ARCH}.tar.gz" \
  && export URL="https://github.com/${REPOSITORY}/releases/download/${VERSION}/${PKG}" \
  && echo "---------------------------------------------------------"\
  && echo "INFO[${NAME}] Installed:" \
  && echo "INFO[${NAME}]   Command:        ${NAME}" \
  && echo "INFO[${NAME}]   Package:        ${PKG}" \
  && echo "INFO[${NAME}]   Latest Release: ${VERSION}" \
  && echo "INFO[${NAME}]   Architecture:   ${ARCH}" \
  && echo "INFO[${NAME}]   Source:         ${URL}" \
  && echo "---------------------------------------------------------"\
  && ${curl} ${URL} | tar xzvf - --directory /tmp \
  && chmod 755 /tmp/${NAME} \
  && chown root:root /tmp/${NAME} \
  && mv /tmp/${NAME} /usr/local/bin/ \
  && ${dir_clean} \
  && ${TEST} \
  && echo

# Generate and Set Locale
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
  && locale-gen \
  && update-locale LANG=en_US.UTF-8

# Install Python Packages
RUN pip3 install --no-cache-dir pynvim

# Install Perl Neovim Extension
RUN cpanm --notest Neovim::Ext

# Install Ruby Neovim Gem
RUN gem install neovim bashly

#################################################################################
# Install Programming Language Tooling
# - nodejs
# - npm
# - yarn
RUN echo \
    && sudo apt-get remove -y nodejs npm libnode-dev || true \
    && mkdir -p /etc/apt/keyrings \
    && curl -L https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key \
    | sudo gpg --yes --batch --dearmor -o /etc/apt/keyrings/nodesource.gpg \
    && export NODE_MAJOR=20 \
    && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" \
       | sudo tee -a /etc/apt/sources.list.d/nodesource.list \
    && apt-get update \
    && apt-get install -y nodejs \
    && apt-get clean \
    && apt-get autoremove -y \
    && apt-get purge -y --auto-remove \
    && ${dir_clean} \
    && node --version \
    && npm --version \
    && npm install --global yarn \
    && yarn --version \
    && true

#################################################################################
# Build and Install Neovim from Source
#################################################################################

# Build Packages
ARG BUILD_PKGS="\
make \
wget \
build-essential \
ninja-build \
gettext \
libtool \
libtool-bin \
autoconf \
automake \
cmake \
pkg-config \
unzip \
doxygen \
"

# Optional: replace vim with nvim
# && apt-get purge -y --auto-remove $(echo "${BUILD_PKGS}" | tr -d '\n' | sed 's/  */ /g') \
RUN echo \
  && export NAME="neovim" \
  && export TEST="nvim --version" \
  && export REPOSITORY="neovim/neovim" \
  && export VERSION="$(${curl_github} https://api.github.com/repos/${REPOSITORY}/releases/latest | jq --raw-output .tag_name)" \
  && echo "---------------------------------------------------------" \
  && echo "INFO[${NAME}] Building Neovim Version: ${VERSION}" \
  && echo "---------------------------------------------------------" \
  && ${apt_update} \
  && bash -c "${apt_install} ${BUILD_PKGS}" \
  && git clone --depth 1 --branch ${VERSION} https://github.com/${REPOSITORY}.git /tmp/neovim \
  && cd /tmp/neovim \
  && make CMAKE_BUILD_TYPE=Release \
  && make install \
  && rm -rf /tmp/neovim \
  && update-alternatives --install /usr/bin/vim vim /usr/local/bin/nvim 100 \
  && update-alternatives --set vim /usr/local/bin/nvim \
  && bash -c "${apt_clean}" \
  && ${dir_clean} \
  && ${TEST} \
  && echo

USER vscode
RUN set -ex \
  && git clone https://github.com/LazyVim/starter ~/.config/nvim \
  && nvim --headless "+Lazy! sync" +qa \
  && nvim --headless "+Lazy! sync" +qa \
  && echo 'vim.keymap.set("i", "jj", "<ESC>", { silent = true })' | tee -a /etc/skel/.config/nvim/lua/config/keymaps.lua \
  && echo 'vim.keymap.set("i", "jj", "<ESC>", { silent = true })' | tee -a /home/vscode/.config/nvim/lua/config/keymaps.lua \
  && true

# Install Node.js Neovim Package and tree-sitter-cli
RUN sudo npm install -g neovim tree-sitter-cli

#################################################################################
# Set the default command
#################################################################################
CMD ["bash"]
