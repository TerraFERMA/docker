FROM quay.io/cianwilson/terrafermatestbase
MAINTAINER terraferma <terraferma@lists.columbia.edu>

USER root
WORKDIR /tmp

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
                       python \
                       python-numpy \
                       python-scipy \
                       python-matplotlib \
                       python-ply \
                       python-sympy \
                       python-pytest \
                       python-setuptools \
                       swig3.0 \
                       libeigen3-dev \
                       libboost-dev \
                       libboost-filesystem-dev \
                       libboost-program-options-dev \
                       libboost-iostreams-dev \
                       libboost-timer-dev \
                       libvtk5-dev \
                       python-vtk \
                       libvtk-java \
                       libvtk5-qt4-dev \
                       libhdf5-openmpi-dev \
                       gmsh && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV PETSC_PREFIX /usr/local/petsc
RUN git clone -q https://bitbucket.org/tferma/petsc.git && \
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

ENV FENICS_PREFIX /usr/local/fenics
RUN mkdir -p $FENICS_PREFIX/lib/python2.7/site-packages
ENV PYTHONPATH $PYTHONPATH:$FENICS_PREFIX/lib/python2.7/site-packages

RUN git clone -q https://bitbucket.org/tferma/ufl.git && \
    cd ufl && \
    python setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*
RUN git clone -q https://bitbucket.org/tferma/fiat.git && \
    cd fiat && \
    python setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*
RUN git clone -q https://bitbucket.org/tferma/dijitso.git && \
    cd dijitso && \
    python setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*
RUN git clone -q https://bitbucket.org/tferma/instant.git && \
    cd instant && \
    python setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*
RUN git clone -q https://bitbucket.org/tferma/ffc.git && \
    cd ffc && \
    python setup.py install --prefix=$FENICS_PREFIX && \
    rm -rf /tmp/*

ENV PETSC_DIR $PETSC_PREFIX
ENV PATH $FENICS_PREFIX/bin:$PATH

RUN git clone -q https://bitbucket.org/tferma/dolfin.git && \
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

ENV LD_LIBRARY_PATH $FENICS_PREFIX/lib:$LD_LIBRARY_PATH
ENV PKG_CONFIG_PATH $FENICS_PREFIX/lib/pkgconfig:$PKG_CONFIG_PATH

RUN bash -c "source $FENICS_PREFIX/share/dolfin/dolfin.conf"


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
    rm -rf /tmp/*

ENV NO_AT_BRIDGE 1

WORKDIR $TF_HOME

USER tfuser
RUN echo 'source ~/.profile' >> $TF_HOME/.bash_profile && \
    echo '. ~/fenics.env.conf' >> $TF_HOME/.profile
COPY WELCOME $TF_HOME/WELCOME

USER root
