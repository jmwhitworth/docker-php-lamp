FROM php:8.3.12-fpm-bullseye

ARG USER_NAME
ARG USER_UID
ARG USER_GID
ARG GIT_EMAIL
ARG GIT_NAME
ARG NODE_VERSION
ARG COMPOSER_VERSION
ENV HOME_DIR /home/${USER_NAME}
ENV NVM_DIR ${HOME_DIR}/.nvm

# Update container and install core packages
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    make \
    build-essential \
    libssl-dev \
    default-libmysqlclient-dev \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    llvm \
    libncursesw5-dev \
    xz-utils \
    tk-dev \
    libxml2-dev \
    libxmlsec1-dev \
    libffi-dev \
    liblzma-dev \
    git \
    gnupg \
    nano \
    iputils-ping \
    python3 \
    python3-dev \
    python3-pip \
    pkg-config \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install PHP extensions - https://github.com/mlocati/docker-php-extension-installer
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
RUN chmod +x /usr/local/bin/install-php-extensions \
    && install-php-extensions \
        exif \
        gd \
        gmp \
        mysqli \
        # imagick \ *NOT CURRENTLY SUPPORTED
        intl \
        opcache \
        redis \
        pdo_mysql \
        zip \
        xdebug \
        @composer-${COMPOSER_VERSION} \
        calendar

# Set up new user and add to sudoers
RUN groupadd --gid ${USER_GID} ${USER_NAME} \
    && useradd --uid ${USER_UID} --gid ${USER_GID} -m -s /bin/bash ${USER_NAME} \
    && apt-get update && apt-get install -y sudo \
    && echo ${USER_NAME} ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/${USER_NAME} \
    && chmod 0440 /etc/sudoers.d/${USER_NAME}

# Create the composer directory in advance and set permissions so composer can store cache in the volume
RUN mkdir ${HOME_DIR}/.composer \
    && chown ${USER_NAME}:${USER_NAME} ${HOME_DIR}/.composer

# Switch to the non-root user
USER ${USER_NAME}

# Set git user credentials
RUN git config --global user.email ${GIT_EMAIL} \
    && git config --global user.name ${GIT_NAME} \
    && git config --global pull.rebase false

# Install nvm with node and npm
RUN curl https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash \
    && . ${NVM_DIR}/nvm.sh \
    && nvm install ${NODE_VERSION} \
    && nvm alias default ${NODE_VERSION} \
    && nvm use default \
    && npm install -g yarn

# Add node to path
ENV NODE_PATH ${NVM_DIR}/versions/node/v${NODE_VERSION}/lib/node_modules
ENV PATH      ${NVM_DIR}/versions/node/v${NODE_VERSION}/bin:$PATH

# Install Heroku CLI
RUN wget -qO- https://cli-assets.heroku.com/install.sh | sh \
    && heroku plugins:install apps-table \
    && heroku plugins:install @heroku-cli/plugin-data-maintenance

# Install ZSH https://github.com/deluan/zsh-in-docker
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.1.5/zsh-in-docker.sh)" -- \
    -a '. ~/.bash_aliases' \
    -a 'tput rmcup' \
    -p git \
    -p ssh-agent \
    -p https://github.com/zsh-users/zsh-autosuggestions \
    -p https://github.com/zsh-users/zsh-completions && \
    sudo chsh -s /bin/zsh ${USER_NAME}

# Install aws-cli
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" \
    && sudo apt-get install -y unzip \
    && unzip awscliv2.zip \
    && sudo ./aws/install \
    && rm -rf awscliv2.zip aws

# Install pipenv and pyenv
RUN pip install --user pipenv && curl https://pyenv.run | bash

# Add pipenv to $PATH
ENV PATH ${HOME_DIR}/.local/bin:$PATH
