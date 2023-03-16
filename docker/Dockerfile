FROM mingc/android-build-box:1.23.1


# Set default RUN shell to /bin/bash
SHELL ["/bin/bash", "-cu"]


# Set environment variables
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8


# TUNA Mirror (optional)
#COPY project/codebase/misc/sources.list.ubuntu20.04 /etc/apt/sources.list


# Install basic packages for compiling and building
#ENV DEBIAN_FRONTEND=noninteractive
#RUN apt-get update && apt-get install -y --allow-downgrades --allow-change-held-packages --no-install-recommends \
#    build-essential \
#    cmake \
#    g++-7 \
#    git \
#    curl \
#    wget \
#    ca-certificates \
#    libjpeg-dev \
#    libpng-dev \
#    librdmacm1 \
#    libibverbs1 \
#    ibverbs-providers \
#    tzdata \
#    libgl1-mesa-glx \
#    libglib2.0-0 \
#    fontconfig \
#    && rm -rf /var/lib/apt/lists/*


# Below is the example installation command for stable versions:
# ENV PYTORCH_VERSION=1.8.2
# ENV TORCHVISION_VERSION=0.9.2
# RUN pip install --no-cache-dir \
#     torch==${PYTORCH_VERSION} \
#     torchvision==${TORCHVISION_VERSION} \
#     -f https://download.pytorch.org/whl/torch_stable.html


# ==================================================
# *Tools installation (optional)
#
# You find some tools useful or unwanted, comment or uncomment them as you wish.
# Check this page for more installation commands:
# https://github.com/hughplay/memo/blob/master/scripts/prepare_dl.sh
# ==================================================

# update key: https://github.com/yarnpkg/yarn/issues/7866
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -

# Install basic packages
RUN apt-get update && apt-get install -y --allow-downgrades --allow-change-held-packages --no-install-recommends \
    vim \
    rsync \
    zsh \
    sudo \
    && rm -rf /var/lib/apt/lists/*


# Set ZSH as default shell & Install Oh My Zsh and zsh plugin: zsh-autosuggestions
RUN chsh -s `which zsh` \
    && curl https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh | bash -s -- --unattended \
    && wget https://raw.githubusercontent.com/oskarkrawczyk/honukai-iterm/master/honukai.zsh-theme -O ${ZSH:-~/.oh-my-zsh}/themes/honukai.zsh-theme --no-check-certificate \
    && sed -i.bak '/ZSH_THEME/s/\".*\"/\"honukai\"/' ~/.zshrc \
    && git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH:-~/.oh-my-zsh}/custom/plugins/zsh-autosuggestions\
    && sed -i.bak '/plugin/s/(.*)/(git zsh-autosuggestions)/' ~/.zshrc

# make the color of zsh-autosuggestions right
ENV TERM xterm-256color


# Install docker (to make docker in docker work)
#RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
#    | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
#    && echo \
#    "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
#    $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null \
#    && apt-get update \
#    && apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin \
#    && rm -rf /var/lib/apt/lists/*


# Install Tmux
# RUN apt-get update && apt-get install -y libevent-dev ncurses-dev automake pkg-config \
#     && cd /tmp \
#     && wget -O tmux-2.8.tar.gz https://github.com/tmux/tmux/archive/2.8.tar.gz \
#     && tar zxvf tmux-2.8.tar.gz \
#     && cd tmux-2.8 \
#     && ./autogen.sh \
#     && ./configure --prefix=/usr/local \
#     && make \
#     && make install \
#     && rm -rf /var/lib/apt/lists/*

# Tmux configuration
# RUN git clone https://github.com/hughplay/tmux-config.git /tmp/tmux-config \
#     && bash /tmp/tmux-config/install.sh \
#     && rm -rf /tmp/tmux-config \
#     && echo "set -g default-shell `which zsh`" >> ~/.tmux.conf


# Install lightvim, a configuration for vim
# RUN wget https://raw.githubusercontent.com/hughplay/lightvim/master/install.sh -O - | sh

# ==================================================
# *Fonts installation
#
# Install fonts that are commonly used in plotting.
# Question: "Which font looks best in a scientific figure?"
# Answer: "Arial or Helvetica, always."
# https://pubs.acs.org/doi/10.1021/acs.chemmater.6b00306
# ==================================================

#RUN cd /tmp \
#    && wget https://github.com/hughplay/memo/raw/master/code/snippet/drawing/plot_fonts.tar.gz \
#    && tar zxvf plot_fonts.tar.gz \
#    && cp plot_fonts/* /usr/share/fonts/truetype/ \
#    && rm -rf plot_fonts.tar.gz plot_fonts \
#    && fc-cache -fv


# ==================================================
# Install python packages
# ==================================================

## Setup TUNA mirror (optional)
#RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
#COPY misc/.condarc /root/.condarc
#
## By default, install packages from `requirements.txt` with pip.
#COPY requirements.txt /tmp/requirements.txt
#RUN pip install --no-cache-dir -r /tmp/requirements.txt \
#    && rm -f /tmp/requirements.txt


# Another way is installing packages from a `env.yaml` with conda.
# COPY env.yaml /tmp/env.yaml
# RUN conda env create -f /tmp/env.yaml && rm -f /tmp/env.yaml


# ==================================================
# Post-installation steps
#
# Create a user that has the same UID and GID as the host user. This will
# prevent many privileges issues.
# ==================================================

# COPY .localrc
COPY misc/.localrc /root
RUN echo ". ~/.localrc" >> ~/.zshrc


# Add a user with the same UID and GID as the host user, to prevent privilege issues.
ARG USER_ID=1011
ARG GROUP_ID=1011
ARG USER_NAME=docker
RUN if [ $USER_NAME != "root" ] ; \
    then addgroup --gid ${GROUP_ID} ${USER_NAME} \
    && adduser --disabled-password --gecos '' --uid $USER_ID --gid $GROUP_ID ${USER_NAME} \
    && usermod -aG sudo ${USER_NAME} \
    && echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers ; fi


# Copy installed configuration files from root to user
COPY misc/init_workspace /usr/local/bin
RUN chmod +x /usr/local/bin/init_workspace
RUN /usr/local/bin/init_workspace --user ${USER_NAME} --home /home/${USER_NAME}
RUN cp -r /home/${USER_NAME} /${USER_NAME}_home_bak


# Switch to the created user
USER ${USER_NAME}


# Set working directory to /workspace
WORKDIR "/project"
