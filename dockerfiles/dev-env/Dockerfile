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
FROM terraferma/base:fenics-2019.1.0
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
                       flex \
                       libblas-dev \
                       liblapack-dev \
                       libeigen3-dev \
                       libboost-dev \
                       libboost-filesystem-dev \
                       libboost-program-options-dev \
                       libboost-iostreams-dev \
                       libboost-timer-dev \
                       gmsh \
                       wget \
                       gir1.2-gtksource-3.0 \
                       gir1.2-gtk-3.0 \
                       tcl \
                       vim \
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
                       python3 \
                       python3-numpy \
                       python3-scipy \
                       python3-sympy \
                       python3-matplotlib \
                       python3-ply \
                       python3-pytest \
                       python3-setuptools \
                       python3-h5py \
                       python3-mpi4py \
                       python3-dev \
                       python3-future \
                       python3-gi \
                       python3-lxml \
                       cython3 \
                       python3-cairocffi \
                       python3-cairo \
                       python3-gi-cairo \
                       python3-pip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# download and install openmpi from source
ENV OPENMPI_PREFIX /usr/local/openmpi
RUN wget https://download.open-mpi.org/release/open-mpi/v3.1/openmpi-3.1.2.tar.bz2 && \
    tar -xjf openmpi-3.1.2.tar.bz2 && \
    cd openmpi-3.1.2 && \
    ./configure --prefix=$OPENMPI_PREFIX && \
    make && \
    make install && \
    rm -rf /tmp/*
# install an environment module for openmpi
COPY openmpi.configmodule /usr/local/share/environment-modules/openmpi

# download and install hdf5 from source
ENV HDF5_PREFIX /usr/local/hdf5
RUN export PATH=$OPENMPI_PREFIX/bin:$PATH && \
    export LD_LIBRARY_PATH=$OPENMPI_PREFIX/lib:$LD_LIBRARY_PATH && \
    export INCLUDE=$OPENMPI_PREFIX/include:$INCLUDE && \
    wget https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.4/src/hdf5-1.10.4.tar.bz2 && \
    tar -xjf hdf5-1.10.4.tar.bz2 && \
    cd hdf5-1.10.4 && \
    ./configure --enable-parallel --enable-fortran --prefix=$HDF5_PREFIX && \
    make && \
    make install && \
    rm -rf /tmp/*
# install an environment module for openmpi
COPY hdf5.configmodule /usr/local/share/environment-modules/hdf5

# download and install pybind11 from source
ENV PYBIND11_PREFIX /usr/local/pybind11
RUN mkdir -p $PYBIND11_PREFIX/lib/python3.6/site-packages
RUN export PYTHONPATH=$PYBIND11_PREFIX/lib/python3.6/site-packages && \
    git clone -b v2.2.4 -q https://github.com/pybind/pybind11.git && \
    cd pybind11 && \
    mkdir build && \
    cd build && \
    cmake -DCMAKE_INSTALL_PREFIX=${PYBIND11_PREFIX} ../. && \
    make && \
    make install && \
    rm -rf /tmp/*
# install an environment module for pybind11
COPY pybind11.configmodule /usr/local/share/environment-modules/pybind11

# download and install petsc from source
ENV PETSC_PREFIX /usr/local/petsc
RUN export PATH=$OPENMPI_PREFIX/bin:$PATH && \
    export LD_LIBRARY_PATH=$OPENMPI_PREFIX/lib:$LD_LIBRARY_PATH && \
    export INCLUDE=$OPENMPI_PREFIX/include:$INCLUDE && \
    git clone -b v3.11.3 -q https://gitlab.com/petsc/petsc.git && \
    cd petsc && \
    python3 ./configure --COPTFLAGS="-O2" \
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
                        --download-petsc4py=1 \
                        --with-debugging=0 \
                        --prefix=$PETSC_PREFIX && \
    make all && \
    make install && \
    rm -rf /tmp/*
# install an environment module for petsc
COPY petsc.configmodule /usr/local/share/environment-modules/petsc

# download and install all the fenics components from source
ENV FENICS_PREFIX /usr/local/fenics
RUN mkdir -p $FENICS_PREFIX/lib/python3.6/site-packages
# ufl
RUN export PYTHONPATH=$FENICS_PREFIX/lib/python3.6/site-packages && \
    git clone -b tferma-master-2019.1.0 -q https://bitbucket.org/tferma/ufl.git && \
    cd ufl && \
    python3 setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*
# fiat
RUN export PYTHONPATH=$FENICS_PREFIX/lib/python3.6/site-packages && \
    git clone -b tferma-master-2019.1.0 -q https://bitbucket.org/tferma/fiat.git && \
    cd fiat && \
    python3 setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*
# dijitso
RUN export PYTHONPATH=$FENICS_PREFIX/lib/python3.6/site-packages && \
    git clone -b tferma-master-2019.1.0 -q https://bitbucket.org/tferma/dijitso.git && \
    cd dijitso && \
    python3 setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*
# ffc
RUN export PYTHONPATH=$FENICS_PREFIX/lib/python3.6/site-packages && \
    git clone -b tferma-master-2019.1.0 -q https://bitbucket.org/tferma/ffc.git && \
    cd ffc && \
    python3 setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*
# dolfin
RUN export PYTHONPATH=$PYBIND11_PREFIX/lib/python3.6/site-packages:$FENICS_PREFIX/lib/python3.6/site-packages && \
    export PETSC_DIR=$PETSC_PREFIX && \
    export PATH=$PYBIND11_PREFIX/bin:$FENICS_PREFIX/bin:$HDF5_PREFIX/bin:$OPENMPI_PREFIX/bin:$PATH && \
    export LD_LIBRARY_PATH=$HDF5_PREFIX/lib:$OPENMPI_PREFIX/lib:$LD_LIBRARY_PATH && \
    export LIBRARY_PATH=$PYBIND11_PREFIX/lib:$HDF5_PREFIX/lib:$LIBRARY_PATH && \
    export INCLUDE=$HDF5_PREFIX/include:$OPENMPI_PREFIX/include:$INCLUDE && \
    export CPATH=$HDF5_PREFIX/include:$CPATH && \
    export HDF5_DIR=$HDF5_PREFIX && \
    export HDF5_INCLUDE_DIR=$HDF5_PREFIX/include && \
    export HDF5_INC=$HDF5_PREFIX/include && \
    export HDF5_LIB=$HDF5_PREFIX/lib && \
    export PKG_CONFIG_PATH=$PYBIND11_PREFIX/lib/pkgconfig:$PKG_CONFIG_PATH && \
    git clone -b tferma-master-2019.1.0 -q https://bitbucket.org/tferma/dolfin.git && \
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
          -DCMAKE_INSTALL_PREFIX=${FENICS_PREFIX} \
          ../. && \
    make && \
    make install && \
    cd ../python && \
    python3 setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*
# install an environment module for fenics
COPY fenics.configmodule /usr/local/share/environment-modules/fenics

# download and install spud from source
ENV SPUD_PREFIX /usr/local/spud
RUN git clone -q https://github.com/FluidityProject/spud.git && \
    cd spud && \
    ./configure --prefix=$SPUD_PREFIX && \
    make && \
    make install && \
    rm -rf /tmp/*
# install an environment module for spud
COPY spud.configmodule /usr/local/share/environment-modules/spud

# supress some spurious gtk errors
ENV NO_AT_BRIDGE 1

WORKDIR $TF_HOME

# load modules at startup
USER tfuser
COPY bash.load-modules $TF_HOME/.bash.load-modules
RUN cat $TF_HOME/.bash.load-modules >> $TF_HOME/.profile
COPY WELCOME $TF_HOME/WELCOME

USER root
