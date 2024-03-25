# Instalação do GenieACS no Debian 12

Este guia fornece instruções passo a passo para instalar o GenieACS em um servidor Debian 12. O GenieACS é um ACS (Auto Configuration Server) de código aberto para gerenciar dispositivos de rede.

## Pré-requisitos

- Servidor Debian 12 instalado e configurado.
- Acesso de superusuário (sudo ou root).

## Passo 1: Atualizar o sistema

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

## Passo 2: Instalar dependências

```bash
sudo apt-get install gnupg curl rsyslog -y
```

## Passo 3: Configurar repositórios e instalar MongoDB

```bash
# Adicionar chaves para o MongoDB
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
echo "deb [signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg] http://repo.mongodb.org/apt/debian bullseye/mongodb-org/7.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Atualizar e instalar o MongoDB
sudo apt-get update
sudo apt-get install mongodb-org -y 
sudo systemctl enable mongod --now
```

## Passo 4: Instalar Node.js

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo bash -
sudo apt install nodejs -y
```

## Passo 5: Instalar GenieACS

```bash
sudo npm install -g genieacs
sudo useradd --system --no-create-home --user-group genieacs
sudo mkdir /opt/genieacs /opt/genieacs/ext /var/log/genieacs

# Configurar ambiente GenieACS
sudo tee /opt/genieacs/genieacs.env <<EOF
GENIEACS_CWMP_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-cwmp-access.log
GENIEACS_NBI_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-nbi-access.log
GENIEACS_FS_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-fs-access.log
GENIEACS_UI_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-ui-access.log
GENIEACS_DEBUG_FILE=/var/log/genieacs/genieacs-debug.yaml
NODE_OPTIONS=--enable-source-maps
GENIEACS_EXT_DIR=/opt/genieacs/ext
GENIEACS_UI_JWT_SECRET=secret
EOF

sudo chown genieacs. /opt/genieacs -R
sudo chmod 600 /opt/genieacs/genieacs.env
sudo chown genieacs. /var/log/genieacs

# Configurar serviços GenieACS
for service in cwmp nbi fs ui; do
    sudo tee "/etc/systemd/system/genieacs-$service.service" <<EOF
[Unit]
Description=GenieACS $service
After=network.target

[Service]
User=genieacs
EnvironmentFile=/opt/genieacs/genieacs.env
ExecStart=/usr/bin/genieacs-$service

[Install]
WantedBy=default.target
EOF
    sudo systemctl daemon-reload
    sudo systemctl enable "genieacs-$service"
done

# Configurar logrotate
sudo tee /etc/logrotate.d/genieacs <<EOF
/var/log/genieacs/*.log /var/log/genieacs/*.yaml {
    daily
    rotate 30
    compress
    delaycompress
    dateext
}
EOF

# Iniciar serviços GenieACS
for service in cwmp nbi fs ui; do
    sudo systemctl start "genieacs-$service"
done

# Exibir status dos serviços GenieACS
sudo systemctl status genieacs-cwmp genieacs-nbi genieacs-fs genieacs-ui --no-pager
```
