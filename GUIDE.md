## Introduccion y Requisitos

Para poder armar este laboratorio se requiere previamente de un **visualizador** y conexion a internet. El virtualizador se Utilizara principalmente para el servidor y para virtualizar un entorno seguro donde atacar un OS que se encuentre conectado al servidor WAZUH.
Opcionalmente se puede integrar con un firewall a nivel de red como PFSENSE.

![image](https://github.com/user-attachments/assets/424f93ea-db53-47dc-b347-b29e03054b32)

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

### Logstash










