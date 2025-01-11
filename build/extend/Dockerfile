# Set base image
FROM debian:buster-slim

# Set Dockerfile variables
ARG FOAM_VER=4.0
ARG FOAM_PATH=/opt/FOAM-Extend

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

RUN apt-get install -y \ 
        binutils-dev \
        bison \
        build-essential \
        cmake \
        curl \
        flex \
        g++-7 \
        gcc-7 \
        git-core \
        graphviz \
        htop \
        libmetis-dev \
        libncurses5-dev \
        libptscotch-dev \
        libxt-dev \
        mercurial \
        neofetch \
        python \
        python3 \
        python3-pip \
        python-dev \
        ranger \
        rpm \
        tmux \
        vim \
        zlib1g-dev \
        zsh

# Download the source pack
WORKDIR ${FOAM_PATH}
RUN git clone git://git.code.sf.net/p/foam-extend/foam-extend-${FOAM_VER} ${FOAM_PATH}/foam-extend-${FOAM_VER}

# Configure the environment and build FOAM-Extend
ENV USER root
ENV SHELL /bin/bash
SHELL ["/bin/bash", "-c"]
RUN sed -i 's/foamInstall=$HOME\/$WM_PROJECT/# foamInstall=$HOME\/$WM_PROJECT%/' ${FOAM_PATH}/foam-extend-${FOAM_VER}/etc/bashrc && \
    sed -i 's/# foamInstall=\/opt\/$WM_PROJECT/foamInstall=\/opt\/FOAM-Extend/' ${FOAM_PATH}/foam-extend-${FOAM_VER}/etc/bashrc && \
    source ${FOAM_PATH}/foam-extend-${FOAM_VER}/etc/bashrc && \
    echo "export WM_CC='gcc-7'"  >> ${FOAM_PATH}/foam-extend-${FOAM_VER}/etc/prefs.sh && \
    echo "export WM_CXX='g++-7'"  >> ${FOAM_PATH}/foam-extend-${FOAM_VER}/etc/prefs.sh && \
    cd ${FOAM_PATH}/foam-extend-${FOAM_VER} && \
    sed -i -e 's=rpmbuild --define=rpmbuild --define "_build_id_links none" --define=' ${FOAM_PATH}/foam-extend-${FOAM_VER}/ThirdParty/tools/makeThirdPartyFunctionsForRPM && \
    sed -i -e 's/gcc/\$(WM_CC)/' ${FOAM_PATH}/foam-extend-${FOAM_VER}/wmake/rules/linux64Gcc/c && \
    sed -i -e 's/g++/\$(WM_CXX)/' ${FOAM_PATH}/foam-extend-${FOAM_VER}/wmake/rules/linux64Gcc/c++ && \
    ./Allwmake.firstInstall 1>/dev/null

# Add user 'foamextend'
RUN groupadd --gid 1000 foamextend && \
    useradd --uid 1000 --gid 1000 --shell /bin/bash --create-home --home-dir /home/foamextend foamextend && \
    chmod 755 ${FOAM_PATH}
USER foamextend
WORKDIR /home/foamextend

# Install 'oh-my-zsh'
RUN curl -sL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | bash /dev/stdin && \
    echo "source ${FOAM_PATH}/foam-extend-${FOAM_VER}/etc/zshrc" | tee -a ~/.zshenv > /dev/null && \
    echo "export OMPI_MCA_btl_vader_single_copy_mechanism=none" | tee -a ~/.zshenv > /dev/null && \
    echo "neofetch" | tee -a ~/.zprofile > /dev/null && \
    mkdir ~/cases

# Install 'pyFoam'
RUN export PATH="${HOME}/.local/bin:${PATH}" && \
    pip3 install PyFoam
ENV PATH ${PATH}:~/.local/bin

# Reconfigure environment for zsh shell
ENV USER foamextend
ENV SHELL /bin/zsh
SHELL ["/bin/zsh", "-c"]

# Run an interactive zsh shell
ENTRYPOINT ["/bin/zsh", "-c", "tmux new -A -s foamextend"]