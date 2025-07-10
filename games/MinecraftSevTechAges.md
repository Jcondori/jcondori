# Manual de Instalación: SevTech: Ages

## Requisitos del Sistema

- **Sistema operativo**: Ubuntu 20.04+
- **Java**: OpenJDK 8
- **Herramientas**: `wget`, `unzip`, `ufw` (firewall opcional).
- **Acceso**: usuario con permisos `sudo`.

---

## 1. Instalar Java

```bash
sudo apt update
sudo apt install -y openjdk-8-jre-headless unzip nano
java -version  # Debe salir algo como "1.8.0_xxx"
```

## 2. Descarga el paquete servidor desde CurseForge

```bash
mkdir -p /opt/sevtech
cd /opt/sevtech
wget -O sevtech.zip "https://mediafilez.forgecdn.net/files/3570/46/SevTech_Ages_Server_3.2.3.zip"
```

> Pagina oficial: https://www.curseforge.com/minecraft/modpacks/sevtech-ages/files/3569915/additional-files

## 4. instalar el servidor

```bash
unzip -q sevtech.zip
rm sevtech.zip
chmod +x *.sh
./Install.sh
```

## 5. Aceptar el EULA

```bash
echo "eula=true" > eula.txt
```

## 6. Configurar servicio systemd

Crea `/etc/systemd/system/sevtech.service`:

```bash
nano /etc/systemd/system/sevtech.service
```

```ini
[Unit]
Description = SevTech Ages Minecraft Server
After = network.target

[Service]
WorkingDirectory = /opt/sevtech
ExecStart = /opt/sevtech/ServerStart.sh
Restart = on-failure
TimeoutStartSec = 600
```

```bash
sudo systemctl daemon-reload
```

Iniciar:

```bash
service sevtech start
```

Detener:

```bash
service sevtech stop
```

Estado:

```bash
service sevtech status
```

Ver log:

```bash
sudo journalctl -u sevtech -f
```