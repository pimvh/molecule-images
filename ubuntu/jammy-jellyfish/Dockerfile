# Inspired on a combination of:
# - https://github.com/ansible-community/molecule-plugins/blob/main/src/molecule_plugins/docker/playbooks/Dockerfile.j2
# - https://gitlab.com/aussielunix/ansible/molecule-containers

# Jammy Jellyfish
FROM ubuntu:22.04

LABEL maintainer="Pim van Helvoirt"
LABEL build_date="!!REPLACE_DATE"

# Create `ansible` user with sudo permissions and membership in `DEPLOY_GROUP`
ENV ANSIBLE_USER=ansible DEPLOY_GROUP=deployer
ENV container=docker
ENV SUDO_GROUP=sudo

ENV LANG en_US.utf8
ENV LC_ALL C.UTF-8

# non interactive debian
ENV DEBIAN_FRONTEND=noninteractive

# Do specific actions per type of distribution
# For example for Ubuntu, generate a locate in addition to install a number of packages
RUN apt-get update && \
    apt-get install -y locales && \
    localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8 && \
    apt-get -y install --no-install-recommends systemd systemd-sysv \
    sudo procps python3-pip python3-dev python3-setuptools python3-wheel bash iproute2 ca-certificates \
    # Clean up
    && rm -rf /var/lib/apt/lists/* \
    && rm -Rf /usr/share/doc && rm -Rf /usr/share/man \
    && apt-get clean -y

# Create a group and a user
RUN set -xe \
  && groupadd -r ${ANSIBLE_USER} \
  && groupadd -r ${DEPLOY_GROUP} \
  && useradd -m -g ${ANSIBLE_USER} ${ANSIBLE_USER} \
  && usermod -aG ${SUDO_GROUP} ${ANSIBLE_USER} \
  && usermod -aG ${DEPLOY_GROUP} ${ANSIBLE_USER} \
  && sed -i "/^%${SUDO_GROUP}/s/ALL\$/NOPASSWD:ALL/g" /etc/sudoers

# Upgrade pip to latest version to avoid wheel / cryptography issues
RUN pip3 install --upgrade pip

# Install Ansible via pip.
RUN pip3 install ansible-core ansible-lint cryptography

# Switch back to dialog for any ad-hoc use of apt-get
ENV DEBIAN_FRONTEND=dialog

RUN rm -f /lib/systemd/system/multi-user.target.wants/* \
    /etc/systemd/system/*.wants/* \
    /lib/systemd/system/local-fs.target.wants/* \
    /lib/systemd/system/sockets.target.wants/*udev* \
    /lib/systemd/system/sockets.target.wants/*initctl* \
    /lib/systemd/system/sysinit.target.wants/systemd-tmpfiles-setup* \
    /lib/systemd/system/systemd-update-utmp*

CMD ["/lib/systemd/systemd", "log-level=info", "unit=sysinit.target"]
