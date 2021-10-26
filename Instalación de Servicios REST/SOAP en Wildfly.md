# Instalación de Servicios REST/WS en Wildfly

En el siguiente documento se va a mostrar como instalar y desplegar un servicio REST y otro SOAP en el contenedor de aplicaciones Wildfly.

# Requerimientos

Disponer de Java JDK, Maven, permisos de administrador y haber realizado los pasos de este [repositorio](https://github.com/gdborja/wildfly/blob/main/Instalaci%C3%B3n%20de%20Wildfly%20en%20Linux.md).

# Instalación del servicio helloworld-rs

Descargar el repositorio de ejemplos [WildFly Quickstarts](https://github.com/wildfly/quickstart/).

Dentro del mismo, localizar el directorio helloworld-rs, entrar en este y ejecutar el comando mvn clean install para crear el fichero ***.war** del servicio.

```console
mvn clean install
```

# Despliegue del servicio

En el navegador, acceder al panel administrativo de wildfly --> seleccionar la pestaña de Deployments --> pulsar + --> Upload Deployment y subir el fichero .war localizado en 
***/helloworld-rs/target/helloworld-rs.war***.

Continuar pulsando el botón next y se mostrará un panel donde le podemos cambiar el nombre al servicio y activarlo o no.

# Instalación del servicio helloworld-ws

Para instalar y desplegar este servicio hay que realizar los pasos anteriores, pero en el directorio helloworld-ws y seleccionando el fichero .war de este.