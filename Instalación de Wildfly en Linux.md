# Instalación de Wildfly en Linux

En el siguiente repositorio se va a mostrar como instalar y configurar el contenedor de aplicaciones Wildfly.

# Requerimientos

Disponer de Java JDK, Maven y permisos de administrador.

# Instalación de Wildfly

Como es habitual, el primero paso es actualizar los repositorios y a raíz de estos actualizar el sistema.

```console
sudo apt update && sudo apt upgrade
```

Dirigirse al directorio de archivos temporales /tmp y descargar de la [página oficial de Wildfly](https://www.wildfly.org/) la versión más estable.

```console
cd /tmp
wget https://github.com/wildfly/wildfly/releases/download/25.0.0.Final/wildfly-25.0.0.Final.tar.gz
```

Por motivos de seguridad, crear el nuevo grupo y usuario wildfly sin acceso desde la terminal.

```console
sudo groupadd -r wildfly
sudo useradd -r -g wildfly -d /opt/wildfly -s /sbin/nologin wildfly
```

Descomprimir el paquete descargado y mover el mismo a /opt/wildfly-25.0.0.Final

```console
tar -xvzf wildfly-25.0.0.Final.tar.gz
sudo mv wildfly-25.0.0.Final /opt/wildfly-25.0.0.Final
```
Crear un enlace simbólico al directorio.

```console
sudo ln -s /opt/wildfly-25.0.0.Final /opt/wildfly
```
Cambiar como dueño y grupo del directorio /opt/wildfly al usuario y grupo wildfly

```console
sudo chown -R wildfly:wildfly /opt/wildfly/
```

# Iniciar el Servicio

Crear el directorio /etc/wildfly y copiar dentro del mismo el fichero de configuración de arranque wildfly.conf

```console
sudo mkdir /etc/wildfly
sudo cp /opt/wildfly/docs/contrib/scripts/systemd/wildfly.conf /etc/wildfly/
```

El fichero tiene por defecto el arranque standalone el cual utiliza el fichero de configuración standalone.xml

Copiar el ejercutable en el directorio /opt/wildfly/bin

```console
sudo cp /opt/wildfly/docs/contrib/scripts/systemd/launch.sh /opt/wildfly/bin/
```
Anadir al fichero permisos de ejecución.

```console
sudo sh -c 'chmod +x /opt/wildfly/bin/*.sh'
```
Copiar el fichero wildfly.service dentro del directorio /etc/systemd/system/ para que el sistema trate a este como un servicio más del sistema.

```console
sudo cp /opt/wildfly/docs/contrib/scripts/systemd/wildfly.service /etc/systemd/system/
```

Recargar todos los servicios del sistema, iniciar el servicio wildfly y comprobar que este ejecutándose correctamente.

```console
sudo systemctl daemon-reload
sudo systemctl start wildfly
sudo systemctl status wildfly
```

Anadir el inicio automático cada vez que se arranque el sistema.

```console
sudo systemctl enable wildfly
```

# Configurar Wildfly

Por defecto Wildfly utiliza el puerto 8080, es posible que este lo tengamos ocupado por otro servicio, por lo tanto, modificaremos el puerto por el 8083 en la etiqueta del fichero standalone.xml

```console
sudo nano /opt/wildfly/standalone/configuration/standalone.xml

<socket-binding name="http" port="${jboss.http.port:8083}"/>
```

Permitimos el trafico por este puerto en el cortafuegos. 

```console
sudo ufw allow 8083/tcp
```

# Anadir usuarios

Para añadir usuarios con capacidad de administrar las aplicaciones desplegadas en Wildfly, tenemos que lanzar el ejecutable que se encuentra en el directorio /opt/wildfly/bin/add-user.sh e ir especificando las propiedades del mismo.

```console
sudo /opt/wildfly/bin/add-user.sh
elegir la opcion a - para crearlo como administrador.
username - nombre del usuario.
password - contraseña del usuario.
group - ManagementRealm para añadirlo al grupo de los administradores.
```
Este usuario es el que utilizaremos para acceder al panel de administración.

# Gestionar la consola de forma remota

Ejecuta el siguiente comando:

```console
sudo nano /etc/wildfly/wildfly.conf
```

Obteniendo algo similar:

```console  
WILDFLY_CONSOLE_BIND=0.0.0.0

#Nos permite restringir la configuracion de accesso
```
```console  
sudo nano /opt/wildfly/bin/launch.sh
```

 y verificando que se encuentra:
```console
$WILDFLY_HOME/bin/domain.sh -c $2 -b $3 -bmanagement $4

else

$WILDFLY_HOME/bin/standalone.sh -c $2 -b $3 -bmanagement $4
```

Esta ejecución indica en que modo deseamos lanzar el servidor (_standalone o domain_).

Para finalizar hemos de realizar:

```console
  sudo systemctl restart wildfly

  sudo nano /etc/systemd/system/wildfly.service

  ExecStart=/opt/wildfly/bin/launch.sh $WILDFLY_MODE $WILDFLY_CONFIG $WILDFLY_BIND $WILDFLY_CONSOLE_BIND

  sudo systemctl daemon-reload

  sudo systemctl restart wildfly
```  
<!--
  y abriríamos la consola de _cli_

```console
  cd /opt/wildfly/bin/
  ./jboss-cli.sh --connect
```
-->

 Si lo deseamos o es necesario podemos abrir el acceso a la consola de administración modificando el fichero __standalone.xml__ indicado anteriormente, y buscando la siguiente entrada :

  ```console  
    <interfaces>
        <interface name="management">
            <inet-address value="${jboss.bind.address.management:127.0.0.1}"/>
        </interface>
        <interface name="public">
            <inet-address value="${jboss.bind.address:127.0.0.1}"/>
        </interface>
    </interfaces>  
  ```
  Eliminando o comentando las etiquetas __inet-address__, o indicando la __ip publica__ o __0.0.0.0__, según necesidades.
