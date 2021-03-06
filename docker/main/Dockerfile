#
# SpinalDev Dockerfile
#
# https://github.com/plex1/SpinalDev
#
# This Dockerfile creates a image full of useful tools for
# SpinalHDL development. See associated README.md for more
# information. This Dockerfile is mostly based on the instructions
# found at https://spinalhdl.github.io/SpinalDoc.

# Pull base image
FROM ubuntu:16.04

# Set the maintainer
MAINTAINER Felix Arnold (plex1) <felix.arnold@gmail.com>

# Global arguments
ARG USER=spinaldev
ARG USERPWD=spinaldev 
ARG WORKDIR=/home/spinaldev
ARG ROOTPWD=spinaldev

# Set frontend required for docker
ENV DEBIAN_FRONTEND noninteractive

####### Packages #######################################
# Install some base tools and x11 tools
RUN apt-get update && apt-get install -y \
  apt-utils \
  sudo \
  unzip \
  wget \
  adwaita-icon-theme-full \ 
  emacs \
  git \
  x11-apps \
  xfce4\
  xrdp \ 
  xfce4-terminal 
 

####### Linux ##########################################

# Change root password
RUN echo "root:${ROOTPWD}" | chpasswd

# Create the default user
RUN useradd -m -s /bin/bash -d ${WORKDIR} ${USER}
RUN echo "${USER}:${USERPWD}" | chpasswd
RUN adduser ${USER} sudo

# Customize terminal
RUN echo 'RESET="\[$(tput sgr0)\]"' >> $WORKDIR/.bashrc
RUN echo 'GREEN="\[$(tput setaf 2)\]"' >> $WORKDIR/.bashrc 
RUN echo 'export PS1="${GREEN}\u:\W${RESET} $ "' >> $WORKDIR/.bashrc 

# owner settings:
#  - user root for all files in /opt
#  - user spinaldev for all files in /home/spinaldev


####### Spinal HDL and tools ##############################

USER root
# Install java and scala
RUN apt-get update && apt-get install -y \
  openjdk-8-jdk \
  scala

# Install sbt
RUN apt-get install -y apt-transport-https #required for sbt debian
RUN echo "deb https://dl.bintray.com/sbt/debian /" | sudo tee -a /etc/apt/sources.list.d/sbt.list
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 2EE0EA64E40A89B84B2DF73499E82A75642AC823
RUN apt-get update
RUN apt-get -y install sbt

USER $USER

# Install spinal core & lib
RUN mkdir -p $WORKDIR/tools/spinal
RUN git clone https://github.com/SpinalHDL/SpinalHDL.git $WORKDIR/tools/spinal/SpinalHDL
WORKDIR $WORKDIR/tools/spinal/SpinalHDL
RUN sbt clean compile publishLocal

RUN mkdir -p $WORKDIR/projects/spinal
RUN cd $WORKDIR/projects/spinal
RUN git clone https://github.com/SpinalHDL/VexRiscv.git $WORKDIR/projects/spinal/VexRiscv
RUN git clone https://github.com/SpinalHDL/SpinalTemplateSbt.git $WORKDIR/projects/spinal/SpinalTemplateSbt
RUN git clone https://github.com/SpinalHDL/VexRiscvSocSoftware.git $WORKDIR/projects/spinal/VexRiscvSocSoftware

USER root

# Install verilator
# Verilator (version 3.9+ required, in general apt-get will give 3.8)
RUN apt-get update && apt-get install -y \
  git \
  make \
  autoconf \
  g++ \
  flex \
  bison

USER $USER

RUN mkdir -p $WORKDIR/tools
RUN git clone http://git.veripool.org/git/verilator $WORKDIR/tools/verilator
RUN unset VERILATOR_ROOT  
WORKDIR $WORKDIR/tools/verilator
# Create ./configure script
RUN autoconf
RUN ./configure
RUN make -j$(nproc)
USER root
RUN echo "${USER} ALL =(ALL) NOPASSWD:ALL" | sudo tee -a /etc/sudoers
USER $USER
RUN sudo make install

# Install COCOTB
RUN sudo apt-get update && sudo apt-get install -y \
  gcc \
  g++ \
  swig \
  python-dev
  
