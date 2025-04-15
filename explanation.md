
# Why this project?

The motivation for this project comes from a widespread issue in the SailPoint IdentityIQ community.

The **embedded ActiveMQ broker** included in IdentityIQ 8.4 has been known to cause significant operational problems, especially when used in environments with multiple IIQ instances, Access History, or Data Extract modules enabled. These issues have been consistently reported in the SailPoint Developer Community and Compass.

### The Problem:
Because the embedded broker is internal to the product, it cannot be easily modified, monitored, or isolated. When it fails:
- It prevents IdentityIQ from starting
- It causes Data Extract and Access History features to break
- It leads to messaging failures that are difficult to trace

### The Gap:
There was **no official documentation** available for configuring IdentityIQ to use an external ActiveMQ broker — and configuring it manually involved a lot of trial and error.

---

### The Solution:
This project was created to:
- Help teams transition from the embedded broker to a **fully external, production-ready ActiveMQ setup**
- Provide a **tested configuration** that works out-of-the-box with IdentityIQ 8.4
- Avoid the risk that a broker failure crashes the entire IdentityIQ application
- Improve observability and control by running ActiveMQ independently

With this approach, IdentityIQ and ActiveMQ become **decoupled** services. You gain:
- Better stability and fault tolerance
- Cleaner separation of concerns
- Scalable and centralized message handling
- Support for systemd, secure authentication, and monitoring
