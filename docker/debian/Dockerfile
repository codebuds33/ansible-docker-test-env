FROM debian:buster

RUN apt-get update && apt-get upgrade -y

RUN apt-get -y install openssh-client openssh-server sudo python3 python3-pip

RUN adduser --uid 1000 --home /home/exploit exploit
RUN adduser exploit exploit

RUN addgroup exploit sudo

RUN mkdir /home/exploit/.ssh

RUN chown exploit:exploit -R /home/exploit/.ssh

COPY ./authorized_keys /home/exploit/.ssh/authorized_keys

RUN chown exploit:exploit -R /home/exploit /home/exploit/.ssh

RUN grep -qxF 'exploit ALL=(ALL) NOPASSWD:ALL' /etc/sudoers || echo 'exploit ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

RUN mkdir -p /run/sshd

RUN ssh-keygen -A

RUN  sed -i "s/\(#MaxAuthTries  *\).*/MaxAuthTries 10000/" /etc/ssh/sshd_config
RUN  sed -i "s/\(#PasswordAuthentication  *\).*/PasswordAuthentication no/" /etc/ssh/sshd_config

EXPOSE 22

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["sh", "/entrypoint.sh"]
