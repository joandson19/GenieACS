# Criar diretório e dar permissões 
```
mkdir -p /opt/genieacs
mkdir -p /opt/genieacs/ext
chown genieacs:genieacs /opt/genieacs/ext
```
# Configurando arquivos 
```
cat <<EOF > /etc/systemd/system/genieacs-cwmp.service
[Unit]
Description=GenieACS CWMP
After=network.target
[Service]
User=genieacs
EnvironmentFile=/opt/genieacs/genieacs.env
ExecStart=/usr/bin/genieacs-cwmp
[Install]
WantedBy=multi-user.target
EOF
```
```
cat <<EOF > /etc/systemd/system/genieacs-nbi.service
[Unit]
Description=GenieACS NBI
After=network.target
[Service]
User=genieacs
EnvironmentFile=/opt/genieacs/genieacs.env
ExecStart=/usr/bin/genieacs-nbi
[Install]
WantedBy=multi-user.target
EOF
```
```
cat <<EOF > /etc/systemd/system/genieacs-fs.service
[Unit]
Description=GenieACS FS
After=network.target
[Service]
User=genieacs
EnvironmentFile=/opt/genieacs/genieacs.env
ExecStart=/usr/bin/genieacs-fs
[Install]
WantedBy=multi-user.target
EOF
```
```
cat <<EOF > /etc/systemd/system/genieacs-ui.service
[Unit]
Description=GenieACS UI
After=network.target
[Service]
User=genieacs
EnvironmentFile=/opt/genieacs/genieacs.env
ExecStart=/usr/bin/genieacs-ui
[Install]
WantedBy=multi-user.target
EOF
```
```
cat <<EOF > /etc/logrotate.d/genieacs
/var/log/genieacs/*.log /var/log/genieacs/*.yaml {
 daily
 rotate 30
 compress
 delaycompress
 dateext
}
EOF
```

# Crie o arquivo /opt/genieacs/ext/sistema e inclua os dados abaixo.
### Altere os dados das linhas 5,6,7,18 e 19 para os dados do seu SGP onde precisará criar o token e aplicação. 
```
#!/usr/bin/env node
let https = require( "https" );
const options = (dados)=>{
 return {
 hostname: 'demo.sgp.net.br',
 port: 443,
 path: '/api/ura/consultacliente',
 method: 'POST',
 headers: {
 'Content-Type': 'application/json',
 'Content-Length': Buffer.byteLength(JSON.stringify(dados)),
 },
 }
}
function pppoeLoginByMac( args, callback ) {
 const dados = {
 mac_dhcp: args[0],
 "app": "ura",
 "token": "1a551977-a459-4743-9a52-e9d1ca46a480"
 }
 const opts = options(dados)
 const req = https.request(opts, function(response) {
 let data = ''
 response.on('data', function( newData ){
 data += newData
 });
 response.on('end', function(){
 const parsedData = JSON.parse(data);
 let auth = {};
 if( parsedData.contratos && parsedData.contratos.length > 0 ) {
 auth = {
 username: parsedData.contratos[0].servico_login,
 password: parsedData.contratos[0].servico_senha,
 wifi_ssid: parsedData.contratos[0].servico_wifi_ssid,
 wifi_password:
parsedData.contratos[0].servico_wifi_password,
 wifi_channel:
parsedData.contratos[0].servico_wifi_channel,
 wifi_ssid_5:
parsedData.contratos[0].servico_wifi_ssid_5,
 wifi_password_5:
parsedData.contratos[0].servico_wifi_password_5,
 wifi_channel_5:
parsedData.contratos[0].servico_wifi_channel_5,
 };
 }
 callback( null, auth );
 });
 });
 req.write(JSON.stringify(dados))
 req.end()
}
exports.pppoeLoginByMac = pppoeLoginByMac;
```


# Acesse o genieacs pela interface web ex: IP:3000
## Vá até admin >> provision e altere a default apagando tudo e incluindo os dados abaixo.
```
const hourly = Date.now(3600000);
const every_ten_minutes = Date.now(600000);
const dayly = Date.now(24*3600000);
// Refresh basic parameters hourly
declare("InternetGatewayDevice.DeviceInfo.HardwareVersion", {path: hourly, value: hourly});
declare("InternetGatewayDevice.DeviceInfo.SoftwareVersion", {path: hourly, value: hourly});
declare("InternetGatewayDevice.WANDevice.*.WANConnectionDevice.*.WANPPPConnection.*.MACAddress",
{path: hourly, value: hourly});
declare("InternetGatewayDevice.WANDevice.*.WANConnectionDevice.*.WANPPPConnection.*.Username",
{path: hourly, value: every_ten_minutes});
declare("InternetGatewayDevice.WANDevice.*.WANConnectionDevice.*.WANPPPConnection.*.Uptime",
{path: hourly, value: every_ten_minutes});
declare("InternetGatewayDevice.WANDevice.*.WANConnectionDevice.*.WANPPPConnection.*.RemoteIPAddress", {path: hourly, value: every_ten_minutes});
declare("InternetGatewayDevice.WANDevice.*.WANConnectionDevice.*.WANPPPConnection.*.DefaultGateway",
{path: hourly, value: every_ten_minutes});
declare("InternetGatewayDevice.WANDevice.*.WANConnectionDevice.*.WANIPConnection.*.DefaultGateway",
{path: hourly, value: every_ten_minutes});
declare("InternetGatewayDevice.WANDevice.*.WANConnectionDevice.*.WANIPConnection.*.ExternalIPAddress",{path: hourly, value: hourly});
declare("InternetGatewayDevice.LANDevice.*.WLANConfiguration.*.SSID", {path: hourly, value:
hourly});
declare("InternetGatewayDevice.LANDevice.*.WLANConfiguration.*.Standard", {path: hourly,
value: hourly});
declare("InternetGatewayDevice.LANDevice.*.WLANConfiguration.*.Enabled", {path: hourly,
value: hourly});
declare("InternetGatewayDevice.LANDevice.*.WLANConfiguration.*.Standard", {path: hourly,
value: hourly});
declare("InternetGatewayDevice.LANDevice.*.WLANConfiguration.*.*.Standard", {path: hourly,
value: hourly});
declare("InternetGatewayDevice.LANDevice.*.Hosts.Host.*.HostName", {path: hourly, value:
hourly});
declare("InternetGatewayDevice.LANDevice.*.Hosts.Host.*.IPAddress", {path: hourly, value:
hourly});
declare("InternetGatewayDevice.LANDevice.*.Hosts.Host.*.MACAddress", {path: hourly, value:
hourly});
```

