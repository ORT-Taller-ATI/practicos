## Application Servers

### Apache Tomcat

Vamos a instalar Apache Tomcat 9 (último release `v9.0.75`) en `Amazon Linux 2`

Pasos:

1. Instalmos jdk y dependencias: `$ sudo yum install java-1.8*`
2. Podemos comprobar que esté instalado con el comando `java -version`

```bash
# java -version
openjdk version "1.8.0_362"
OpenJDK Runtime Environment (build 1.8.0_362-b08)
OpenJDK 64-Bit Server VM (build 25.362-b08, mixed mode)
```

3. Descargamos `Apache tomcat` desde la página oficial: `wget https://downloads.apache.org/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz`
4. Descomprimimos: `$ tar -xzf apache-tomcat-9.0.75.tar.gz`
5. Renombramos el directorio: `mv apache-tomcat-9.0.75 tomcat`
6. Creamos el usuario tomcat: `useradd -r tomcat`
7. Movemos el directorio tomcat a /opt: `sudo mv tomcat /opt/`
8. Asignamos permisos: `sudo chown -R tomcat:tomcat /opt/tomcat`
9. Creamos el service unit: 
```bash
$ sudo tee /etc/systemd/system/tomcat.service<<EOF

[Unit]
Description=Tomcat Server
After=syslog.target network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment=CATALINA_HOME=/opt/tomcat9
Environment=CATALINA_BASE=/opt/tomcat9
Environment=CATALINA_PID=/opt/tomcat9/temp/tomcat.pid

ExecStart=/opt/tomcat9/bin/catalina.sh start
ExecStop=/opt/tomcat9/bin/catalina.sh stop

RestartSec=12
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```
10. Recargamos systemd: `sudo systemctl daemon-reload`
11. Configuramos acceso al app server desde cualquier origen: 

```bash
$ sudo vim /opt/tomcat/webapps/manager/META-INF/context.xml

## Agregamos el siguiente bloque dentro de <context> tag:
<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />
```

Asi quedaria el archivo completo:

```xml
<Context antiResourceLocking="false" privileged="true" >
  <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                   sameSiteCookies="strict" />
          <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
```
12. Configuramos usuarios

```bash
$ vim /opt/tomcat/conf/tomcat-users.xml

## Bajo el tag <tomcat-users> agregamos esto:

role rolename="admin"/> 
<role rolename="admin-gui"/> 
<role rolename="admin-script"/> 
<role rolename="manager"/> 
<role rolename="manager-gui"/> 
<role rolename="manager-script"/> 
<role rolename="manager-jmx"/> 
<role rolename="manager-status"/> 
<user name="admin" password="una-password-segura" roles="admin,manager,admin-gui,admin-script,manager-gui,manager-script,manager-jmx,manager-status" />
```
13. Iniciamos el servicio: `systemctl start tomcat`