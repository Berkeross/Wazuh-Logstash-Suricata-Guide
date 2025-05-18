# <ims>Metasploit</ims>

Esta Guia para descargar el Framework esta realizada en ubuntu 24.04.<br>

Primero es necesario descargar el archivo `.deb` desde la pagina de [Metasploit](https://apt.metasploit.com), en esta sepuede ir a la parte final de la pagina donde se encuentra lso archivos de Metasploit, donde para Ubuntu es necesario el archivo `amd64`.
Este archivo se puede descargar copiando el enlace del `.deb` elegido y utilizando el siguiente comando.

```
wget https://apt.metasploit.com/pool/main/m/metasploit-framework/metasploit-framework_<LA_VERSIÓN_A_UTILIZAR>_amd64.deb
```

Con el archivo descargado se debe instalar utilizando el comando.

```
sudo dpkg -i metasploit-framework_<LA_VERSIÓN_A_UTILIZAR>_amd64.deb
```

Una vez finalizado metasploit quedaria instalado en el Sistema Operativo. Para ejecutar el primer comando se debe utilizar `msfconsole`, en este momento el framework te pregunta si "te gustaria configurar una nueva Database" donde se debe responder `yes`, una vez respondido se prosedera a iniciarse metasplit.
Con metasploit en funcionamiento los comandos para ingresar y salir serian respectivamente, `msfconsole` y `exit`.
