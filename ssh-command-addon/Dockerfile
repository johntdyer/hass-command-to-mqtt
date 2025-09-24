# https://developers.home-assistant.io/docs/add-ons/configuration#add-on-dockerfile
ARG BUILD_FROM
FROM $BUILD_FROM

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install requirements for add-on
RUN \
    apk add --no-cache \
        python3 \
        py3-pip \
        openssh-client \
    && pip3 install --no-cache-dir \
        paho-mqtt \
        paramiko \
        pyyaml

# Copy root filesystem
COPY rootfs /

# Make run script executable
RUN chmod a+x /usr/bin/ssh-command-mqtt

# Set the default command
CMD [ "/usr/bin/ssh-command-mqtt" ]