# Spring Boot Kubernetes Probes Demo (Liveness, Readiness, Startup)

This project demonstrates how to deploy a **Spring Boot application on Kubernetes with production-style health probes**.

The repository focuses on an important Kubernetes reliability pattern used in real DevOps environments:

- **Liveness Probe**
- **Readiness Probe**
- **Startup Probe**
- **Graceful Shutdown**
- **preStop Lifecycle Hook**

These mechanisms help achieve **high availability, zero downtime deployments, and automatic container recovery** in Kubernetes clusters.

---

# Table of Contents

1. Introduction  
2. Why Health Probes Are Important  
3. Kubernetes Probe Types  
4. Graceful Shutdown in Kubernetes  
5. How Zero Downtime Deployment Works  

---

# 1. Introduction

Modern cloud-native applications are deployed inside containers and orchestrated using Kubernetes.

However, Kubernetes needs a way to determine:

- Is the application **alive**?
- Is the application **ready to receive traffic**?
- Has the application **started successfully**?

To solve this, Kubernetes provides **health probes**.

Spring Boot exposes health endpoints through **Spring Boot Actuator**, which Kubernetes can use to check application status.

---

# 2. Why Health Probes Are Important

Without health checks, Kubernetes cannot determine if an application is functioning correctly.

Common production issues include:

- Application thread deadlock
- Database connection failure
- Application startup delay
- Memory leaks
- Stuck processes

Health probes allow Kubernetes to:

- Restart unhealthy containers automatically
- Prevent traffic from reaching unready pods
- Avoid premature container restarts during startup

---

# 3. Kubernetes Probe Types

Kubernetes provides **three main types of probes**.

---

## 1. Liveness Probe

The **liveness probe** checks whether the application is still running.

If the probe fails repeatedly:
- Kubernetes restarts the container

### Typical failure scenarios:

- Application deadlock
- JVM freeze
- Infinite loop

### Example configuration

```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
```
---
## 2. Readiness Probe

The **readiness probe** determines if the application is ready to receive traffic.

If the probe fails:
- Pod is removed from Service endpoints

Traffic will not be routed to that pod until it becomes ready again.

### Typical scenarios

- Database connection not ready
- Cache warming
- External service dependency unavailable

### Example configuration

```yaml
readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
```
---

## 3. Startup Probe

The **startup probe** is designed for applications that take a long time to start.

Without a startup probe, Kubernetes may think the application is unhealthy and restart it too early.

Startup probe ensures:
- Liveness probe does not run until the application has successfully started

### Example configuration

```yaml
startupProbe:
  httpGet:
    path: /actuator/health
    port: 8080
```
---

# 4. Graceful Shutdown in Kubernetes

During deployments or scaling operations, Kubernetes terminates running pods.

Without graceful shutdown:

- Active user requests may fail

To prevent this problem, Kubernetes supports lifecycle hooks.

The preStop hook allows the container to execute logic before termination.

### Example configuration

```yaml
lifecycle:
  preStop:
    exec:
      command: ["sh", "-c", "sleep 10"]
```
---

This allows the application to:
- Stop receiving new traffic
- Complete existing requests
- Shut down safely

---

# 5. How Zero Downtime Deployment Works

During a rolling update, Kubernetes replaces old pods with new ones gradually.

### Deployment Process
1. New pod starts
2. Readiness probe checks if pod is ready
3. Service sends traffic to ready pods only
4. Old pod begins termination
5. preStop hook delays shutdown
6. Existing requests complete


