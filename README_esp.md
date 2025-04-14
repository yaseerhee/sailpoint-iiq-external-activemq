
# ActiveMQ configurado como Broker Externo para SailPoint IIQ 8.4

Este repositorio contiene un archivo `activemq.xml` listo para usar con Apache ActiveMQ como **broker externo** en una integración con **SailPoint IdentityIQ 8.4**, especialmente para el módulo de **Data Extract** y colas de estadísticas.

> ⚠️ Solo se publica el archivo `activemq.xml` para mantener el repositorio limpio. Otros archivos necesarios se documentan más abajo.

---

## 📄 Archivo incluido

### `conf/activemq.xml`

Este archivo define:

- Las colas necesarias para estadísticas y procesamiento de tareas de SailPoint IIQ.
- Roles y permisos de usuarios.
- Autenticación mediante JAAS (`jaasAuthenticationPlugin`).
- Permisos para colas y topics internos de ActiveMQ (`Advisory`, `Statistics`).

---

## 🔐 Configuración de autenticación con archivos planos

Después de descargar y descomprimir ActiveMQ desde la web oficial, deberás configurar dos archivos clave si usas autenticación basada en archivos (`login.config` + JAAS):

### `users.properties`

Define los usuarios y contraseñas que usarán SailPoint IIQ para conectarse al broker:

```properties
# Usuarios definidos para conexiones desde SailPoint IIQ
amq_adminuser=adminpassword123
amq_produceruser=producerpassword123
amq_consumeruser=consumerpassword123
```

📍 Ubicación esperada: `/opt/activemq/conf/users.properties`

---

### `groups.properties`

Define los roles que se asignan a cada usuario. Estos deben coincidir con los roles definidos en `activemq.xml` (admins, producers, consumers).

```properties
# Roles asignados a los usuarios
admins=amq_adminuser
producers=amq_produceruser
consumers=amq_consumeruser
```

📍 Ubicación esperada: `/opt/activemq/conf/groups.properties`

---

## 🔄 Reinicio necesario

Cada vez que modifiques los archivos de configuración:

```bash
/opt/activemq/bin/activemq stop
/opt/activemq/bin/activemq start
```

---

## 🧩 Integración con Active Directory (opcional para entornos productivos)

Puedes reemplazar el uso de archivos planos por una integración con **Active Directory** mediante JAAS. Para ello:

### 1. Modifica el archivo `login.config`:

```bash
sudo nano /opt/activemq/conf/login.config
```

Ejemplo de configuración para LDAP (AD):

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

> Puedes usar también el módulo `LdapLoginModule` de Apache Shiro o Spring LDAP, si deseas más control.

---

### 2. Configura los grupos de AD en `activemq.xml`

Debes asegurarte de que los grupos de Active Directory se llamen igual que los definidos como roles en ActiveMQ:

```xml
<authorizationEntry queue=">" read="consumers" write="producers" admin="admins"/>
```

Y que tus usuarios en AD pertenezcan a esos grupos (`consumers`, `producers`, `admins`).

---

## ⚙️ Configuración en SailPoint IIQ

Ve a `Global Settings → Messaging Configuration` y rellena como en el siguiente ejemplo:

📷 Aquí deberías incluir la imagen `sailpoint-messaging-config.png` que muestre cómo has rellenado el formulario.

### Valores típicos:

| Campo                             | Valor Ejemplo                                      |
|----------------------------------|----------------------------------------------------|
| Broker Type                      | External                                           |
| Client Connection String         | `failover:(tcp://activemq-host:61616)?initialReconnectDelay=100&maxReconnectAttempts=5` |
| Consumer Username                | `amq_consumeruser`                                 |
| Consumer Password                | `consumerpassword123`                              |
| Producer Username                | `amq_produceruser`                                 |
| Producer Password                | `producerpassword123`                              |
| Admin Username                   | `amq_adminuser`                                    |
| Admin Password                   | `adminpassword123`                                 |
| Broker Statistics Queue Name     | `queue://iiqBrokerStatsQueue`                      |
| Destinations Statistics Queue    | `queue://iiqDestinationStatsQueue`                 |
| Subscriptions Statistics Queue   | `queue://iiqSubscriptionStatsQueue`                |

---

## 🛡️ Recomendaciones finales

- Usa autenticación con Active Directory o Vault en entornos productivos.
- Asegura los puertos del broker y permite solo tráfico desde servidores autorizados.
- Considera usar SSL (`ssl://`) en lugar de TCP sin cifrar.
- Monitorea el broker desde la consola web o vía JMX.

---

## 👤 Autor

Creado por [Yaser Haddad ]([https://github.com/yaseerhee]), Arquitecto de soluciones IAM en SailPoint IIQ.
