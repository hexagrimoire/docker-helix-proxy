FROM centos:8
LABEL maintainer "ueno.s <ueno.s@gamestudio.co.jp>"

RUN dnf -y install centos-release-stream yum-utils tzdata logrotate cronie sudo glibc-langpack-ja glibc glibc-common \
    && dnf -y swap centos-{linux,stream}-repos \
    && dnf -y distro-sync \
    && rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial \
    && dnf -y upgrade \
    && dnf -y install glibc-locale-source \
    && dnf clean all \
    && cd /lib/systemd/system/sysinit.target.wants/ && \
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
    && yum install -y helix-p4d helix-proxy \
    && ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime \
    && localedef -f UTF-8 -i ja_JP ja_JP.UTF-8 \
    && sed -i -e '/override_install_langs/s/$/,ja_JP.utf8/g' /etc/locale.conf \
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