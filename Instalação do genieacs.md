# Instalação do GenieACS no Debian 12

Este guia fornece instruções passo a passo para instalar o GenieACS em um servidor Debian 12. O GenieACS é um ACS (Auto Configuration Server) de código aberto para gerenciar dispositivos de rede.

## Pré-requisitos

- Servidor Debian 12 instalado e configurado.
- Acesso de superusuário (sudo ou root).

## Passo 1: Atualizar o sistema

```bash
apt-get update
apt-get upgrade -y
```

## Passo 2: Instalar dependências

```bash
apt-get install gnupg curl rsyslog -y
```

## Passo 3: Configurar repositórios e instalar MongoDB

```bash
# Adicionar chaves para o MongoDB
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
echo "deb [signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg] http://repo.mongodb.org/apt/debian bullseye/mongodb-org/7.0 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Atualizar e instalar o MongoDB
apt-get update
apt-get install mongodb-org -y 
systemctl enable mongod --now
```

## Passo 4: Instalar Node.js

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo bash -
apt install nodejs -y
```

## Passo 5: Instalar GenieACS

```bash
npm install -g genieacs
useradd --system --no-create-home --user-group genieacs
mkdir /opt/genieacs /opt/genieacs/ext /var/log/genieacs

# Configurar ambiente GenieACS
tee /opt/genieacs/genieacs.env <<EOF
GENIEACS_CWMP_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-cwmp-access.log
GENIEACS_NBI_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-nbi-access.log
GENIEACS_FS_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-fs-access.log
GENIEACS_UI_ACCESS_LOG_FILE=/var/log/genieacs/genieacs-ui-access.log
GENIEACS_DEBUG_FILE=/var/log/genieacs/genieacs-debug.yaml
NODE_OPTIONS=--enable-source-maps
GENIEACS_EXT_DIR=/opt/genieacs/ext
GENIEACS_UI_JWT_SECRET=secret
EOF

chown genieacs. /opt/genieacs -R
chmod 600 /opt/genieacs/genieacs.env
chown genieacs. /var/log/genieacs

# Configurar serviços GenieACS
tee "/etc/systemd/system/genieacs-$service.service" <<EOF
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
systemctl daemon-reload
systemctl enable "genieacs-$service"
done

# Configurar logrotate
tee /etc/logrotate.d/genieacs <<EOF
/var/log/genieacs/*.log /var/log/genieacs/*.yaml {
    daily
    rotate 30
    compress
    delaycompress
    dateext
}
EOF

# Iniciar serviços GenieACS
systemctl start "genieacs-$service"
done

# Exibir status dos serviços GenieACS
systemctl status genieacs-cwmp genieacs-nbi genieacs-fs genieacs-ui --no-pager
```
