FROM quay.io/cianwilson/terrafermatestdevenv
MAINTAINER terraferma <terraferma@lists.columbia.edu>

USER root
WORKDIR /tmp

ENV TF_PREFIX /usr/local/terraferma

RUN bash -c "source $FENICS_PREFIX/share/dolfin/dolfin.conf" && \
    git clone -q https://github.com/TerraFERMA/TerraFERMA.git && \
    cd TerraFERMA && \
    mkdir build && \
    cd build && \
    cmake -DCMAKE_INSTALL_PREFIX=${TF_PREFIX} \
          ../. && \
    make && \
    make install && \
    rm -rf /tmp/*

WORKDIR $TF_HOME

USER tfuser
COPY WELCOME $TF_HOME/WELCOME

USER root


