# Set base image
FROM debian:buster-slim

# Set Dockerfile variables
ARG FOAM_VER=9
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
        git \
        gnuplot \
        htop \
        libboost-system-dev \
        libboost-thread-dev \
        libncurses-dev \
        libopenmpi-dev \
        libreadline-dev \
        libxt-dev \
        openmpi-bin \
        python3 \
        python3-pip \
        ranger \
        neofetch \
        tmux \
        vim \
        zlib1g-dev \
        zsh

# Download the source pack
WORKDIR ${FOAM_PATH}
RUN curl -L http://dl.openfoam.org/source/${FOAM_VER} | tar xvz && \
    curl -L http://dl.openfoam.org/third-party/${FOAM_VER} | tar xvz && \
    mv OpenFOAM-${FOAM_VER}-version-${FOAM_VER} ${FOAM_PATH}/OpenFOAM-${FOAM_VER} && \
    mv ThirdParty-${FOAM_VER}-version-${FOAM_VER} ${FOAM_PATH}/ThirdParty-${FOAM_VER} && \
    cd ${FOAM_PATH}/ThirdParty-${FOAM_VER} && \
    curl -L https://github.com/CGAL/cgal/releases/download/releases%2FCGAL-5.0.2/CGAL-5.0.2.tar.xz | tar xvJf - && \
    curl -L http://glaros.dtc.umn.edu/gkhome/fetch/sw/metis/metis-5.1.0.tar.gz | tar xzvf -

# Configure the environment and build OpenFOAM
ENV USER root
ENV SHELL /bin/bash
SHELL ["/bin/bash", "-c"]
RUN source ${FOAM_PATH}/OpenFOAM-${FOAM_VER}/etc/bashrc && \
    cd ${FOAM_PATH}/ThirdParty-${FOAM_VER} && \
    ./Allwmake -j 1>/dev/null && \
    cd ${FOAM_PATH}/OpenFOAM-${FOAM_VER} && \
    ./Allwmake -j 1>/dev/null

# Add user 'openfoam'
RUN groupadd --gid 1000 openfoam && \
    useradd --uid 1000 --gid 1000 --shell /bin/bash --create-home --home-dir /home/openfoam openfoam && \
    chmod 755 ${FOAM_PATH}
USER openfoam
WORKDIR /home/openfoam

# Install zsh
RUN curl -sL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | bash /dev/stdin && \
    echo "source ${FOAM_PATH}/OpenFOAM-${FOAM_VER}/etc/bashrc" | tee -a ~/.zshenv > /dev/null && \
    echo "export OMPI_MCA_btl_vader_single_copy_mechanism=none" | tee -a ~/.zshenv /dev/null && \
    echo '# Add user local private bin to PATH if it exists\nif [ -d "$HOME/.local/bin" ] ; then\n\tPATH="$HOME/.local/bin:$PATH"\nfi' | tee -a  ~/.zprofile > /dev/null && \
    echo "neofetch" | tee -a  ~/.zprofile > /dev/null && \
    mkdir ~/cases

# Install pyFoam and other Python utilities
RUN export PATH="~/.local/bin:${PATH}" && \
    pip3 install numpy matplotlib PyFoam
ENV PATH ${PATH}:~/.local/bin

# Reconfigure environment for zsh shell
ENV USER openfoam
ENV SHELL /bin/zsh
SHELL ["/bin/zsh", "-c"]

# Run an interactive zsh shell
ENTRYPOINT ["/bin/zsh", "-c", "tmux new -A -s openfoam"]