FROM fedora

# Install updates and required base packages
RUN yum update -y
RUN yum install wget -y
RUN yum groupinstall fonts -y

# Install Google Chrome repository
ADD google-chrome.repo /etc/yum.repos.d/ 
RUN wget -q https://dl-ssl.google.com/linux/linux_signing_key.pub
RUN rpm --import linux_signing_key.pub
RUN rm linux_signing_key.pub

# Install Chrome, SSHD, and other required packages
RUN yum install google-chrome-stable openssh-server openssh-clients xorg-x11-xauth passwd pwgen hostname -y

# Initialize SSHD and create the webuser user
RUN /usr/libexec/openssh/sshd-keygen ecdsa
RUN /usr/libexec/openssh/sshd-keygen rsa
RUN /usr/libexec/openssh/sshd-keygen ed25519
RUN useradd webuser

EXPOSE 22
CMD echo -n "Initializing..."; IP=$(hostname -i); PW=$(pwgen -s 16 1); echo "$PW" | passwd --stdin webuser > /dev/null; echo -e "\r               \rIP address:\t$IP\nPassword:\t$PW\nGoogle Chrome:\tssh -X webuser@$IP google-chrome --no-sandbox"; /usr/sbin/sshd -D
