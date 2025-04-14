
# ActiveMQ configured as External Broker for SailPoint IIQ 8.4

This repository contains a ready-to-use `activemq.xml` file configured for Apache ActiveMQ as an **external messaging broker** integrated with **SailPoint IdentityIQ 8.4**, especially for the **Data Extract** module and statistics queues.

> ⚠️ Only the `activemq.xml` file is published to keep the repository clean. Other required configuration files are documented below.

---

## 📄 Included File

### `conf/activemq.xml`

This file defines:

- The queues required for SailPoint IIQ data extraction and statistics.
- Roles and access control for users.
- Authentication via JAAS (`jaasAuthenticationPlugin`).
- Permissions for internal ActiveMQ topics (`Advisory`, `Statistics`).

---

## 🔐 Authentication Configuration Using Flat Files

After downloading and extracting ActiveMQ from the official website, configure the following files if using file-based authentication (`login.config` + JAAS):

### `users.properties`

Defines the users and passwords that SailPoint IIQ will use to connect to the broker:

```properties
# Users for connections from SailPoint IIQ
amq_adminuser=adminpassword123
amq_produceruser=producerpassword123
amq_consumeruser=consumerpassword123
```

📍 Expected path: `/opt/activemq/conf/users.properties`

---

### `groups.properties`

Defines the roles assigned to each user. These must match the roles defined in `activemq.xml` (admins, producers, consumers).

```properties
# Role assignments for the users
admins=amq_adminuser
producers=amq_produceruser
consumers=amq_consumeruser
```

📍 Expected path: `/opt/activemq/conf/groups.properties`

---

## 🔄 Required Restart

Whenever you modify these configuration files, restart ActiveMQ:

```bash
/opt/activemq/bin/activemq stop
/opt/activemq/bin/activemq start
```

---

## 🧩 Integration with Active Directory (for production environments)

You can replace flat file authentication with integration via **Active Directory (AD)** using JAAS. To do this:

### 1. Edit the `login.config` file:

```bash
sudo nano /opt/activemq/conf/login.config
```

Example LDAP/AD configuration:

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

> You may also use Apache Shiro or Spring LDAP modules for more advanced control.

---

### 2. Map AD groups to ActiveMQ roles in `activemq.xml`

Ensure your AD groups are named exactly as the roles in the authorization section:

```xml
<authorizationEntry queue=">" read="consumers" write="producers" admin="admins"/>
```

Make sure your AD users belong to the matching groups (`consumers`, `producers`, `admins`).

---

## ⚙️ SailPoint IIQ Configuration

In IdentityIQ, go to:

`Global Settings → Messaging Configuration`

Fill out the form as follows:

📷 Replace this text with a screenshot named `sailpoint-messaging-config.png` showing how you filled out the form.

### Typical Values:

| Field                            | Example Value                                      |
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

## 🛡️ Additional Recommendations

- In production, use secure authentication (e.g., AD, Vault).
- Restrict broker access to authorized servers only.
- Consider switching to SSL (`ssl://`) instead of plain TCP.
- Monitor broker health using the web console or JMX.

---

## 👤 Author

Created by [Yasser](https://github.com/tuusuario), IAM Solution Architect specialized in SailPoint IIQ.
