FROM terraferma/dev-env:latest
MAINTAINER terraferma <terraferma@lists.columbia.edu>

USER root
WORKDIR /tmp

ENV TF_PREFIX /usr/local/terraferma

RUN export PYTHONPATH=$FENICS_PREFIX/lib/python2.7/site-packages && \
    export PETSC_DIR=$PETSC_PREFIX && \
    export PATH=$FENICS_PREFIX/bin:$PATH && \
    export LD_LIBRARY_PATH=$FENICS_PREFIX/lib:$LD_LIBRARY_PATH && \
    export PKG_CONFIG_PATH=$FENICS_PREFIX/lib/pkgconfig:$PKG_CONFIG_PATH && \
    git clone -q https://github.com/TerraFERMA/TerraFERMA.git && \
    cd TerraFERMA && \
    mkdir build && \
    cd build && \
    cmake -DCMAKE_INSTALL_PREFIX=${TF_PREFIX} \
          ../. && \
    make && \
    make install && \
    rm -rf /tmp/*

COPY terraferma.configmodule /usr/local/share/environment-modules/terraferma

WORKDIR $TF_HOME

USER tfuser
COPY bash.load-modules $TF_HOME/.bash.load-modules
RUN cat $TF_HOME/.bash.load-modules >> $TF_HOME/.bashrc
COPY WELCOME $TF_HOME/WELCOME

USER root


