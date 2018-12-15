# PKCS11-eToken-Ubuntu-18.04

### PKCS11 token ssh/sftp usage Ubuntu/Xubuntu 18.04

---

You should receive deb package from Your workspace or manufacturer website.

SETUP:
```
sudo su -
apt install opensc opensc-pkcs11 gnutls-bin

mkdir -p /etc/pkcs11/modules

echo "module: /usr/lib/libeTPkcs11.so" >> /etc/pkcs11/modules/.module
```


VPN:
```
p11-kit list-modules #find hardware token

p11tool --list-tokens # get url from token where You can find Your manufacter name

p11tool --list-all-certs 'full_url_to_token'

openconnect -c 'full_url_to_cert' --servercert sha256:someHashCode vpnUrl
```

SSH:
```
ssh -I /usr/lib/libeTPkcs11.so user@server
```

SFTP:
```
sftp -oPKCS11Provider=/usr/lib/libeTPkcs11.so user@server
```

---

You are most probably running gnome-keyring-daemon which just pretends to be a normal ssh-agent.
Unfortunately it does not support PKCS11 tokens. You will be met with prompts shown below:
```
$ ssh-add -s /usr/lib/libeTPkcs11.so

Enter passphrase for PKCS#11:
Could not add card "/usr/lib/libeTPkcs11.so": agent refused operation
```

That means You are gonna need to compile open-ssh for Yourself.

I've experienced problems with libraries needed for compilation so I fall back to compiling it on virtual machine with ubuntu 16.04 and open-ssh 7.7p2.

Visit https://www.openssh.com/portable.html and download Your target version

Then in extracted directory of Your downloaded open-ssh
```

# apt install libz-dev libssl-dev
# mkdir -p /var/empty
# mkdir /opt/openssh-pkcs11
# chown $USER:$USER /opt/openssh-pkcs11

# mkdir /var/empty
# chown root:sys /var/empty
# chmod 755 /var/empty
# groupadd sshd
# useradd -g sshd -c 'sshd privsep' -d /var/empty -s /bin/false sshd


$ ./configure --prefix=/opt/openssh-pkcs11
$ make && make install

```

then for normal run:
```
$ eval `/opt/openssh-pkcs11/bin/ssh-agent`

$ ssh-add -s /usr/lib/libeTPkcs11.so
```

Keep in mind that if Your .so object is not in /usr/lib/ or /usr/local/lib/ then You will need to provide whitelist using -P param

Handy debug mode for ssh-agent:
```
$ /opt/openssh-pkcs11/bin/ssh-agent -d
```

