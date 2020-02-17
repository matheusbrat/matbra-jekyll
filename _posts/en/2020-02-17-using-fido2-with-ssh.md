---
layout: post
title: Building OpenSSH 8.2 and using FIDO2 U2F on ssh authentication  
lang: en
description: Building OpenSSH 8.2 and generate key with FIDO2 support and sshd.
tags: linux ubuntu fedora sshd ssh fido2 u2f
comments: true
--- 

OpenSSH 8.2 was just released with support for FIDO2 U2F keys. This is a nice extra layer for security! 

As this is not yet on official repository for Fedora, we will need to build openssh 8.2 if we want to test.

OpenSSH 8.2 needs libfido2 and libfido2 needs libcbor systemd-devel. There is no package for FIDO2 on Fedora 31 yet, therefore we also need to build it. 

Let's start installing some dependencies:

```
$ sudo dnf group install 'Development Tools'
$ sudo dnf install libselinux-devel libselinux libcbor libcbor-devel systemd-devel cmake
```

To install libfido:
```
$ git clone git@github.com:Yubico/libfido2.git
$ cd libfido2
$ (rm -rf build && mkdir build && cd build && cmake ..)
$ make -C build
$ sudo make -C build install
```
Here we are cloning the code and basically using their commands to install it.


With this dependency ready let's get openssh-8.2:
```
$ mkdir openssl-8
$ cd openssl-8
$ mkdir test-openssh
$ wget http://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-8.2p1.tar.gz
$ tar xvzf openssh-8.2p1.tar.gz
$ cd openssh-8.2p1
``` 

With the code in place we will use configure to prepare it:
```
$ ./configure --with-security-key-builtin --with-md5-passwords --with-selinux --with-privsep-path=$HOME/openssl-8/test-openssh --sysconfdir=$HOME/openssl-8/test-openssh --prefix=$HOME/openssl-8/test-openssh
```
Note: `--with-security-key-builtin` is important to have support for FIDO2 internally. This command will prepare the path as `$HOME/openssl-8/test-openssh` my idea here is to avoid messing with my existing ssh.


After this is completed we can make/make install
```
$ make
$ make install
```

I also had to create a udev rule:
```
$ sudo vim /etc/udev/rules.d/90-fido.rules
```

With this content:
```
KERNEL=="hidraw*", SUBSYSTEM=="hidraw", \
  MODE="0664", GROUP="plugdev", ATTRS{idVendor}=="1050"
```

After all this I entered on the binary folder
```
$ cd $HOME/openssl-8/test-openssh/bin
```

To run the binary we must use `./` otherwise it will use the other binary which are system wide and we want to run the exact one which we just build. I'm not exactly sure why, but when I was running ssh-keygen, I was having some issues to find the libfido2.so.2
```
$ ./ssh-keygen -t ecdsa-sk -f /tmp/test_ecdsa_sk
Generating public/private ecdsa-sk key pair.
You may need to touch your authenticator to authorize key generation.
/home/matheus/openssl-8/test-openssh/libexec/ssh-sk-helper: error while loading shared libraries: libfido2.so.2: cannot open shared object file: No such file or directory
ssh_msg_recv: read header: Connection reset by peer
client_converse: receive: unexpected internal error
reap_helper: helper exited with non-zero exit status
Key enrollment failed: unexpected internal error
```

In my case I found the location of this file and copied it to "/usr/lib64/libfido2.so.2"

After this when running the command to generate it without the fido2 plugged in I got:
```
$ ./ssh-keygen -t ecdsa-sk -f /tmp/test_ecdsa_sk
Generating public/private ecdsa-sk key pair.
You may need to touch your authenticator to authorize key generation.
Key enrollment failed: device not found
```

Plugin the key in and trying again
```
$ ./ssh-keygen -t ecdsa-sk -f /tmp/test_ecdsa_sk
Generating public/private ecdsa-sk key pair.
You may need to touch your authenticator to authorize key generation.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in -f /tmp/test_ecdsa_sk/test_ecdsa_sk
Your public key has been saved in -f /tmp/test_ecdsa_sk/test_ecdsa_sk.pub
The key fingerprint is:
SHA256:.../... host@boom
```

The key was generated succesfully!! 

Now, I needed a server which supports this. Therefore I created a dockerfile from ubuntu:20.04 with an sshd running and openssh 8.2

I'm using ubuntu:20.04 as it has libfido2 on apt and libcbor too.
```
FROM ubuntu:20.04
RUN apt-get update && apt-get -y install software-properties-common build-essential zlib1g-dev libssl-dev libcbor-dev wget
RUN apt-add-repository -y ppa:yubico/stable && apt-get update && apt-get -y install libfido2-dev
RUN apt-get -y install ssh && apt-get -y remove ssh
RUN wget http://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-8.2p1.tar.gz
RUN tar xvzf openssh-8.2p1.tar.gz
RUN cd openssh-8.2p1 && ./configure --with-security-key-builtin --with-md5-passwords && make && make install 
EXPOSE 22
CMD ["/usr/local/sbin/sshd", "-D"]
```

To build and run this:
``` 
$ docker build -t ubuntussh .
$ docker run -p 2222:22 -v /tmp/test_ecdsa_sk.pub:/root/.ssh/authorized_keys -it ubuntussh bash
```

Now you will be inside the docker instance and I had to chown the authorized key file and run the sshd:
```
$ chown -R root:root ~/.ssh/
$ /usr/local/sbin/sshd
```


Open a new terminal and cd into the openssl 8 bin folder again. 

```
SSH_AUTH_SOCK= ./ssh -o "PasswordAuthentication=no" -o "IdentitiesOnly=yes" -i /tmp/test_ecdsa_sk root@localhost -p 2222
```

The `SSH_AUTH_SOCK` is to avoid using the ssh-agent which is already running, -i to specify exactly the key I want to use. 

This outputs:
```
Enter passphrase for key '/tmp/test_ecdsa_sk': 
Confirm user presence for key ECDSA-SK SHA256:bsIjeSdrNiB4FhxfYBoHH2sCXLiISu9sxDFNrFLgBwY
```

Now we are in the ubuntussh with FIDO2+password! 

Hope this helps you,
Matheus

Reference:
[https://bugs.archlinux.org/task/65513](https://bugs.archlinux.org/task/65513)
[https://github.com/Yubico/libfido2](https://github.com/Yubico/libfido2)
[https://www.openssh.com/txt/release-8.2](https://www.openssh.com/txt/release-8.2)
[http://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/](http://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/)