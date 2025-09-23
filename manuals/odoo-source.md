
# Manual de Instalación: PostgreSQL, Odoo y Nginx

## Requisitos del Sistema

- Sistema operativo: Ubuntu 20.04 / 22.04 (recomendado)
- Acceso root o usuario con privilegios sudo
- Conexión a Internet
- Programas necesarios: `nano` o tu editor preferido

---

## 1. Instalación de PostgreSQL

### 1.1 Actualizar repositorios
```bash
sudo apt update
```

### 1.2 Instalar PostgreSQL
```bash
sudo apt install postgresql -y
```

### 1.3 Crear usuario para Odoo
```bash
sudo -u postgres createuser -s -P odoo
```
> **Nota:** Puedes cambiar `odoo` por el nombre del usuario que usará la base de datos.

---

## 2. Conexión local (opcional)

### 1.2 Configurar variables de entorno, debes volver a conectarte para que surta efecto
```bash
sudo nano /etc/environment
```
```ini
PGHOST=127.0.0.1
PGPORT=5432
PGUSER=odoo
PGDATABASE=postgres
```
> **Nota:** Puedes cambiar `postgres` por el nombre de la base de datos a utilizar.

### 1.2 Configurar archivo de conexión para la contraseña
```bash
printf '127.0.0.1:5432:*:odoo:odoo\n' > ~/.pgpass
chmod 600 ~/.pgpass
```
> **Nota:** La estructura de .pgpass es `host:port:database:username:password`.

## 2. Instalación de dependencias para Odoo

```bash
sudo apt install python3-pip libldap2-dev libpq-dev libsasl2-dev  git python3-venv wkhtmltopdf -y
```

---

## 3. Instalación de Odoo

### 3.2 Crear estructura de directorios
```bash
sudo mkdir -p /opt/odoo
sudo mkdir -p /opt/odoo/odoo
sudo mkdir -p /opt/odoo/custom_addons
sudo mkdir -p /opt/odoo/venv
sudo mkdir -p /etc/odoo
sudo mkdir -p /var/log/odoo
sudo mkdir -p /var/lib/odoo
sudo ln -s /var/log/odoo /opt/odoo/log
sudo ln -s /etc/odoo /opt/odoo/conf
```

### 3.3 Descargar el código de Odoo
```bash
sudo git clone https://www.github.com/odoo/odoo --depth 1 --branch 16.0 --single-branch /opt/odoo/odoo
```
#### Enterprise (si aplica)
```bash
sudo git clone https://www.github.com/odoo/enterprise --depth 1 --branch 16.0 --single-branch /opt/odoo/custom_addons/enterprise
```

### 3.4 Crear y activar entorno virtual

```bash
python3 -m venv /opt/odoo/venv
source /opt/odoo/venv/bin/activate
```

#### En caso se tenga una version de python diferente a la 3.12
```bash
sudo add-apt-repository -y ppa:deadsnakes/ppa
sudo apt update
sudo apt install -y python3.12 python3.12-venv python3.12-dev
python3.12 -m venv /opt/odoo/venv
source /opt/odoo/venv/bin/activate
```

### 3.5 Instalar dependencias de Python

```bash
pip install wheel
pip install -r /opt/odoo/odoo/requirements.txt
```

## 4. Configurar Odoo

### 4.1 Crear usuario del sistema
```bash
sudo adduser --system --home=/opt/odoo --group odoo
```
> **Nota:** En caso se necesite ingresar como el nuevo usuario `sudo -u odoo -s`.

#### Código solo lectura para odoo
```bash
sudo chown -R root:odoo /opt/odoo/odoo /opt/odoo/custom_addons /etc/odoo
sudo chmod -R 750 /opt/odoo/odoo /opt/odoo/custom_addons /etc/odoo
```
#### Venv, datos y logs escribibles por odoo
```bash
sudo chown -R odoo:odoo /opt/odoo/venv /var/lib/odoo /var/log/odoo
```

### 4.2 Crear archivo de configuración

```bash
sudo touch /etc/odoo/odoo.conf
sudo chown -R root:odoo /etc/odoo
sudo chmod -R 750 /etc/odoo
```

### 4.3 Editar `/etc/odoo/odoo.conf`

```bash
sudo nano /etc/odoo/odoo.conf
```
```ini
[options]
list_db = False
db_host = 127.0.0.1
db_port = 5432
db_user = odoo
db_password = odoo
db_name = 
addons_path = /opt/odoo/odoo/addons,/opt/odoo/custom_addons/enterprise
data_dir = /opt/odoo/.local/share/Odoo
without_demo = True
;logfile = Se define en el servicio para mayor flexibilidad
```

---

## 5. Crear servicio systemd para Odoo

```bash
sudo nano /etc/systemd/system/odoo.service
```
#### Contenido:
```ini
[Unit]
Description=Odoo Open Source ERP and CRM
After=network.target

[Service]
Type=simple
User=odoo
Group=odoo
ExecStart=/opt/odoo/venv/bin/python3 /opt/odoo/odoo/odoo-bin -c /etc/odoo/odoo.conf --logfile /var/log/odoo/odoo-server.log
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable --now odoo
```