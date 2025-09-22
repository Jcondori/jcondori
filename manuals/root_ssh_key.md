# Autenticación con ssh key para root

###  
```bash
sudo install -d -m 700 -o root -g root /root/.ssh
```

### Pega aquí la clave pública, normalmente inicia con `ssh-rsa AAAA...`
```bash
sudo nano /root/.ssh/authorized_keys
```

### Se establecen los permisos correctos
```bash
sudo chmod 600 /root/.ssh/authorized_keys
sudo chown root:root /root/.ssh/authorized_keys
```

###  
```bash
sudo nano /etc/ssh/sshd_config.d/99-root-login.conf
```
#### contenido:
```bash
PermitRootLogin prohibit-password
PubkeyAuthentication yes
PasswordAuthentication no
```

###  
```bash
sudo sshd -t && sudo systemctl reload ssh
```