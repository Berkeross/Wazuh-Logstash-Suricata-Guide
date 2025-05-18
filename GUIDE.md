## Introduccion y Requisitos

Para poder armar este laboratorio se requiere previamente de un **visualizador** y conexion a internet. El virtualizador se Utilizara principalmente para el servidor y para virtualizar un entorno seguro donde atacar un OS que se encuentre conectado al servidor WAZUH.
Opcionalmente se puede integrar con un firewall a nivel de red como PFSENSE.

![image](/Capturas/map.png)

Los requisitos de para los dos sistemas virtualizados son:
- SERVIDOR: Ubuntu Server 24.04, 50gb (min)
- OS ATACADO: Linux Mint 22.1 (xfce), 15gb (min)

# Guía

### WAZUH-SERVER
El Sistema SIEM que se utiliza para este laboratorio se puede instalar de dos formas. La primera es la forma manual donde se instalan todas las dependencias de forma independiente, y la segunda que el la mas sencilla y la que se utiliza en esta guia que es utilizando el **quickstart** de wazuh.

Con la maquina virtual ubuntu server ejecutada, se puede iniciar con la instalacion de wazuh-server, para esto, primero se debe actualizar APT y despues:

```shell
    curl -sO https://packages.wazuh.com/4.11/wazuh-install.sh
    sudo bash ./wazuh-install.sh -a
```
Con este comando se empieza con la instalacion de WAZUH-SERVER. Una vez finalizado es necesario ingresar en el dashboard para verificar que se instalo correctamente. se ingreda en `https://<TU-IP>:443` con las siguentes credenciales:</br>
User: `admin`</br>
Password: `Esta se genera automaticamente al finalizar la instalacion de wazuh`

Una vez ingresado en la plataforma y visualizar que no se encuentran errores, se puede cambiar la contraseña para facilitar el acceso utilizando:

```shell
sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh -u admin -p <nueva_contraseña>
```

Teniendo el servidor en funcionamiento es necesarios ingresar los `agentes`, estos se pueden instalar mediante una linea de codigo que genera el propio sistema en `Menu => Agents Management => Sumary => Deploy new Agent`.

Con estos se puede empezar a visualizar los logs y alertas que generon los agentes dentro del dashbard.

![image](/Capturas/wazuhmap.png)

### Logstash

Logstash se debe instalar dentro del servidor, para esto es necesario instalar los paquetes necesarios.

```shell
sudo apt-get install apt-transport-https
```

Con los paquetes instalados, se debe descargar la KEY para integrarla a APT.

```shell
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg
```

Luego se debe ingresar la KEY a APT.

```shell
echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-9.x.list
```

Por ultimo instalar Logstash.

```shell
sudo apt install logstash
```

Logstash Indexa logs para integrarlos en el servidor y poder visualizarlos, estos logs son enviados de algunas herramientas como los `Beats` que se encuentan en los endpoints.

Para que Logstash pueda encontrar los logs que se le manda se debe crear un archivo de configuracion donde indique el puerto que tiene que escuchar y el archivo qeu tiene que buscar.</br>
Este archivo `.conf` se debe crear con el comando:

```shell
sudo nano /etc/logstash/conf.d/winlogbeat.conf
```
Una vez dentro del archivo se debe colocar:

```shell
input { 
  beats {
    port => 5044
  }
}

output {
  opensearch {
    hosts => ["https://localhost:9200"]
    user => "admin"
    password => "<nueva_contraseña>"
    index => "winlogbeat-%{+YYYY.MM.dd}"
    ssl => true
    ssl_certificate_verification => false
  }
}
```

con el comando `ssl => true` y `ssl_certificate_verification => false` se permite que el log ingrese por un canal seguro pero no necesite verificacion, con esto se puede ahorrar un paso.

 > [!IMPORTANT]
> odas estas especificaciones se4 hacen dentro de un entorno de prueba para practicar dentro de un sistema SIEM, de ser posible cuanto mas seguridad tenga el servidor para todos los archivos que integre mejor.

Logstash tamien nececita los plugins con los cuales detecta los `Outputs`, y como WAZUH esta basado en `OpenSearch` se debe instalar el plugin con.

```shell
sudo /usr/share/logstash/bin/logstash-plugin install logstash-output-opensearch
```

Con todo el sistema completamente instalado se debe habilitar e iniciar.

```
sudo systemctl enable logstash
sudo systemctl Start logstash
```

Para verificar que el sistema esta en completo funcionamiento pasados 30 segundos utilizar el comando `sudo systemctl status logstash` 

### Winlogbeat y Filebeat

Dentro de todos los endpoints donde se instalo `Wazuh-Agent` se debe instalar la herramienta que esta intentando escucha logstash. Winlogbeat se ocupa de mandar los logs que wazuh-agent no detecta o no considere importantes, este funciona unicamente dentro de los sistemas windows pero existen otros `beats` que funcionan para multiples OS.