## Acesse também admin >> provision e altere a bootstrap apagando tudo e incluindo os dados abaixo.
```
/*
Realiza a configuração inicial para dispositivos da TP-Link.
Se o dispositivo estiver com a tag NOT_PROVISION o script não será executado.
Se o usuário do PPPOE estiver correto, o script não será executado.
*/
const now = Date.now();
const notProvision = declare("Tags.NOT_PROVISION", {value: 1});
if (notProvision.value !== undefined) {
 return;
}
const mac = declare(
"InternetGatewayDevice.WANDevice.1.WANEthernetInterfaceConfig.MACAddress", {value:
1}).value[0];
let auth = ext( "sistema", 'pppoeLoginByMac', mac );
if( !auth || !auth.username || !auth.password ) {
}
declare("InternetGatewayDevice.WANDevice.1.WANConnectionDevice.1.WANPPPConnection.*.*",
{path: now}); //refresh
declare("InternetGatewayDevice.LANDevice.*.WLANConfiguration.*.*", {path: now}); //refresh
const username =
declare("InternetGatewayDevice.WANDevice.1.WANConnectionDevice.1.WANPPPConnection.1.Username",
{value: 1} );
if( username && username.value && username.value.length > 1 && auth.username ==
username.value[0] ) {
	return;
}
// configura o PPPOE com o usuario encontrado
createPPPOEConnection( auth );
function createPPPOEConnection( auth ) {
declare("InternetGatewayDevice.WANDevice.1.WANConnectionDevice.1.WANIPConnection.*.Enable",
{path: now, value: now}, {value: false});
 declare("InternetGatewayDevice.WANDevice.1.WANConnectionDevice.1.WANPPPConnection.*",
null, {path: 1});
 declare("InternetGatewayDevice.WANDevice.1.WANConnectionDevice.1.WANPPPConnection.*.*",
{path: now}); //refresh
declare("InternetGatewayDevice.WANDevice.1.WANConnectionDevice.1.WANPPPConnection.*.ConnectionType",
{value: now}, {value: "IP_Routed"});
declare("InternetGatewayDevice.WANDevice.1.WANConnectionDevice.1.WANPPPConnection.*.Enable",
{value: now}, {value: true});
declare("InternetGatewayDevice.WANDevice.*.WANConnectionDevice.*.WANPPPConnection.*.Username",
{value: now}, {value: auth.username});
declare("InternetGatewayDevice.WANDevice.*.WANConnectionDevice.*.WANPPPConnection.*.Password",
{value: now}, {value: auth.password});
declare("InternetGatewayDevice.WANDevice.1.WANConnectionDevice.1.WANPPPConnection.1.PPPAuthenticationP{value: now}", {value: "AUTO_AUTH"});
 if(auth.wifi_ssid && auth.wifi_ssid.trim()){
 	declare("InternetGatewayDevice.LANDevice.*.WLANConfiguration.1.SSID",{value: now},
{value: auth.wifi_ssid});
 }
 if(auth.wifi_password && auth.wifi_password.trim().length >= 8){
declare("InternetGatewayDevice.LANDevice.*.WLANConfiguration.1.PreSharedKey.*.KeyPassphrase",{value:
now}, {value: auth.wifi_password});
 }
 if(auth.wifi_channel && auth.wifi_channel.trim().length){
 	declare("InternetGatewayDevice.LANDevice.1.WLANConfiguration.1.Channel",{value: now},
{value: auth.wifi_channel});
 }
}
```

# Agora vamos configurar os dados do Gerenciador no SGP
## Acesse Menu: SISTEMA >> GERENCIADOR DE CPE

![image](https://github.com/joandson19/GenieACS/assets/36518985/44e3192d-972f-4d0e-8b31-922295305192)

```
{
 "url": "http://XXX.XXX.XXX.XXX:7557",
 "token": "INDEFINIDO"
}
```

# Agora vamos configurar os dados do Roteador
## AACESSE O CONTRATO DO CLIENTE PARA ALTERAR AS SEGUINTES INFORMAÇÕES:

![image](https://github.com/joandson19/GenieACS/assets/36518985/fbc9d925-5d9b-47a5-be75-18d1026330e0)

# SALVAR E JÁ SERÁ POSSÍVEL OBSERVAR A NOVA ABA "GERENCIADOR CPE"
### QUE MOSTRARÁ AS INFORMAÇÕES DA SEGUINTE FORMA:

![image](https://github.com/joandson19/GenieACS/assets/36518985/21160a23-75fe-4df1-99f9-d6844d005098)
