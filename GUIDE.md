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

### Winlogbeat

Dentro de todos los endpoints donde se instalo `Wazuh-Agent` se debe instalar la herramienta que esta intentando escucha logstash. Winlogbeat se ocupa de mandar los logs que wazuh-agent no detecta o no considere importantes, este funciona unicamente dentro de los sistemas windows pero existen otros `beats` que funcionan para multiples OS.

Primero se debe descargar el archivo `.zip` desde la pagina [oficial](https://www.elastic.co/es/downloads/beats/winlogbeat). Con el archivo descargado se debe descomprimir en la ruta `C:\Program Files\winlogbeat`.</br>
Dentro de la carpeta es necesario editar el archivo `winlogbeat.yml` y cambiar 


