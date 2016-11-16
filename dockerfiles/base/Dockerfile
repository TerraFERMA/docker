FROM phusion/baseimage:0.9.19 
MAINTAINER terraferma <terraferma@lists.columbia.edu>

# Get Ubuntu updates
USER root
RUN apt-get update && \ 
    apt-get upgrade -y -o Dpkg::Options::="--force-confold" -o Dpkg::Options::="--force-confdef" && \
    apt-get -y install locales sudo && \
    apt-get clean && \
    echo "C.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Set locale environment
ENV LC_ALL=C.UTF-8 \
    LANG=C.UTF-8 \
    LANGUAGE=C.UTF-8

COPY set-home-permissions.sh /etc/my_init.d/set-home-permissions.sh

# Set up user so that we do not run as root
# See https://github.com/phusion/baseimage-docker/issues/186
# Disable forward logging
# Add script to set up permissions of home directory on myinit
# Run ldconfig so that /usr/local/lib is in the default search
# path for the dynamic linker.
RUN useradd -m -s /bin/bash -G sudo,docker_env tfuser && \
    echo "tfuser:docker" | chpasswd && \
    echo "tfuser ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers && \
    touch /etc/service/syslog-forwarder/down && \
    echo "cat /home/tfuser/WELCOME" >> /home/tfuser/.bashrc && \
    chmod +x /etc/my_init.d/set-home-permissions.sh && \
    ldconfig

USER tfuser
ENV TF_HOME /home/tfuser
RUN touch $TF_HOME/.sudo_as_admin_successful && \
    mkdir $TF_HOME/shared
VOLUME /home/tfuser/shared

COPY WELCOME $TF_HOME/WELCOME

WORKDIR /home/tfuser
USER root
ENTRYPOINT ["/sbin/my_init","--quiet","--","/sbin/setuser","tfuser","/bin/bash","-l","-c"]
CMD ["/bin/bash","-i"]

