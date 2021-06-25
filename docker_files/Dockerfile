FROM centos:centos7
MAINTAINER Martin Hovmöller

RUN yum -y install openssh-server openssh-clients
RUN systemctl enable sshd
RUN yum -y install vi
RUN rm -rf /var/lib/apt/lists/* && \
    mkdir -p /var/run/sshd
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
ADD . /root
WORKDIR /root
COPY ssh_config /etc/ssh/ssh_config
COPY sshd_config /etc/ssh/sshd_config
COPY create-sftp-user /usr/local/bin/
RUN ["chmod", "+x", "/usr/local/bin/create-sftp-user"]
COPY entrypoint /usr/local/bin/
RUN ["chmod", "+x", "/usr/local/bin/entrypoint"]
EXPOSE 2222
ENTRYPOINT ["/usr/local/bin/entrypoint"]