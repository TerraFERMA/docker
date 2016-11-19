# Modified from the Dockerimage used by FEniCS:
# https://bitbucket.org/fenics-project/docker
#
#
# Copyright (C) 2013 Columbia University in the City of New York and others.
#
# Please see the AUTHORS file in the main source directory for a full list
# of contributors.
#
# This file is part of TerraFERMA.
#
# TerraFERMA is free software: you can redistribute it and/or modify
# it under the terms of the GNU Lesser General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# TerraFERMA is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public License
# along with TerraFERMA. If not, see <http://www.gnu.org/licenses/>.
#
#
# Builds a Docker image with the correct environment for compiling 
# TerraFERMA.  The built image can be found at:
#
#   https://hub.docker.com/r/terraferma/dev-env
#

# build from the base image
FROM terraferma/base:dolfin-master
MAINTAINER terraferma <terraferma@lists.columbia.edu>

USER root
WORKDIR /tmp

# install non-python dependencies - splitting these up reduces the size of individual image layers
RUN apt-get update && \
    apt-get -y --with-new-pkgs \
        -o Dpkg::Options::="--force-confold" -o Dpkg::Options::="--force-confdef" upgrade && \
    apt-get -y install \
                       git \
                       gcc \
                       g++ \
                       gfortran \
                       cmake \
                       openmpi-bin \
                       libopenmpi-dev \
                       flex \
                       libblas-dev \
                       liblapack-dev \
                       swig3.0 \
                       libeigen3-dev \
                       libboost-dev \
                       libboost-filesystem-dev \
                       libboost-program-options-dev \
                       libboost-iostreams-dev \
                       libboost-timer-dev \
                       libvtk5-dev \
                       libvtk-java \
                       libvtk5-qt4-dev \
                       libhdf5-openmpi-dev \
                       gmsh \
                       environment-modules && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# enable environment modules
COPY bash.enable-modules /etc/bash.enable-modules
RUN cat /etc/bash.enable-modules >> /etc/profile

# install all python dependencies
RUN apt-get update && \
    apt-get -y --with-new-pkgs \
        -o Dpkg::Options::="--force-confold" -o Dpkg::Options::="--force-confdef" upgrade && \
    apt-get -y install \
                       python \
                       python-numpy \
                       python-scipy \
                       python-matplotlib \
                       python-ply \
                       python-pytest \
                       python-setuptools \
                       python-vtk && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# download and install petsc from source
ENV PETSC_PREFIX /usr/local/petsc
# NOTE: still using tferma-master branch here
RUN git clone -b tferma-master -q https://bitbucket.org/tferma/petsc.git && \
    cd petsc && \
    ./configure --COPTFLAGS="-O2" \
                --CXXOPTFLAGS="-O2" \
                --FOPTFLAGS="-O2" \
                --with-clanguage=C++ \
                --with-shared-libraries=1 \
                --download-suitesparse=1 \
                --download-hypre=1 \
                --download-ml=1 \
                --download-parmetis=1 \
                --download-metis=1 \
                --download-blacs=1 \
                --download-scalapack=1 \
                --download-mumps=1 \
                --with-debugging=0 \
                --prefix=$PETSC_PREFIX && \
    make all && \
    make install && \
    rm -rf /tmp/*

# install an environment module for petsc
COPY petsc.configmodule /usr/local/share/environment-modules/petsc

# download and install all the fenics components from source
ENV FENICS_PREFIX /usr/local/fenics
RUN mkdir -p $FENICS_PREFIX/lib/python2.7/site-packages
# ufl
RUN export PYTHONPATH=$FENICS_PREFIX/lib/python2.7/site-packages && \
    git clone -b master -q https://bitbucket.org/tferma/ufl.git && \
    cd ufl && \
    python setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*
# fiat
RUN export PYTHONPATH=$FENICS_PREFIX/lib/python2.7/site-packages && \
    git clone -b master -q https://bitbucket.org/tferma/fiat.git && \
    cd fiat && \
    python setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*
# dijitso
RUN export PYTHONPATH=$FENICS_PREFIX/lib/python2.7/site-packages && \
    git clone -b master -q https://bitbucket.org/tferma/dijitso.git && \
    cd dijitso && \
    python setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*
# instant
RUN export PYTHONPATH=$FENICS_PREFIX/lib/python2.7/site-packages && \
    git clone -b master -q https://bitbucket.org/tferma/instant.git && \
    cd instant && \
    python setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*
# ffc
RUN export PYTHONPATH=$FENICS_PREFIX/lib/python2.7/site-packages && \
    git clone -b master -q https://bitbucket.org/tferma/ffc.git && \
    cd ffc && \
    python setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*
# dolfin
RUN export PYTHONPATH=$FENICS_PREFIX/lib/python2.7/site-packages && \
    export PETSC_DIR=$PETSC_PREFIX && \
    export PATH=$FENICS_PREFIX/bin:$PATH && \
    git clone -b master -q https://bitbucket.org/tferma/dolfin.git && \
    cd dolfin && \
    mkdir build && \
    cd build && \
    cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo \
          -DDOLFIN_ENABLE_CHOLMOD:BOOL=OFF \
          -DDOLFIN_ENABLE_UMFPACK:BOOL=OFF \
          -DDOLFIN_ENABLE_TRILINOS:BOOL=OFF \
          -DDOLFIN_ENABLE_SLEPC:BOOL=OFF  \
          -DMETIS_LIBRARY=${PETSC_DIR}/lib/libmetis.so \
          -DPARMETIS_INCLUDE_DIRS=${PETSC_DIR}/include \
          -DPARMETIS_LIBRARY=${PETSC_DIR}/lib/libparmetis.so \
          -DDOLFIN_ENABLE_TESTING:BOOL=OFF \
          -DCMAKE_INSTALL_PREFIX=${FENICS_PREFIX} \
          ../. && \
    make && \
    make install && \
    rm -rf /tmp/*

# install an environment module for fenics
COPY fenics.configmodule /usr/local/share/environment-modules/fenics

# install spud and some more dependencies
RUN apt-add-repository ppa:fluidity-core/ppa && \
    apt-get update -qq && \
    apt-get install -y \
                       libatk-adaptor \
                       libcanberra-gtk-dev \
                       libgail-dev \
                       libspud-dev \
                       diamond \
                       python-spud \
                       spudtools && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# supress some spurious gtk errors
ENV NO_AT_BRIDGE 1

WORKDIR $TF_HOME

# load modules at startup
USER tfuser
COPY bash.load-modules $TF_HOME/.bash.load-modules
RUN cat $TF_HOME/.bash.load-modules >> $TF_HOME/.profile
COPY WELCOME $TF_HOME/WELCOME

USER root
