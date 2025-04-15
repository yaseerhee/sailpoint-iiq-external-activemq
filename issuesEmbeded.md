
# Common Issues Reported by the Community

Many organizations have experienced significant issues when using the embedded ActiveMQ broker in IdentityIQ 8.4. These challenges are widely discussed across the SailPoint Developer Community, Compass, and technical forums:

### 1. Startup Failures and Service Downtime
IdentityIQ often fails to start when the embedded broker is misconfigured or locked. Tomcat hangs during initialization, showing errors in the logs related to broker startup, port conflicts, or queue locks.

### 2. Access History Feature Malfunctions
Access History and Data Extract rely heavily on the broker. When embedded messaging fails, these features stop working properly—tasks don’t complete, queues accumulate, and filtering becomes inconsistent.

### 3. Configuration Complexity
The embedded broker is sensitive to configuration details. In clustered or multi-node setups, hostname resolution, database locking, and queue persistence settings are common sources of failure.

### 4. Lack of Observability
The embedded broker lacks real-time monitoring, queue stats, and structured logs—making it difficult to troubleshoot or scale.

---

## Why Move to an External Broker? — An Architectural Perspective

### 1. Separation of Concerns
IdentityIQ and ActiveMQ serve different purposes. Running them in separate processes isolates messaging logic from identity logic, which simplifies maintenance, testing, and scalability.

### 2. Fault Isolation
If the broker fails, you can restart or recover it independently without affecting IdentityIQ. This is key for high-availability environments.

### 3. Scalability and Control
An external broker can be scaled vertically or horizontally, placed on dedicated hardware, and tuned independently of the IIQ JVM. It gives you full control over message retention, delivery modes, and thread handling.

### 4. Security and Authentication
With an external broker, you can:
- Use JAAS, LDAP, or Active Directory for authentication
- Define role-based access to queues and topics
- Enable TLS/SSL encryption for secure communication

### 5. Monitoring and Manageability
External ActiveMQ provides:
- A web-based console for monitoring and diagnostics
- Structured logs and metrics
- Integration with systemd or Kubernetes (Depends of your arch)
- Better alerting, visibility, and compliance