WORKDIR $WORKDIR/tools

RUN git clone https://github.com/potentialventures/cocotb
RUN export COCOTB=$WORKDIR/tools/cocotb
RUN echo 'export COCOTB=$WORKDIR/tools/cocotb' >> ~/.bashrc 

# Install GTKWave
RUN sudo apt-get install -y gtkwave

RUN mkdir -p $WORKDIR/projects/user


####### RICSV #############################################

USER root

# Make a working folder and set the necessary environment variables.
ENV RISCV /opt/riscv
ENV NUMJOBS 1


# Add the GNU utils bin folder to the path.
ENV PATH $RISCV/bin:$PATH
RUN echo 'export PATH=/opt/riscv/bin:$PATH' >> $WORKDIR/.bashrc

# Set the version variables
ARG RISCV_GCC_VER=riscv64-unknown-elf-gcc-20170612-x86_64-linux-centos6

WORKDIR /opt

# Download pre-built gcc compiler
RUN wget https://static.dev.sifive.com/dev-tools/$RISCV_GCC_VER.tar.gz -q && \
    tar -xzvf $RISCV_GCC_VER.tar.gz && \
    mv $RISCV_GCC_VER /opt/riscv && \
    rm $RISCV_GCC_VER.tar.gz

# Run a simple test to make sure compile is setup corretly
RUN mkdir -p $RISCV/test
WORKDIR $RISCV/test
RUN echo '#include <stdio.h>\n int main(void) { printf("Hello \
  world!\\n"); return 0; }' > hello.c 
RUN riscv64-unknown-elf-gcc -o hello hello.c


####### FPGA/ASIC FLOW ####################################

WORKDIR /opt

#openocd-riscv-vecriscv
RUN apt-get install -y \
  libtool automake libusb-1.0.0-dev texinfo libusb-dev libyaml-dev pkg-config
  
RUN git clone https://github.com/SpinalHDL/openocd_riscv.git && \
    cd openocd_riscv && \
    ./bootstrap && \
    ./configure --enable-ftdi --enable-dummy  && \
    make
    
#icepack see http://www.clifford.at/icestorm/   
#icepack dependencies
RUN apt-get install -y \
  pkg-config \
  libftdi-dev \
  libffi-dev
  
#yosys dependencies  
RUN apt-get install -y \ 
  tcl-dev \
  clang\
  gawk \
  libreadline-dev \
  mercurial 
  
# yosys  
RUN git clone https://github.com/YosysHQ/yosys.git yosys && \
    cd yosys && \
    make -j$(nproc) && \
    make install
    
# icepack
RUN git clone https://github.com/cliffordwolf/icestorm.git icestorm && \
    cd icestorm && \
    make -j$(nproc) && \
    make install

# arachne-pnr
RUN git clone https://github.com/YosysHQ/arachne-pnr.git arachne-pnr && \
    cd arachne-pnr && \
    make -j$(nproc) && \
    make install

# iceprogduino
# see https://github.com/OLIMEX/iCE40HX1K-EVB/tree/master/programmer/iceprogduino

#RUN sudo sed -i -e 's/archive.ubuntu.com\|security.ubuntu.com/old-releases.ubuntu.com/g' /etc/apt/sources.list
RUN apt-get update

# next-pnr dependencies
RUN apt-get install -y \
  cmake \
  python3-dev\
  qt5-default \
  libboost-dev \
  libboost-filesystem-dev \
  libboost-thread-dev \
  libboost-program-options-dev \
  libboost-python-dev \
  libboost-iostreams-dev\
  libboost-dev \
  libeigen3-dev

# next-pnr
RUN git clone https://github.com/YosysHQ/nextpnr nextpnr && \
    cd nextpnr && \
    cmake -DARCH=ice40  && \
    make -j$(nproc)  && \
    make install

####### Yosys formal ###################################

WORKDIR /opt

RUN apt-get install -y \
  clang bison flex libreadline-dev \
  gawk tcl-dev libffi-dev git mercurial \
  graphviz xdot pkg-config python python3 \
  libftdi-dev gperf libboost-program-options-dev \
  autoconf libgmp-dev cmake

