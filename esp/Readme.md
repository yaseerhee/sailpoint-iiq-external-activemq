

# ActiveMQ configurado como Broker Externo para SailPoint IIQ 8.4

Este repositorio contiene un archivo `activemq.xml` listo para usar con Apache ActiveMQ como **broker externo de mensajería** integrado con **SailPoint IdentityIQ 8.4**, especialmente para el módulo de **Data Extract** y las colas de estadísticas.

> ⚠️ Solo se publica el archivo `activemq.xml` para mantener el repositorio limpio. Otros archivos requeridos se documentan más abajo.

---

## ⚙️ Requisitos e Instalación

Para utilizar esta configuración, necesitarás lo siguiente:

### ✅ Requisitos del sistema

- **Sistema Operativo**: RHEL 8 o 9 (probado en RHEL 9.0)
- **Java**: OpenJDK 11 (ActiveMQ 5.17.x requiere Java 8 u 11)
- **Memoria**: Mínimo 2 GB RAM (4 GB recomendado)
- **Puertos**: Abrir TCP 61616 (broker) y 8161 (consola web)
- **Red**: Asegúrate de que el servidor de IIQ pueda alcanzar este host de ActiveMQ

### 📦 Software necesario

- Apache ActiveMQ 5.17.x:
  - [Descargar aquí](https://activemq.apache.org/components/classic/download/)
  - Selecciona la distribución binaria (`apache-activemq-5.17.x-bin.tar.gz`)

### 📁 Estructura del directorio de instalación

Asumiendo que lo instalas en `/opt/activemq`:

```bash
sudo mkdir -p /opt/activemq
cd /opt
sudo tar -xzf apache-activemq-5.17.x-bin.tar.gz
sudo mv apache-activemq-5.17.x activemq
```

Asegúrate de que los permisos sean correctos:

```bash
sudo chown -R activemq:activemq /opt/activemq
```

### 🚀 Iniciar el broker

Para iniciar ActiveMQ:

```bash
/opt/activemq/bin/activemq start
```

Opcional: crea un servicio systemd para ejecución persistente.

### 🔥 Abrir puertos necesarios

Asegúrate de que los siguientes puertos estén abiertos:

```bash
sudo firewall-cmd --permanent --add-port=61616/tcp
sudo firewall-cmd --permanent --add-port=8161/tcp
sudo firewall-cmd --reload
```

Una vez que el servicio esté en marcha, ActiveMQ creará automáticamente las colas necesarias cuando SailPoint IdentityIQ las utilice por primera vez (siempre que el usuario autenticado tenga permisos de administrador sobre las colas).

---

## 📄 Archivo incluido

### `conf/activemq.xml`

Este archivo define:

- Las colas necesarias para estadísticas y procesamiento de tareas de SailPoint IIQ.
- Roles y permisos para usuarios.
- Autenticación mediante JAAS (`jaasAuthenticationPlugin`).
- Permisos para topics internos de ActiveMQ (`Advisory`, `Statistics`).

---

## 🔐 Configuración de autenticación con archivos planos

Después de descargar y descomprimir ActiveMQ, debes configurar los siguientes archivos si vas a usar autenticación mediante archivos (`login.config` + JAAS):

### `users.properties`

Define los usuarios y contraseñas que utilizará SailPoint IIQ para conectarse al broker:

```properties
# Usuarios para conexiones desde SailPoint IIQ
amq_adminuser=adminpassword123
amq_produceruser=producerpassword123
amq_consumeruser=consumerpassword123
```

📍 Ruta esperada: `/opt/activemq/conf/users.properties`

---

### `groups.properties`

Define los roles asignados a cada usuario. Estos deben coincidir con los definidos en `activemq.xml` (admins, producers, consumers).

```properties
# Asignación de roles para los usuarios
admins=amq_adminuser
producers=amq_produceruser
consumers=amq_consumeruser
```

📍 Ruta esperada: `/opt/activemq/conf/groups.properties`

---

## 🔄 Reinicio necesario

Cada vez que modifiques estos archivos:

```bash
/opt/activemq/bin/activemq stop
/opt/activemq/bin/activemq start
```

---

## 🧩 Integración con Active Directory (recomendado para producción)

Puedes reemplazar los archivos planos por una integración con **Active Directory** mediante JAAS. Para ello:

### 1. Edita el archivo `login.config`:

```bash
sudo nano /opt/activemq/conf/login.config
```

Ejemplo de configuración para LDAP/AD:

```properties
activemq-domain {
  com.sun.security.auth.module.LdapLoginModule REQUIRED
    userProvider="ldap://dc.example.com:389/ou=Users,dc=example,dc=com"
    authIdentity="{USERNAME}"
    userFilter="(sAMAccountName={USERNAME})"
    useSSL=false
    debug=true;
};
```

> También puedes usar módulos como Apache Shiro o Spring LDAP para mayor control.

---

### 2. Asocia los grupos de AD a los roles de ActiveMQ

Asegúrate de que los grupos de Active Directory se llamen igual que los roles definidos:

```xml
<authorizationEntry queue=">" read="consumers" write="producers" admin="admins"/>
```

Y que tus usuarios de AD pertenezcan a esos grupos (`consumers`, `producers`, `admins`).

---

## ⚙️ Configuración en SailPoint IIQ

En IdentityIQ, ve a:

`Global Settings → Messaging Configuration`

Completa el formulario así:

📷 Aquí deberías incluir la imagen `sailpoint-messaging-config.png` con el formulario completado.

### Valores típicos:

| Campo                             | Valor de ejemplo                                     |
|----------------------------------|------------------------------------------------------|
| Broker Type                      | External                                             |
| Client Connection String         | `failover:(tcp://activemq-host:61616)?initialReconnectDelay=100&maxReconnectAttempts=5` |
| Consumer Username                | `amq_consumeruser`                                   |
| Consumer Password                | `consumerpassword123`                                |
| Producer Username                | `amq_produceruser`                                   |
| Producer Password                | `producerpassword123`                                |
| Admin Username                   | `amq_adminuser`                                      |
| Admin Password                   | `adminpassword123`                                   |
| Broker Statistics Queue Name     | `queue://iiqBrokerStatsQueue`                        |
| Destinations Statistics Queue    | `queue://iiqDestinationStatsQueue`                   |
| Subscriptions Statistics Queue   | `queue://iiqSubscriptionStatsQueue`                  |

---

## 🛠️ Opcional: Crear servicio systemd (recomendado en producción)

Para habilitar el arranque automático y gestión adecuada del broker:

### 📄 Crear el archivo del servicio:

```bash
sudo nano /etc/systemd/system/activemq.service
```

Pega el siguiente contenido:

```ini
[Unit]
Description=Apache ActiveMQ Broker
After=network.target

[Service]
User=activemq
Group=activemq
ExecStart=/opt/activemq/bin/activemq start
ExecStop=/opt/activemq/bin/activemq stop
Restart=on-failure
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

### ✅ Habilita e inicia el servicio

```bash
sudo systemctl daemon-reload
sudo systemctl enable activemq
sudo systemctl start activemq
sudo systemctl status activemq
```

Esto permite que ActiveMQ arranque al iniciar el sistema y sea gestionable vía `systemctl`.

---

## 👤 Autor

Creado por [Yaser Haddad](https://github.com/yaseerhee), arquitecto de soluciones IAM especializado en SailPoint IIQ.
