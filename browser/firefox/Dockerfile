FROM fedora

# Install updates and required base packages
RUN yum update -y
RUN yum install wget -y
RUN yum groupinstall fonts -y

# Install Firefox, SSHD, and other required packages
RUN yum install firefox openssh-server openssh-clients xorg-x11-xauth passwd pwgen hostname -y

# Initialize SSHD and create the webuser user
RUN /usr/libexec/openssh/sshd-keygen ecdsa
RUN /usr/libexec/openssh/sshd-keygen rsa
RUN /usr/libexec/openssh/sshd-keygen ed25519
RUN useradd webuser

EXPOSE 22
CMD echo -n "Initializing..."; IP=$(hostname -i); PW=$(pwgen -s 16 1); echo "$PW" | passwd --stdin webuser > /dev/null; echo -e "\r               \rIP address:\t$IP\nPassword:\t$PW\nFirefox:\tssh -X webuser@$IP firefox"; /usr/sbin/sshd -D
