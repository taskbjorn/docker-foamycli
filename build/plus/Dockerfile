# Set base image
FROM debian:buster-slim

# Set Dockerfile variables
ARG FOAM_VER=2106
ARG FOAM_PATH=/opt/OpenFOAM

# Set system variables
ENV DEBIAN_FRONTEND noninteractive

# Update package sources
RUN apt-get update

# Fix locale waning messages
RUN apt-get install -y locales && \
    sed -i -e 's/# en_GB.UTF-8 UTF-8/en_GB.UTF-8 UTF-8/' /etc/locale.gen && \
    dpkg-reconfigure --frontend=noninteractive locales && \
    update-locale LANG=en_GB.UTF-8
ENV LANGUAGE en_GB
ENV LC_ALL en_GB.UTF-8
ENV LANG en_GB.UTF-8

# Install build prerequisites and system tools
RUN apt-get install -y \
        bison \
        build-essential \
        cmake \
        curl \
        flex \
        git-core \
        gnuplot \
        htop \
        libboost-system-dev \
        libboost-thread-dev \
        libcgal-dev \
        libncurses-dev \
        libopenmpi-dev \
        libreadline-dev \
        libscotch-dev \
        libxt-dev \
        neofetch \
        openmpi-bin \
        python \
        python3 \
        python3-pip \
        python-dev \
        ranger \
        tmux \
        vim \
        zlib1g-dev \
        zsh

# Download the source pack
WORKDIR ${FOAM_PATH}
RUN curl -L https://sourceforge.net/projects/openfoam/files/v${FOAM_VER}/OpenFOAM-v${FOAM_VER}.tgz | tar xvz && \
    curl -L https://sourceforge.net/projects/openfoam/files/v${FOAM_VER}/ThirdParty-v${FOAM_VER}.tgz | tar xvz

# Configure the environment and build OpenFOAM
ENV USER root
ENV SHELL /bin/bash
SHELL ["/bin/bash", "-c"]
RUN source OpenFOAM-v${FOAM_VER}/etc/bashrc && \
    foamSystemCheck && \
    cd ${FOAM_PATH}/ThirdParty-v${FOAM_VER} && \
    ./Allwmake -j 1>/dev/null && \
    cd ${FOAM_PATH}/OpenFOAM-v${FOAM_VER} && \
    ./Allwmake -j 1>/dev/null

# Add user 'openfoam'
RUN groupadd --gid 1000 ofplus && \
    useradd --uid 1000 --gid 1000 --shell /bin/bash --create-home --home-dir /home/ofplus ofplus && \
    chmod 755 ${FOAM_PATH}
USER ofplus
WORKDIR /home/ofplus

# Install 'oh-my-zsh'
RUN curl -sL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | bash /dev/stdin && \
    echo "source ${FOAM_PATH}/OpenFOAM-v${FOAM_VER}/etc/bashrc" | tee -a ~/.zshenv > /dev/null && \
    echo "export OMPI_MCA_btl_vader_single_copy_mechanism=none" | tee -a ~/.zshenv > /dev/null && \
    echo "neofetch" | tee -a ~/.zprofile > /dev/null && \
    mkdir ~/cases

# Install 'pyFoam'
RUN export PATH="~/.local/bin:${PATH}" && \
    pip3 install PyFoam
ENV PATH ${PATH}:~/.local/bin

# Reconfigure environment for zsh shell
ENV USER ofplus
ENV SHELL /bin/zsh
SHELL ["/bin/zsh", "-c"]

# Run an interactive zsh shell
ENTRYPOINT ["/bin/zsh", "-c", "tmux new -A -s ofplus"]