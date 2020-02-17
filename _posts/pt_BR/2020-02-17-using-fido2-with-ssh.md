---
layout: post
title: Compilando OpenSSH 8.2 e utilizando FIDO2 U2F para autenticacão ssh  
lang: pt_BR
description: Compilando OpenSSH 8.2 e utilizando FIDO2 U2F para autenticacão ssh  
tags: linux ubuntu fedora sshd ssh fido2 u2f
comments: true
--- 

OpenSSH 8.2 acabou de ser lançado com suporte a FIDO2/U2F. Isto é uma adição de segurança boa!

Como este ainda não está no repositório oficial do Fedora, nós teremos que compilar o openssh 8.2 se nós quisermos testar.

OpenSSH 8.2 precisa da bibilioteca libfido2 que por sua vez precisa das bibliotecas libcor e systemd-devel. Como não existe pacote pronto para biblioteca libfido2 teremos que compilar a mesma também.

Vamos começar instalando algumas dependencias:
```
$ sudo dnf group install 'Development Tools'
$ sudo dnf install libselinux-devel libselinux libcbor libcbor-devel systemd-devel cmake
```

Para instalar a biblioteca libfido2:
```
$ git clone git@github.com:Yubico/libfido2.git
$ cd libfido2
$ (rm -rf build && mkdir build && cd build && cmake ..)
$ make -C build
$ sudo make -C build install
```
Basicamente estamos clonando o código fonte e utilizando os comandos descritos no repositório para instalar o mesmo.


Com as dependencies prontas vamos preparar o openssh-8.2
```
$ mkdir openssl-8
$ cd openssl-8
$ mkdir test-openssh
$ wget http://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/openssh-8.2p1.tar.gz
$ tar xvzf openssh-8.2p1.tar.gz
$ cd openssh-8.2p1
``` 

Com o código no lugar, vamos configurar o build:
```
$ ./configure --with-security-key-builtin --with-md5-passwords --with-selinux --with-privsep-path=$HOME/openssl-8/test-openssh --sysconfdir=$HOME/openssl-8/test-openssh --prefix=$HOME/openssl-8/test-openssh
```
Nota: `--with-security-key-builtin` é importante para poder suportar o FIDO2. Este comando vai utilizar o caminho `$HOME/openssl-8/test-openssh` a idéia é colocar os binários em um lugar diferente do OS para testarmos sem afetar o ssh local.


Após isso podemos compilar e instalar
```
$ make
$ make install
```

Também precisamos criar uma regra udev:
```
$ sudo vim /etc/udev/rules.d/90-fido.rules
```

Com o seguinte conteudo:
```
KERNEL=="hidraw*", SUBSYSTEM=="hidraw", \
  MODE="0664", GROUP="plugdev", ATTRS{idVendor}=="1050"
```

Após isso entre na pasta onde os binários estarão:
```
$ cd $HOME/openssl-8/test-openssh/bin
```

Para rodar os binários você deve usar `./` caso contrário o binário utilizado será o do sistema. Eu não tenho certeza do por que, mas quando utilizei o ssh-keygen estava tendo alguns problemas com a libfido2.so.2
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

Para solucionar isso eu simplesmente copiei a libfido2 para o seguinte caminho: "/usr/lib64/libfido2.so.2"

Então quando rodei o mesmo novamente sem o token no computador:
```
$ ./ssh-keygen -t ecdsa-sk -f /tmp/test_ecdsa_sk
Generating public/private ecdsa-sk key pair.
You may need to touch your authenticator to authorize key generation.
Key enrollment failed: device not found
```

Depois de plugar o mesmo:
```
$ ./ssh-keygen -t ecdsa-sk -f /tmp/test_ecdsa_sk
Generating public/private ecdsa-sk key pair.
You may need to touch your authenticator to authorize key generation.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /tmp/test_ecdsa_sk
Your public key has been saved in /tmp/test_ecdsa_sk.pub
The key fingerprint is:
SHA256:.../... host@boom
```

A chave foi gerada com sucesso!

Agora precisamos de um servidor que tenha suporte a isso portanto eu criei um Dockerfile com ubuntu:20.04 rodando um sshd e openssh 8.2

A decisão de usar ubuntu:20.04 foi dado que ele possui libfido2 num repositório extra.
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

Para compilar e rodar:
``` 
$ docker build -t ubuntussh .
$ docker run -p 2222:22 -v /tmp/test_ecdsa_sk.pub:/root/.ssh/authorized_keys -it ubuntussh bash
```

Agora estamos dentro da instancia do docker e rodei os seguintes comandos:
```
$ chown -R root:root ~/.ssh/
$ /usr/local/sbin/sshd
```


Abra um novo terminal e entre no diretório com os binários do ssh já compilados:

```
SSH_AUTH_SOCK= ./ssh -o "PasswordAuthentication=no" -o "IdentitiesOnly=yes" -i /tmp/test_ecdsa_sk root@localhost -p 2222
```

A opcão `SSH_AUTH_SOCK` é para evitar a utilização do ssh-agent que já está rodando, -i é para especificar a chave que desejamos utilizar

```
Enter passphrase for key '/tmp/id_ecdsa_sk': 
Confirm user presence for key ECDSA-SK SHA256:bsIjeSdrNiB4FhxfYBoHH2sCXLiISu9sxDFNrFLgBwY
```

Agora estamos no ubuntussh com FIDO2 e senha!

Espero que seja util,
Matheus

Reference:
[https://bugs.archlinux.org/task/65513](https://bugs.archlinux.org/task/65513)
[https://github.com/Yubico/libfido2](https://github.com/Yubico/libfido2)
[https://www.openssh.com/txt/release-8.2](https://www.openssh.com/txt/release-8.2)
[http://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/](http://cdn.openbsd.org/pub/OpenBSD/OpenSSH/portable/)