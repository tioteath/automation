# Vagrant friendly ubuntu trusty
# Based on https://raw.githubusercontent.com
# /tianon/dockerfiles/master/sbin-init/ubuntu/upstart/14.04/Dockerfile
FROM ubuntu:trusty

ENV container docker
ENV DEBIAN_FRONTEND noninteractive
ENV NOTVISIBLE "in users profile"
EXPOSE 20-60179

RUN  printf -- 'description     "In-Container Upstart Fake Events"\n\n \
start on startup\n\n \
script\n \
    rm -rf /var/run/*.pid\n \
    rm -rf /var/run/network/*\n \
    /sbin/initctl emit stopped JOB=udevtrigger --no-wait\n \
    /sbin/initctl emit started JOB=udev --no-wait\n \
    /sbin/initctl emit runlevel RUNLEVEL=3 --no-wait\n \
end script\n' \
> /etc/init/fake-container-events.conf

RUN set -e; \
    apt-get update; \
    apt-get --no-install-recommends --yes install openssh-server curl sudo \
        ca-certificates locales; \
    locale-gen en_US en_US.UTF-8; \
    dpkg-reconfigure locales; \
    sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' \
        -i /etc/pam.d/sshd; \
    useradd --system --create-home --user-group --shell /bin/bash vagrant; \
    printf -- 'vagrant:vagrant' | chpasswd; \
    su --login --command 'umask 0077; \
        mkdir .ssh; \
        curl --silent \
        https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub \
        > .ssh/authorized_keys' -- vagrant; \
    umask 0077 && printf -- 'vagrant ALL=(ALL) NOPASSWD: ALL' \
        > /etc/sudoers.d/vagrant; \
    printf -- "export VISIBLE=now" >> /etc/profile; \
    rm /usr/sbin/policy-rc.d; \
    rm /sbin/initctl; \
    dpkg-divert --rename --remove /sbin/initctl; \
    /usr/sbin/update-rc.d -f ondemand remove; \
    for file in \
        /etc/init/u*.conf \
        /etc/init/mounted-dev.conf \
        /etc/init/mounted-proc.conf \
        /etc/init/mounted-run.conf \
        /etc/init/mounted-tmp.conf \
        /etc/init/mounted-var.conf \
        /etc/init/hostname.conf \
        /etc/init/networking.conf \
        /etc/init/tty*.conf \
        /etc/init/plymouth*.conf \
        /etc/init/hwclock*.conf \
        /etc/init/module*.conf; do \
      dpkg-divert --local --rename --add "$file"; \
    done; \
    cat /dev/null > /lib/init/fstab

CMD ["/sbin/init"]
