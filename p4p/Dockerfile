FROM centos:7
LABEL maintainer "ueno.s <ueno.s@gamestudio.co.jp>"
ENV container docker

# --------------------------------------------------------------------------------
# Docker CentOS 7 install & setup
# --------------------------------------------------------------------------------

# See: https://hub.docker.com/_/centos/
RUN cd /lib/systemd/system/sysinit.target.wants/ && \
    for i in *; do \
        [ $i == systemd-tmpfiles-setup.service ] || rm -vf $i ; \
    done ; \
    rm -vf /lib/systemd/system/multi-user.target.wants/* && \
    rm -vf /etc/systemd/system/*.wants/* && \
    rm -vf /lib/systemd/system/local-fs.target.wants/* && \
    rm -vf /lib/systemd/system/sockets.target.wants/*udev* && \
    rm -vf /lib/systemd/system/sockets.target.wants/*initctl* && \
    rm -vf /lib/systemd/system/basic.target.wants/* && \
    rm -vf /lib/systemd/system/anaconda.target.wants/* && \
    mkdir -p /etc/selinux/targeted/contexts/ && \
    echo '<busconfig><selinux></selinux></busconfig>' > /etc/selinux/targeted/contexts/dbus_contexts \
    && yum clean all --enablerepo='*' \
    && yum clean metadata --enablerepo='*' \
    && yum clean all --enablerepo='*' \
    && rm -rf /var/cache/yum \
    && rpm --import https://package.perforce.com/perforce.pubkey \
    && yum-config-manager --add-repo https://gist.githubusercontent.com/hexagrimoire/f235f327baacdf6f109903eec5d9a717/raw/f54bb09d6a73d390762feaf0ef3344d128677dc9/perforce.centos.repo \
    && yum install -y tzdata helix-p4d helix-proxy logrotate cronie sudo \
    && ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime \
    && localedef -i /usr/share/i18n/locales/ja_JP -f UTF-8 /usr/lib/locale/ja_JP.UTF-8 \
    && localedef --add-to-archive /usr/lib/locale/ja_JP.UTF-8 \
    && source /etc/locale.conf

# Add external files
COPY files/run.sh /usr/local/bin/run.sh
COPY files/p4p /etc/logrotate.d/p4p
RUN chmod +x /usr/local/bin/run.sh

# --------------------------------------------------------------------------------
# P4P ENVIRONMENT
# --------------------------------------------------------------------------------
ARG P4HOME=/opt/perforce/servers
ARG P4PLOGDIR=${P4HOME}/log

ENV LANG="ja_JP.UTF-8" \
    LANGUAGE="ja_JP:ja" \
    LC_ALL="ja_JP.UTF-8" \
    P4PCACHE=${P4HOME}/p4p-master \
    P4SSLDIR=${P4HOME}/ssl \
    P4PLOGFILE=${P4PLOGDIR}/p4p.log

RUN sudo -E -u perforce mkdir -p ${P4PCACHE} \
    && sudo -E -u perforce mkdir -p ${P4SSLDIR} \
    && sudo -E -u perforce mkdir -p ${P4PLOGDIR} \
    && sudo -E -u perforce chmod 700 ${P4SSLDIR} \
    && chmod 644 /etc/logrotate.d/p4p \
    && /usr/sbin/logrotate /etc/logrotate.conf \
    && systemctl enable crond

CMD ["/sbin/init"]