RUN git clone https://github.com/cliffordwolf/SymbiYosys.git SymbiYosys && \
    cd SymbiYosys && \
    make install

RUN git clone https://github.com/Z3Prover/z3.git z3 && \
    cd z3 && \
    python scripts/mk_make.py && \
    cd build && \
    make -j$(nproc) && \
    sudo make install

RUN git clone https://github.com/boolector/boolector && \
    git clone https://github.com/arminbiere/lingeling boolector/deps/lingeling && \
    git clone https://github.com/boolector/btor2tools boolector/deps/btor2tools && \
    ( cd boolector/deps/lingeling  && ./configure.sh -fPIC && make -j$(nproc); ) && \
    ( cd boolector/deps/btor2tools && ./configure.sh -fPIC && make -j$(nproc); ) && \
    ( cd boolector && ./configure.sh && cd build && make -j$(nproc); ) && \
    cp boolector/build/bin/boolector /usr/local/bin/ && \
    cp boolector/build/bin/btor* /usr/local/bin/ && \
    cp boolector/deps/btor2tools/bin/btorsim /usr/local/bin/

WORKDIR $WORKDIR/tools
USER $USER

RUN git clone https://github.com/SymbioticEDA/riscv-formal.git

####### IntelliJ #######################################

WORKDIR /opt
USER root

# Set the version variables
ARG INTELLIJ_VER=2020.1
ARG INTELLIJ_SUBVER=1
# version id can be found on jetbrains webpage when downloading of scala plugin
ARG INTELLIJ_VERID=47787

# Download and install intellij
RUN wget https://download.jetbrains.com/idea/ideaIC-$INTELLIJ_VER.$INTELLIJ_SUBVER.tar.gz -O /tmp/intellij.tar.gz -q && \
    mkdir -p /opt/intellij && \
    tar -xf /tmp/intellij.tar.gz --strip-components=1 -C /opt/intellij && \
    rm /tmp/intellij.tar.gz
RUN cd /opt/intellij/bin && \
    ln -s idea.sh intellij

# Add intellij to path
RUN echo 'export PATH=$PATH:/opt/intellij/bin/' >> $WORKDIR/.bashrc 

USER $USER

# Download and install intellij scala plugin

RUN mkdir -p $WORKDIR/.IdeaIC$INTELLIJ_VER/config/plugins && \
    wget https://plugins.jetbrains.com/plugin/download?updateId=$INTELLIJ_VERID -O $WORKDIR/.IdeaIC$INTELLIJ_VER/config/plugins/scalaplugin.zip -q && \    
    cd $WORKDIR/.IdeaIC$INTELLIJ_VER/config/plugins/ && \
    unzip -o -q scalaplugin.zip && \
    rm scalaplugin.zip

####### Eclipse ###########################################

USER root
WORKDIR /opt

RUN wget https://github.com/gnu-mcu-eclipse/org.eclipse.epp.packages/releases/download/v4.5.1-20190101-2018-12/20190101-2023-gnumcueclipse-4.5.1-2018-12-R-linux.gtk.x86_64.tar.gz -O eclipse.tar.gz -q  && \
    tar -xvzf eclipse.tar.gz && \
    rm eclipse.tar.gz

ENV PATH /opt/eclipse:$PATH
RUN echo 'export PATH=/opt/eclipse:$PATH' >> $WORKDIR/.bashrc


########## add project templates and project tool (project.bash) ######
USER $USER

#RUN git clone https://github.com/plex1/SpinalDevTemplateStandalone.git $WORKDIR/projects/user/SpinalDevTemplateStandalone

#RUN git clone https://github.com/plex1/SpinalDevTemplateSoc.git $WORKDIR/projects/user/SpinalDevTemplateSoc

USER root
COPY ./files/project.bash $WORKDIR/projects/user/
RUN chown -R $USER $WORKDIR/projects/user/project.bash
RUN chmod +x $WORKDIR/projects/user/project.bash

####### Startup Script ####################################

USER root
WORKDIR /
# run the startup script each time the container is created
COPY ./startup.sh /opt
RUN chmod +x /opt/startup.sh
ENTRYPOINT ["/opt/startup.sh"]




