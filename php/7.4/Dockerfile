FROM php:7.4.33-fpm-bullseye

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
    build-essential \
    libssl-dev \
    git \
    gnupg \
    nano \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install PHP extensions - https://github.com/mlocati/docker-php-extension-installer
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
RUN chmod +x /usr/local/bin/install-php-extensions \
    && install-php-extensions \
        exif \
        gd \
        mysqli \
        imagick \
        intl \
        opcache \
        redis \
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
RUN curl https://cli-assets.heroku.com/install-ubuntu.sh | sh \
    && heroku plugins:install apps-table

# Install ZSH https://github.com/deluan/zsh-in-docker
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.1.5/zsh-in-docker.sh)" -- \
    -a '. ~/.bash_aliases' \
    -p git \
    -p ssh-agent \
    -p https://github.com/zsh-users/zsh-autosuggestions \
    -p https://github.com/zsh-users/zsh-completions && \
    sudo chsh -s /bin/zsh ${USER_NAME}