Primero se debe descargar el archivo `.zip` desde la pagina oficial de [Winlogbeat](https://www.elastic.co/es/downloads/beats/winlogbeat) o [Filebeat](https://www.elastic.co/es/downloads/beats/filebeat). La diferencia entre estos se encunetra en que **Winlogbeat** captura logs unicamente de windows y **Filebeat** permite capturar logs desde cualquier Sistema operativo. Para el ejemplo se va a explicar la instalacion de Winlogbeat, aun asi la instalación funciona casi igual para ambos.</br>
Con el archivo descargado se debe descomprimir en la ruta `C:\Program Files\winlogbeat`.</br>
Dentro de la carpeta es necesario editar el archivo [`winlogbeat.yml`](/Conf-File/winlogbeat.yml) y cambiar las siguientes opciones dentro de la seccion **Outputs**.

```
# ================================== Outputs ===================================

# Configure what output to use when sending the data collected by the beat.

# ---------------------------- Elasticsearch Output ----------------------------
#output.elasticsearch:   <= colocar un "#" en todas las opciones del elastic output 
  # Array of hosts to connect to.
  #hosts: ["localhost:9200"]

  # Protocol - either `http` (default) or `https`.
  #protocol: "https"

  # Authentication credentials - either API key or username/password.
  #api_key: "id:api_key"
  #username: "elastic"
  #password: "changeme"

  # Pipeline to route events to security, sysmon, or powershell pipelines.
  #pipeline: "winlogbeat-%{[agent.version]}-routing"

# ------------------------------ Logstash Output -------------------------------
output.logstash:   <= quitar el "#" las siguientes dos opciones
  # The Logstash hosts
  hosts: ["<IP-WAZUHSERVER>:5044"]  <=

  # Optional SSL. By default is off.
  # List of root certificates for HTTPS server verifications
  #ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]

  # Certificate for SSL client authentication
  #ssl.certificate: "/etc/pki/client/cert.pem"

  # Client Certificate Key
  #ssl.key: "/etc/pki/client/cert.key"
```

Una vez terminada la configuracion, se debe guardar en el mismo formato "`.conf`".</br>
para instalar winlogbeat se debe ejecutar el archivo `install-service-winlogbeat.ps1`, este arechivo se debe ejecutrar dentro de powershell en modo **Administrador**, dentro de Powershell deberia verse asi.

```shell
C:\Program Files\winlogbeat>.\install-service-winlogbeat.ps1
```

> [!NOTE]
> Si el comando falla puede ser posible que se trate de los permisos del Firewall, se puede solucionar utilizando el comando `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` y aplicarlo utilizando el comando `Acept to all`.

con el servicio instalado se activa utilizando el comando `Start-Service winlogbeat`, si al utilizar este comando se genera un error se puede verificar la configuración utilizando `.\winlogbeat.exe test config`.

Pasado un pequeño lapso de timepo se deberia poder ver los logs en "`Menu => Indexer Management => Index Management`".

Para poder agrupar estos logs es necesario dirigirse a “`Menu => Dashboard-manager => Index patterns`”, desde este lugar se puede
crear un grupo de indexadores en el boton `Create index pattern`.

### Suricata

Wazuh permite integración con Suricata, un sistema de detección de intrusiones basado en la red (NIDS) para mejorar la detección de amenazas mediante la supervisión y el análisis del tráfico de la red. Suricata puede proporcionar información adicional sobre la seguridad de su red con sus capacidades de inspección del tráfico de red.</br>
Suricata se puede integrar Tanto en OS Linux como en Windows, Para esta guia solo se va a guiar para la instalación de linux, aun asi la guia de instalacion en Windows se va a dejar [aqui](https://letsdefend.io/blog/how-to-install-and-configure-suricata-on-windows).

Primero se agregrega el reposityorio a APT y se instala Suricata.

```shell
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt-get update
sudo apt-get install suricata -y
```

Luego se descargan las `rules` con las que funcione Suricata

```shell
cd /tmp/ && curl -LO https://rules.emergingthreats.net/open/suricata-6.0.8/emerging.rules.tar.gz
sudo tar -xvzf emerging.rules.tar.gz && sudo mkdir /etc/suricata/rules && sudo mv rules/*.rules /etc/suricata/rules/
sudo chmod 640 /etc/suricata/rules/*.rules
```

Antes de iniciar el sistema es necesario configurar el archivo `.yaml` en `/etc/suricata/suricata.yaml`, este se edita utilizando el comando nano y se deben cambiar las siguientes secciones.

```shell
HOME_NET: "<LINUXorWIN_IP>"
EXTERNAL_NET: "any"

-----------------------------------------------------

stats:
enabled: yes

---------------------------------------------------
 
af-packet:
  - interface: <enp0s3 en linux - Ethernet en Win>

----------------------------------------------------

default-rule-path: <`/etc/suricata/rules` en linux o `C:\\Program File\\Suricata\\Rules\\` en Win>
rule-files:
- "*.rules"
```

Una vez configurado dentro del **OS linux** se debe Reiniciar Suricata utilizando `sudo systemctl restart suricata`. Una vez reiniciado y este no genere errores es momento de configurar el archivo `ossec.conf` para que Wazuh-Agent, Para esto se ejecuta el comando `sudo nano /var/ossec/etc/ossec.conf` y agregando las siguientes lineas.

```shell
<ossec_config>
  <localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
  </localfile>
</ossec_config>
```

Con la configuración finalizada, se debe reiniciar utilizando `sudo systemctl restart wazuh-agent`. Con esto Wazuh-Server deveria estar recibiendo los logs de suricata.

### Ataque




