FROM ubuntu:22.04

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        bash-completion \
        bison \
        bc \
        build-essential \
        cmake \
        curl \
        dstat \
        flex \
        gdb \
        git \
        libc6-dbg \
        libicu-dev \
        libreadline-dev \
        locales \
        pkg-config \
        tmux \
        valgrind \
        vim \
        wget \
        zlib1g \
        zlib1g-dev && \
    locale-gen en_US.UTF-8 && \
    update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 && \
    rm -rf /var/lib/apt/lists/*

ENV LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

# Create a non-root user for development
ARG USERNAME=postgresdev
ARG USERPASS=postgresdev

RUN adduser --disabled-password --gecos "" ${USERNAME} && \
    echo "${USERNAME}:${USERPASS}" | chpasswd && \
    usermod -aG sudo ${USERNAME}

RUN printf '\n# enable bash completion if available\nif [ -f /etc/bash_completion ] && ! shopt -oq posix; then\n  . /etc/bash_completion\nfi\n' >> /etc/bash.bashrc

USER ${USERNAME}

CMD ["/bin/bash"]
