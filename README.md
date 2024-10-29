# Guía de Instalación de Headwind MDM
# **Introducción

Headwind MDM es una solución de gestión de dispositivos móviles (MDM) de código abierto diseñada para ayudar a las organizaciones a gestionar, supervisar y asegurar sus dispositivos móviles de manera eficiente. Esta guía te proporcionará los pasos necesarios para instalar Headwind MDM en tu servidor, permitiéndote configurar un entorno de gestión que simplifique el control de dispositivos, la implementación de aplicaciones y el cumplimiento de políticas de seguridad.

El objetivo de esta guía es proporcionarte instrucciones claras y concisas para que puedas instalar y configurar Headwind MDM de manera efectiva. Al finalizar, deberías ser capaz de acceder a la interfaz web de Headwind MDM y comenzar a gestionar tus dispositivos móviles de forma centralizada.

# **Requisitos previos**

- Servidor Ubuntu 22.04 LTS (para pruebas, se recomienda una máquina virtual con 4 Gb de RAM, 2xCPU, 20 Gb SSD) [**Requisitos de hardware para producción**](https://h-mdm.com/hardware-requirements/)
- Dirección IP pública (IPv4)
- Acceso SSH
- Nombre de dominio vinculado a la dirección pública (usamos **build.h-mdm.com**
    
    )
    
- [**Puertos**](https://qa.h-mdm.com/16781/) abiertos
- Acceso directo a Internet (al menos durante la instalación)

La configuración debe realizarse como **root** .

# **De un vistazo**

En este video, configuramos un servidor desde el principio (creando una VM).

[https://youtu.be/QZXkaljKcy8](https://youtu.be/QZXkaljKcy8)

# **1. Instale el software requerido**

```
apt update
```

```
apt install -y aapt tomcat9 postgresql vim certbot unzip net-tools
```

Aviso: la versión de Tomcat 9 instalada por apt en Ubuntu 20.04 (9.0.31) tiene un error relacionado con HTTPS y Headwind MDM no funciona correctamente con HTTPS. El instalador de Headwind MDM actualizará automáticamente Tomcat a la versión mínima requerida; no omita este paso.

# **2. Configurar la base de datos**

```
su - postgres
```

```
psql
```

```
CREATE USER hmdm WITH PASSWORD 'dendroidecode_BD0';
```

```
CREATE DATABASE hmdm_dendroidecode WITH OWNER=hmdm;
```

```
\q
```

```
exit
```

*Aviso: es posible que desee utilizar su propia contraseña para mayor seguridad. Recuérdelo y utilícelo en el paso 4 cuando ejecute un script de instalación de Headwind MDM.*

# **3. Descargue y descomprima el instalador binario.**

*Aviso: obtenga la URL de la última versión del instalador web en la página [**"Descargar"**](https://h-mdm.com/download) .*

```
wget https://h-mdm.com/files/hmdm-5.24-install-ubuntu.zip
```

```
unzip hmdm-5.24-install-ubuntu.zip
```

```
cd hmdm-install/
```

# **Alternativa: build Headwind MDM**

```
git clone https://github.com/h-mdm/hmdm-server.git
cd hmdm-server/
apt install -y maven
cp server/build.properties.example server/build.properties
mvn install
```

# **4. Instale MDM en contra del viento**

Para iniciar la instalación, ejecute el comando de la consola:

```
./hmdm_install.sh
```

Recomendamos confirmar las respuestas sugeridas a las preguntas del instalador (instalar el software requerido, actualizar Tomcat, etc.).

![https://h-mdm.com/wp-content/uploads/hmdm_install_new_1.jpg](https://h-mdm.com/wp-content/uploads/hmdm_install_new_1.jpg)

Importante: en Tomcat 9, debe utilizar un subdirectorio del “Tomcat sandbox” (/var/lib/tomcat9) para almacenar archivos, porque Tomcat no tiene permiso para escribir archivos fuera del sandbox. Los scripts y otros archivos no relacionados con Tomcat se colocan en /opt/hmdm de forma predeterminada.

![https://h-mdm.com/wp-content/uploads/hmdm_install_new_2.jpg](https://h-mdm.com/wp-content/uploads/hmdm_install_new_2.jpg)

![https://h-mdm.com/wp-content/uploads/hmdm_install_new_3.jpg](https://h-mdm.com/wp-content/uploads/hmdm_install_new_3.jpg)

Después de este paso, ya puede comprobar que el panel web de Headwind MDM se puede abrir abriendo **http://build.h-mdm.com:8080** en un navegador web.

Si recibe el error "Error al implementar el archivo WAR", simplemente reinicie el script del instalador.

Además, el instalador configura HTTPS a través de LetsEncrypt (un motor de certificados HTTPS gratuito), configura la renovación periódica de certificados y descarga los archivos APK necesarios. Recomendamos responder "SÍ" a todos los pasos del instalador.

![https://h-mdm.com/wp-content/uploads/hmdm_install_new_4.jpg](https://h-mdm.com/wp-content/uploads/hmdm_install_new_4.jpg)

LetsEncrypt le pedirá que ingrese su correo electrónico. Puede compartir su correo electrónico de forma segura porque LetsEncrypt nunca envía spam. Después de aceptar los términos y condiciones (obligatorio), deshabilite el envío de correo electrónico respondiendo "NO".

![https://h-mdm.com/wp-content/uploads/hmdm_install_new_5.jpg](https://h-mdm.com/wp-content/uploads/hmdm_install_new_5.jpg)

# **5. Validar la instalación**

Asegúrese de que el panel de administrador esté funcionando. **https://build.h-mdm.com** debería abrir el panel web.

El nombre de usuario y la contraseña predeterminados son admin:admin (se le pedirá que cambie la contraseña; elija una **segura** ).

Si tuvo algún problema al instalar Headwind MDM, debe consultar los registros de Tomcat para diagnosticar el problema. Tomcat 9 escribe sus registros en el registro del sistema de Linux:

```
journalctl -u tomcat9.service
```

**¡Haga una copia de seguridad de su archivo de configuración XML!**

Hay un error en Tomcat 9 que provoca la eliminación ocasional de la configuración XML después de actualizar el archivo WAR. Para evitar fallos del servidor después de la actualización, recomendamos encarecidamente realizar una copia de seguridad.

```
cp /var/lib/tomcat9/conf/Catalina/localhost/ROOT.xml /var/lib/tomcat9/conf/Catalina/localhost/ROOT.xml~
```

# **6. Registrar dispositivos**

Abra la sección **Dispositivos** y haga clic en el ícono del código QR.

![https://h-mdm.com/wp-content/uploads/check_qr_code.jpg](https://h-mdm.com/wp-content/uploads/check_qr_code.jpg)

Si ve el código QR, **la instalación de Headwind MDM se completó** , ¡felicidades!

# 7. Configuración SSL

En este paso, debe ajustar la configuración de Tomcat para usar el archivo pem generado como certificado ssl del dominio.

Copiar los archivos pem, crt, key, etc. En el subdirectorio accesible para Tomcat (aviso: Tomcat funciona en un "sandbox" y no tiene acceso a archivos fuera de /var/lib/tomcat9 incluso si otorga acceso al usuario de Tomcat).

```bash
mkdir /var/lib/tomcat9/ssl
```

```bash
cp /home/administrador/dendroidecode /var/lib/tomcat9/ssl/
```

```bash
chown -R tomcat:tomcat /var/lib/tomcat9/ssl
```

Editar la configuración del servidor Tomcat /var/lib/tomcat9/conf/server.xml

Encuentra el bloque comentado:

```bash
<!--

    <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"

               maxThreads="150" SSLEnabled="true">

        <SSLHostConfig>

            <Certificate certificateKeystoreFile="conf/localhost-rsa.jks"

                         type="RSA" />

        </SSLHostConfig>

    </Connector>

    -->
```

Descomentarlo y actualizarlo, para que se vea así:

```bash
<Connector port="8443" protocol="org.apache.coyote.http11.Http11AprProtocol"
               maxThreads="150" SSLEnabled="true" >
        <UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
        <SSLHostConfig>
            <Certificate certificateKeyFile="/var/lib/tomcat9/ssl/dendroidecode/dendroidecode.key"
                         certificateFile="/var/lib/tomcat9/ssl/dendroidecode/dendroidecode.pem"
                         type="RSA" />
        </SSLHostConfig>
    </Connector>
```

Reiniciar Tomcat con el comando

```bash
service tomcat9 restart
```

Puedes comprobar que todo está bien ejecutando el comando:

```bash
journalctl -f -u tomcat9.service
```

Editar la configuración del sistema Headwind MDM

```bash
nano /var/lib/tomcat9/conf/Catalina/localhost/ROOT.xml
```

Deberá tener la siguiente configuración, cambiando el puerto a utilizar el ssl globalmente.

```bash
<?xml version="1.0" encoding="UTF-8"?>
<Context>
    <!-- database configurations -->
    <Parameter name="JDBC.driver"   value="org.postgresql.Driver"/>
    <Parameter name="JDBC.url"      value="jdbc:postgresql://localhost:5432/hmdm_dendroidecode"/>
    <Parameter name="JDBC.username" value="hmdm"/>
    <Parameter name="JDBC.password" value="dendroidecode_BD0"/>

    <!-- This directory is used to as a base directory to store app data -->
    <Parameter name="base.directory" value="/var/lib/tomcat9/work"/>

    <!-- This directory is used to store uploaded app files, must be accessible for tomcat user -->
    <Parameter name="files.directory" value="/var/lib/tomcat9/work/files"/>

    <!-- URL used to open Headwind MDM control panel -->
    <Parameter name="base.url" value="https://mdm.dendroidecode.com"/>

    <!-- private / shared; shared can be used only in Enterprise solution -->
    <Parameter name="usage.scenario" value="private" />

    <!-- If set to 1, the device configuration request must be signed by a shared secret (setup in hash.secret and in the Android app)
         0 or empty value does not require request signature which is less secure -->
    <Parameter name="secure.enrollment" value="0"/>
    <!-- A shared secret between mobile app and control panel.
         Don't change this unless you know what you're doing -->
    <Parameter name="hash.secret" value="changeme-C3z9vi54"/>

    <!-- This directory is used to store files by plugins, must be accessible for tomcat user -->
    <Parameter name="plugins.files.directory" value="/var/lib/tomcat9/work/plugins"/>

    <!-- Configuration for logging plugin, do not change this -->
    <Parameter name="plugin.devicelog.persistence.config.class" value="com.hmdm.plugins.devicelog.persistence.postgres.DeviceLogPostgresPersistenceConfiguration"/>

    <!-- Don't change this -->
    <Parameter name="role.orgadmin.id" value="2"/>

    <!-- Swagger Docs UI location -->
    <Parameter name="swagger.host" value="mdm.dendroidecode.com"/>
    <Parameter name="swagger.base.path" value="/rest"/>

    <Parameter name="initialization.completion.signal.file" value="/var/lib/tomcat9/work/hmdm_install_flag"/>

    <Parameter name="log4j.config" value="file:///var/lib/tomcat9/work/log4j-hmdm.xml"/>

    <Parameter name="aapt.command" value="aapt"/>

    <!-- MQTT notification service parameters -->
    <Parameter name="mqtt.server.uri" value="mdm.dendroidecode.com:31000"/>

    <!-- Optional tag for delaying MQTT messages in milliseconds
     to avoid congestion when all devices are updating configuration at the same time -->
    <!-- <Parameter name="mqtt.message.delay" value="100"/> -->

    <!-- Fast device search by last characters, here's the length -->
    <Parameter name="device.fast.search.chars" value="5"/>

    <!-- Optional tag for MQTT authentication for more security
         (supported by Headwind MDM launcher v5.05 and above) -->
    <Parameter name="mqtt.auth" value="1"/>

    <!-- If you have any reverse proxies, specify them here (IP addresses,
    comma-separated) for correct logging of IP addresses -->
    <!-- <Parameter name="proxy.addresses" value="192.168.1.101"/> -->

    <!-- Name of the HTTP header containing the device IP address.
    Defaults to X-Real-IP -->
    <!-- <Parameter name="proxy.ip.header" value="X-Forwarded-For"/> -->

    <!-- Email parameters are necessary for password recovery -->
    <Parameter name="smtp.host" value="smtp.gmail.com"/>
    <Parameter name="smtp.port" value="465"/>
    <Parameter name="smtp.ssl" value="1"/>
    <Parameter name="smtp.starttls" value="1"/>
    <Parameter name="smtp.username" value="facturacion@dendroidecode.com"/>
    <Parameter name="smtp.password" value="gbuzidujkkczhtlf"/>
    <Parameter name="smtp.from" value="tics@dendroidecode.com.ec"/>
    <!-- Uncomment this line if you get 'Could not convert socket to TLS' -->
    <!-- <Parameter name="smtp.ssl.protocols" value="TLSv1.2"/> -->

<!-- These are the customer email templates
     Email paths may contain _LANGUAGE_ replaced to a two-letter language
     The default language is en -->
    <Parameter name="email.recovery.subj" value="/var/lib/tomcat9/work/emails/_LANGUAGE_/recovery_subj.txt"/>
    <Parameter name="email.recovery.body" value="/var/lib/tomcat9/work/emails/_LANGUAGE_/recovery_body.txt"/>

    <!-- IP filters for devices and web panel UI users, comma-separated networks or single IPs -->
<!--    <Parameter name="device.allowed.address" value='192.168.0.0/16,10.0.0.0/24'/>
    <Parameter name="ui.allowed.address" value='192.168.0.0/16,10.0.0.0/24,213.11.0.3'/> -->

</Context>
```

Reiniciar Tomcat con el comando

```bash
service tomcat9 restart
```

# Registro dispositivo

Puede continuar con la inscripción del dispositivo. Para registrar el dispositivo, siga [**estas instrucciones**](https://h-mdm.com/quick-start/#device) o mire el manual en video.

[https://youtu.be/S1t5bh4wLY0](https://youtu.be/S1t5bh4wLY0)

# Definición de base de datos

Para ver las bases de datos en PostgreSQL, puedes ejecutar el siguiente comando en la línea de comandos de `postgres=#`:

```sql
\l
```

Este comando lista todas las bases de datos disponibles en tu servidor de PostgreSQL, incluyendo la base de datos que configuraste para Headwind MDM (`hmdm_dendroidecode`).

Luego, para conectarte y ver las tablas dentro de esa base de datos específica, puedes usar:

```sql
\c hmdm_dendroidecode
```

```sql
\dt
```

1. `\c hmdm_dendroidecode` conecta a la base de datos `hmdm_dendroidecode`.
2. `\dt` muestra todas las tablas dentro de la base de datos conectada.

Para ver la estructura de una tabla por ejemplo `devices` y verificar los nombres de las columnas, puedes usar el siguiente comando en PostgreSQL:

```sql
\d devices
```

Si necesitas ver datos específicos de alguna tabla, puedes hacer una consulta básica, por ejemplo:

```sql
SELECT * FROM nombre_tabla LIMIT 10;
```

Esto mostrará los primeros 10 registros de la tabla que especifiques.

# Documentación de la Base de Datos

## List of Relations (Esquema relacional de la base de datos)

| **Schema** | **Name** | **Type** | **Owner** |
| --- | --- | --- | --- |
| public | applicationfilestocopytemp | table | hmdm |
| public | applications | table | hmdm |
| public | applicationversions | table | hmdm |
| public | applicationversionstemp | table | hmdm |
| public | configurationapplicationparameters | table | hmdm |
| public | configurationapplications | table | hmdm |
| public | configurationapplicationsettings | table | hmdm |
| public | configurationfiles | table | hmdm |
| public | configurations | table | hmdm |
| public | customers | table | hmdm |
| public | databasechangelog | table | hmdm |
| public | databasechangeloglock | table | hmdm |
| public | deviceapplicationsettings | table | hmdm |
| public | devicegroups | table | hmdm |
| public | devices | table | hmdm |
| public | devicestatuses | table | hmdm |
| public | groups | table | hmdm |
| public | icons | table | hmdm |
| public | pendingpushes | table | hmdm |
| public | pendingsignup | table | hmdm |
| public | permissions | table | hmdm |
| public | plugin_audit_log | table | hmdm |
| public | plugin_deviceinfo_deviceparams | table | hmdm |
| public | plugin_deviceinfo_deviceparams_device | table | hmdm |
| public | plugin_deviceinfo_deviceparams_gps | table | hmdm |
| public | plugin_deviceinfo_deviceparams_mobile | table | hmdm |
| public | plugin_deviceinfo_deviceparams_mobile2 | table | hmdm |
| public | plugin_deviceinfo_deviceparams_wifi | table | hmdm |
| public | plugin_deviceinfo_settings | table | hmdm |
| public | plugin_devicelog_log | table | hmdm |
| public | plugin_devicelog_setting_rule_devices | table | hmdm |
| public | plugin_devicelog_settings | table | hmdm |
| public | plugin_devicelog_settings_rules | table | hmdm |
| public | plugin_messaging_messages | table | hmdm |
| public | plugin_push_messages | table | hmdm |
| public | plugins | table | hmdm |
| public | pluginsdisabled | table | hmdm |
| public | pushmessages | table | hmdm |
| public | settings | table | hmdm |
| public | trialkey | table | hmdm |
| public | uploadedfiles | table | hmdm |
| public | usagestats | table | hmdm |
| public | userconfigurationaccess | table | hmdm |
| public | userdevicegroupsaccess | table | hmdm |
| public | userhints | table | hmdm |
| public | userhinttypes | table | hmdm |
| public | userrolepermissions | table | hmdm |
| public | userroles | table | hmdm |
| public | userrolesettings | table | hmdm |
| public | users | table | hmdm |

## Descripción de las Tablas

1. **applicationfilestocopytemp**: Almacena archivos temporales que se copiarán durante la instalación o actualización de aplicaciones.
2. **applications**: Contiene información sobre las aplicaciones registradas en el sistema.
3. **applicationversions**: Guarda las versiones de las aplicaciones disponibles.
4. **applicationversionstemp**: Almacena versiones temporales de las aplicaciones para su procesamiento o revisión.
5. **configurationapplicationparameters**: Configura parámetros específicos para las aplicaciones.
6. **configurationapplications**: Almacena configuraciones generales para las aplicaciones.
7. **configurationapplicationsettings**: Contiene ajustes específicos de configuración para las aplicaciones.
8. **configurationfiles**: Almacena archivos de configuración del sistema.
9. **configurations**: Guarda configuraciones del sistema y parámetros generales.
10. **customers**: Contiene información sobre los clientes registrados en el sistema.
11. **databasechangelog**: Registra los cambios realizados en la estructura de la base de datos.
12. **databasechangeloglock**: Bloquea la tabla de cambios de la base de datos para evitar conflictos durante las actualizaciones.
13. **deviceapplicationsettings**: Almacena configuraciones específicas de las aplicaciones en los dispositivos.
14. **devicegroups**: Contiene grupos de dispositivos para facilitar la gestión y configuración.
15. **devices**: Almacena información sobre los dispositivos registrados en el sistema.
16. **devicestatuses**: Contiene el estado de los dispositivos (activo, inactivo, etc.).
17. **groups**: Almacena información sobre los grupos de usuarios o dispositivos.
18. **icons**: Contiene los íconos utilizados en las aplicaciones o en la interfaz del sistema.
19. **pendingpushes**: Almacena mensajes o acciones pendientes de ser enviadas a los dispositivos.
20. **pendingsignup**: Contiene información sobre registros de usuarios pendientes de confirmación.
21. **permissions**: Almacena los permisos asignados a los diferentes roles de usuarios.
22. **plugin_audit_log**: Registra las auditorías realizadas por los plugins del sistema.
23. **plugin_deviceinfo_deviceparams**: Almacena parámetros relacionados con la información de los dispositivos a través de plugins.
24. **plugin_deviceinfo_deviceparams_device**: Contiene información de parámetros específicos de los dispositivos gestionados por plugins.
25. **plugin_deviceinfo_deviceparams_gps**: Almacena parámetros de GPS relacionados con los dispositivos.
26. **plugin_deviceinfo_deviceparams_mobile**: Contiene parámetros específicos de los dispositivos móviles.
27. **plugin_deviceinfo_deviceparams_mobile2**: Almacena una segunda serie de parámetros para dispositivos móviles.
28. **plugin_deviceinfo_deviceparams_wifi**: Contiene parámetros relacionados con la información Wi-Fi de los dispositivos.
29. **plugin_deviceinfo_settings**: Almacena configuraciones de los plugins de información de dispositivos.
30. **plugin_devicelog_log**: Registra los logs generados por los plugins relacionados con dispositivos.
31. **plugin_devicelog_setting_rule_devices**: Almacena reglas de configuración para los logs de dispositivos gestionados por plugins.
32. **plugin_devicelog_settings**: Contiene configuraciones de logging para dispositivos a través de plugins.
33. **plugin_devicelog_settings_rules**: Almacena reglas específicas para la configuración de logs.
34. **plugin_messaging_messages**: Contiene mensajes enviados a través de los plugins de mensajería.
35. **plugin_push_messages**: Almacena mensajes que se envían a los dispositivos a través de push notifications.
36. **plugins**: Contiene información sobre los plugins instalados en el sistema.
37. **pluginsdisabled**: Almacena información sobre plugins desactivados.
38. **pushmessages**: Contiene mensajes pendientes de ser enviados a los dispositivos mediante notificaciones push.
39. **settings**: Almacena configuraciones generales del sistema.
40. **trialkey**: Contiene claves de prueba para el uso temporal de características del sistema.
41. **uploadedfiles**: Almacena archivos que han sido subidos al sistema.
42. **usagestats**: Contiene estadísticas de uso del sistema y las aplicaciones.
43. **userconfigurationaccess**: Almacena información sobre el acceso de los usuarios a configuraciones específicas.
44. **userdevicegroupsaccess**: Contiene información sobre el acceso de los usuarios a grupos de dispositivos.
45. **userhints**: Almacena sugerencias o consejos para los usuarios.
46. **userhinttypes**: Contiene tipos de sugerencias disponibles para los usuarios.
47. **userrolepermissions**: Almacena los permisos asignados a los diferentes roles de usuarios.
48. **userroles**: Contiene información sobre los roles asignados a los usuarios.
49. **userrolesettings**: Almacena configuraciones específicas para los roles de los usuarios.
50. **users**: Contiene información sobre los usuarios registrados en el sistema.


