# <ins>Atomic Red Team</ins>

Atomic Red Team es una herramienta un poco compleja de instalar y ejecutar. Primero es necesario desactivar Windows Defender, para esto hay que dirigirse a [<ins> Menú del buscador → Settings → Update & Security → Windows Defender → Virus & Threat Protection → Virus & Threat Protection Settings → Manage Settings </ins>]. En esta pestaña desactiva todas las opciones. Una vez desactivado, el último paso es ejecutar el comando:

	Set-MpPreference -DisableRealtimeMonitoring $true

Después, para que PowerShell pueda instalar las dependencias, necesita el módulo ".yaml", el cual se instala con: 

	Install-Module powershell-yaml -Force -Scope CurrentUser
	Import-Module powershell-yaml

Con el antivirus completamente desactivado, se deben descargar dos carpetas. La primera es [atomic-red-team-master](https://github.com/redcanaryco/atomic-red-team.git) y la segunda es [invoke-atomicredteam-master](https://github.com/redcanaryco/invoke-atomicredteam.git), ambas se encuentran en GitHub descargando el archivo .zip.<br/>
Antes de extraerlos se debe crear una carpeta con nombre "AtomicRedTeam" en la carpeta "C:\" (disco local). Estos archivos descargados se deben descomprimir en la carpeta AtomicRedTeam, allí debería quedar:
	
 	C:\AtomicRedTeam\
	    └── atomic-red-team-master\
	    ├    └── atomic-red-team-master\
	    ├    	├── .devcontainer\
	    ├    	├── atomics\
        ├		└── ...\	
	    └── invoke-atomicredteam-master\
	         	└── invoke-atomicredteam-master\
	         		├── .git\
	    			├── docker\
	         		└── ...
 
Como se puede ver, ambos archivos al extraerlos crean una carpeta del mismo nombre dentro de su propia carpeta con el contenido necesario. Para evitar problemas con los comandos que se deben ejecutar, se debe quitar la carpeta intermedia con nombre repetido para tener una ruta más corta, además de mover la carpeta "\atomics" a la ruta "C:\AtomicRedTeam\". Esto debería quedar así:

  	C:\AtomicRedTeam\
	    ├── atomic-red-team-master\
	    ├ 	 └── .devcontainer\
	    ├	 └── ...
	    ├── atomics\
	    ├	 └── indexes\
	    ├	 └── ...
	    └── invoke-atomicredteam-master\
	         └── .git\
	         └── ...

Una vez las carpetas estén colocadas correctamente, se debe instalar. En este repositorio se explicará el paso a paso de esta versión. En caso de encontrarte con un error, no dudes en consultar con la [Wiki](https://github.com/redcanaryco/invoke-atomicredteam/wiki) de Atomic Red Team. Primero se debe ingresar a PowerShell como administrador y ejecutar:

	powershell -exec bypass

Luego de que se recargue PowerShell se debe ejecutar:

	Install-Module -Name invoke-atomicredteam,powershell-yaml -Scope CurrentUser

Finalizada la instalación del módulo, se debe verificar que se instaló correctamente con el código:

	Get-Module

Si al ejecutarlo aparece "Invoke-AtomicRedTeam" en la lista de módulos quiere decir que el programa se instaló satisfactoriamente. Para poder simular un ataque es necesario ingresar un código. Este código es el que se usa para categorizar ataques en la organización ["MITRE ATT&CK"](https://attack.mitre.org/matrices/enterprise/).<br/>

> [!NOTE]  
> El link que está atado a MITRE ATT&CK lleva a un cuadro con todos los ataques categorizados. Si a estos les pasas el mouse por encima verás un código como "T1548" o "T1016". Estos son los códigos utilizados para lanzar ataques con Atomic Red Team.
<br/>

Para iniciar un ataque se debe utilizar el comando `Invoke-AtomicTest` seguido de un código de ataque. También se le puede agregar tags al final de la línea como por ejemplo `-ShowDetailsBrief`, que muestra una lista de funciones del ataque en concreto, o `-CheckPrereqs`, el cual muestra una lista de requisitos para que se pueda ejecutar el ataque (mayormente siempre se cumplen los requisitos, ya que son programas que vienen incluidos con Windows).
