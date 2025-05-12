# Wazuh-Logstash-Suricata-Guide
Guide to building a labSIEM system using Wazuh, Logstash implementing Winlogbeat and Suricata.

### Descripción
Este repositorio es una guía Para implementar un Servidor WAZUH-SIEM(basado en Opensearch) el cual esta formado por WAZUH-INDEXER como motor de busqueda, WAZUH-MANAGER quien analiza los datos para generar alertas y WAZUH-DASHBOARD que funciona como interfaz web para visualizar los datos indexados y las alertas. Asi mismo cada ENDPOINT que se quiera monitorizar debe contar con un servicio WAZUH-AGENT.</br>

Ademas del Servidor WAZUH, va a contar con 2 herramientas mas:</br>
LOGSTASH, este va a estar dentro del servidor y va a funcionar como puente para conectar los logs que va a recibir de un servicio Beat (ejem: Winlogbeat o Filebeat) dentro de los ENDPOINTS.</br>
SURICATA, este va a estar conectado a los ENDPOINTS junto al Beat elegido. Suricata funciona como un NIDS que se encarga de enviar logs a WAZUH-SERVER.</br>

Estas herramientas estan elegidas para suplir algunas limitaciones que cuenta WAZUH como servicio unico, Estas implementaciones estan explicadas en la siguiente publicacion de [LinkedIn](https://www.linkedin.com/posts/alejo-bergero-688a9531a_ciberseguridad-soc-elasticsiem-activity-7315942584340611073-gY0v?utm_source=share&utm_medium=member_desktop&rcm=ACoAAFD2VogBamGpXj3dO3QI9REDzrwa2Gi2pmQ)
