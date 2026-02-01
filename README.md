# Spring Boot WAR with HTTPS, System-Scope Libraries, and OpenShift Deployment
## Complete End-to-End Implementation Guide

## 📋 Table of Contents
- [Project Overview](#project-overview)
- [Prerequisites](#prerequisites)
- [File Structure](#file-structure)
- [Implementation Steps](#implementation-steps)
- [Build & Verification](#build--verification)
- [Docker & OpenShift](#docker--openshift)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)

## 🎯 Project Overview
Deploy a Spring Boot WAR application with:
- ✅ Custom system-scope JAR dependencies
- ✅ HTTPS/TLS encryption
- ✅ Docker containerization
- ✅ OpenShift deployment with TLS passthrough
- ✅ Full production-ready pipeline

---

## 🔧 Prerequisites
```bash
# Verify all tools are installed
java -version                    # Java 17+
mvn -version                     # Maven 3.8+
podman --version                 # or docker
oc version                       # OpenShift CLI
openssl version                  # For certificate generation

# Install missing packages (RHEL/CentOS)
yum install -y java-17-openjdk-devel maven podman openssl
```

---

## 📁 File Structure
```bash
springboot-war-tls/
├── README.md
├── pom.xml
├── Dockerfile
├── libs/
│   ├── commons-io-2.16.1.jar
│   └── commons-lang3-3.14.0.jar
├── src/
│   └── main/
│       ├── java/com/example/demo/
│       │   ├── DemoApplication.java
│       │   └── api/
│       │       └── UtilController.java
│       └── resources/
│           ├── application.properties
│           └── keystore.p12          # Generated TLS keystore
│
├── scripts/
│   └── generate-tls.sh
│
└── target/                         # Generated after build
    └── myapp.war
```

---

## 🚀 Implementation Steps

## 📁 **COMPLETE FILE STRUCTURE WITH CODE**

### **1. `src/main/java/com/example/demo/DemoApplication.java`**
```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

@SpringBootApplication
public class DemoApplication extends SpringBootServletInitializer {
  
  @Override
  protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
    return application.sources(DemoApplication.class);
  }
  
  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }
}
```

### **2. `src/main/java/com/example/demo/api/UtilController.java`**
```java
package com.example.demo.api;

import org.apache.commons.io.IOUtils;
import org.apache.commons.lang3.StringUtils;
import org.apache.commons.lang3.RandomStringUtils;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.ByteArrayInputStream;
import java.nio.charset.StandardCharsets;
import java.util.LinkedHashMap;
import java.util.Map;

@RestController
public class UtilController {
  
  @GetMapping(value = "/api/util", produces = MediaType.APPLICATION_JSON_VALUE)
  public Map<String, Object> util() throws Exception {
    // Demonstrate commons-lang3 usage
    String raw = "   Hello OpenShift with HTTPS   ";
    String trimmed = StringUtils.trim(raw);
    String randomToken = RandomStringUtils.randomAlphanumeric(16);
    boolean containsOpenShift = StringUtils.containsIgnoreCase(raw, "openshift");
    
    // Demonstrate commons-io usage
    String payload = "commons-io stream operations working with HTTPS";
    ByteArrayInputStream inputStream = new ByteArrayInputStream(payload.getBytes(StandardCharsets.UTF_8));
    String readBack = IOUtils.toString(inputStream, StandardCharsets.UTF_8);
    
    // Build response
    Map<String, Object> response = new LinkedHashMap<>();
    response.put("trimmed", trimmed);
    response.put("randomToken", randomToken);
    response.put("containsOpenShift", containsOpenShift);
    response.put("readBack", readBack);
    response.put("httpsEnabled", true);
    response.put("timestamp", System.currentTimeMillis());
    response.put("status", "SUCCESS");
    
    return response;
  }
  
  @GetMapping("/api/health")
  public Map<String, String> health() {
    Map<String, String> health = new LinkedHashMap<>();
    health.put("status", "UP");
    health.put("service", "Spring Boot WAR with HTTPS");
    health.put("timestamp", String.valueOf(System.currentTimeMillis()));
    return health;
  }
}
```

### **3. `src/main/resources/application.properties`**
```properties
# Server Configuration
server.port=8443
server.servlet.context-path=/

# HTTPS Configuration
server.ssl.enabled=true
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-type=PKCS12
server.ssl.key-store-password=changeit
server.ssl.key-alias=myapp
server.ssl.key-password=changeit

# Application Configuration
spring.application.name=myapp-war-https
logging.level.com.example.demo=DEBUG

# Optional: HTTP to HTTPS redirect
# server.ssl.enabled-protocols=TLSv1.2,TLSv1.3
```

### **4. `pom.xml`** (Project root directory)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
  
  <modelVersion>4.0.0</modelVersion>
  
  <groupId>com.example</groupId>
  <artifactId>myapp</artifactId>
  <version>1.0.0</version>
  <packaging>war</packaging>
  
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.2</version>
    <relativePath/>
  </parent>
  
  <properties>
    <java.version>17</java.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
  
  <dependencies>
    <!-- Spring Boot Web -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- Tomcat provided for WAR deployment -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
      <scope>provided</scope>
    </dependency>
    
    <!-- System-scope dependencies -->
    <dependency>
      <groupId>commons-io</groupId>
      <artifactId>commons-io</artifactId>
      <version>2.16.1</version>
      <scope>system</scope>
      <systemPath>${project.basedir}/libs/commons-io-2.16.1.jar</systemPath>
    </dependency>
    
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
      <version>3.14.0</version>
      <scope>system</scope>
      <systemPath>${project.basedir}/libs/commons-lang3-3.14.0.jar</systemPath>
    </dependency>
  </dependencies>
  
  <build>
    <finalName>myapp</finalName>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
          <includeSystemScope>true</includeSystemScope>
          <mainClass>com.example.demo.DemoApplication</mainClass>
        </configuration>
        <executions>
          <execution>
            <goals>
              <goal>repackage</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
      
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-war-plugin</artifactId>
        <version>3.4.0</version>
        <configuration>
          <failOnMissingWebXml>false</failOnMissingWebXml>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

### **5. `Dockerfile`** (Project root directory)
```dockerfile
# Spring Boot WAR with HTTPS for OpenShift (s390x compatible)
FROM eclipse-temurin:17-jre

LABEL maintainer="dev-team@example.com"
LABEL version="1.0.0"
LABEL description="Spring Boot WAR with HTTPS and custom JARs"

# Working directory
WORKDIR /app

# Ensure arbitrary UID can write
RUN chmod -R g=u /app /etc

# Copy WAR
COPY target/myapp.war /app/app.war

# Copy keystore
COPY src/main/resources/keystore.p12 /etc/tls/keystore.p12

# JVM tmpdir for OpenShift
ENV JAVA_OPTS="-Djava.io.tmpdir=/tmp"

EXPOSE 8443

ENTRYPOINT ["sh","-c","java $JAVA_OPTS -jar /app/app.war"]

```

### **6. `scripts/generate-tls.sh`** (In scripts directory)
```bash
#!/bin/bash

echo "Generating TLS certificates..."
mkdir -p src/main/resources

# Generate private key
openssl genrsa -out src/main/resources/tls.key 2048

# Generate self-signed certificate
openssl req -x509 -new -nodes \
  -key src/main/resources/tls.key \
  -sha256 -days 365 \
  -subj "/C=US/ST=NY/L=NYC/O=MyOrg/OU=IT/CN=myapp.example.com" \
  -out src/main/resources/tls.crt

# Convert to PKCS12 keystore (Spring Boot compatible)
openssl pkcs12 -export \
  -in src/main/resources/tls.crt \
  -inkey src/main/resources/tls.key \
  -name myapp \
  -out src/main/resources/keystore.p12 \
  -passout pass:changeit

echo "TLS certificates generated:"
ls -la src/main/resources/*.p12 src/main/resources/*.key src/main/resources/*.crt
```

## 📋 **CREATION COMMANDS**

### **Step-by-Step File Creation**
```bash
# 1. Create project structure
mkdir -p springboot-war-tls/{src/main/{java/com/example/demo/api,resources},libs,scripts}
cd springboot-war-tls

# 2. Create Java files
cat > src/main/java/com/example/demo/DemoApplication.java << 'EOF'
[PASTE DemoApplication.java CODE HERE]
EOF

cat > src/main/java/com/example/demo/api/UtilController.java << 'EOF'
[PASTE UtilController.java CODE HERE]
EOF

# 3. Create application.properties
cat > src/main/resources/application.properties << 'EOF'
[PASTE application.properties CODE HERE]
EOF

# 4. Create pom.xml
cat > pom.xml << 'EOF'
[PASTE pom.xml CODE HERE]
EOF

# 5. Create Dockerfile
cat > Dockerfile << 'EOF'
[PASTE Dockerfile CODE HERE]
EOF

# 6. Create TLS generation script
cat > scripts/generate-tls.sh << 'EOF'
[PASTE generate-tls.sh CODE HERE]
EOF
chmod +x scripts/generate-tls.sh

# 7. Download JARs
curl -L -o libs/commons-io-2.16.1.jar \
  https://repo1.maven.org/maven2/commons-io/commons-io/2.16.1/commons-io-2.16.1.jar

curl -L -o libs/commons-lang3-3.14.0.jar \
  https://repo1.maven.org/maven2/org/apache/commons/commons-lang3/3.14.0/commons-lang3-3.14.0.jar

# 8. Generate TLS certificates
./scripts/generate-tls.sh
```

## 📊 **FINAL FILE STRUCTURE**
```
springboot-war-tls/
├── pom.xml
├── Dockerfile
├── libs/
│   ├── commons-io-2.16.1.jar
│   └── commons-lang3-3.14.0.jar
├── scripts/
│   └── generate-tls.sh
├── src/
│   └── main/
│       ├── java/
│       │   └── com/
│       │       └── example/
│       │           └── demo/
│       │               ├── DemoApplication.java          ← Spring Boot main class
│       │               └── api/
│       │                   └── UtilController.java       ← REST controller
│       └── resources/
│           ├── application.properties                    ← HTTPS config
│           ├── keystore.p12                             ← TLS keystore
│           ├── tls.crt                                  ← Certificate
│           └── tls.key                                  ← Private key
└── target/                                              ← After build
    └── myapp.war
```

## ✅ **VERIFICATION COMMANDS**

```bash
# 1. Verify Java files exist
find src/main/java -type f -name "*.java"

# 2. Verify package structure
head -5 src/main/java/com/example/demo/DemoApplication.java
head -5 src/main/java/com/example/demo/api/UtilController.java

# 3. Build the project
mvn clean compile

# 4. Verify compilation (no errors should appear)
echo $?
```

## 🎯 **Key Points**

1. **DemoApplication.java** → `src/main/java/com/example/demo/`
   - Main Spring Boot class
   - Extends `SpringBootServletInitializer` for WAR deployment

2. **UtilController.java** → `src/main/java/com/example/demo/api/`
   - REST controller with API endpoints
   - Demonstrates both commons-io and commons-lang3 usage

3. **application.properties** → `src/main/resources/`
   - HTTPS configuration on port 8443
   - TLS keystore configuration

4. **keystore.p12, tls.crt, tls.key** → `src/main/resources/`
   - Generated by `generate-tls.sh` script
   - Used for HTTPS encryption

5. **pom.xml** → Project root
   - WAR packaging
   - System-scope dependencies pointing to `libs/` directory

6. **Dockerfile** → Project root
   - Container configuration with non-root user
   - Copies WAR and keystore

7. **libs/** → Project root
   - Contains downloaded JAR files
   - Referenced in pom.xml via system scope


### **Step 10: Test Locally with HTTPS**
```bash
# Run as executable WAR
java -jar target/myapp.war

# In another terminal, test with curl
curl -k https://localhost:8443/api/health
curl -k https://localhost:8443/api/util

# Expected response:
# {
#   "trimmed": "Hello OpenShift with HTTPS",
#   "randomToken": "aB3cDeFgHiJkLmN0",
#   "containsOpenShift": true,
#   "readBack": "commons-io stream operations working with HTTPS",
#   "httpsEnabled": true,
#   "timestamp": 1706812345678,
#   "status": "SUCCESS"
# }
```

### **Step 12: Build and Test Docker Image**
```bash
# Build image
podman build -t myapp-war-https:1.0 .

# Run container
podman run --name myapp-https -p 8443:8443 -d localhost/myapp-war-https:1.0

# Check logs
podman logs -f myapp-https

# Test HTTPS endpoint
curl -k https://localhost:8443/api/util

# Stop container
podman stop myapp-https && podman rm myapp-https
```

---

## ☸️ OpenShift Deployment

### **Step 13: Prepare for OpenShift**
```bash
# Login to OpenShift
oc login --token=<your-token> --server=<openshift-api-url>

# Create project
oc new-project springboot-war-demo --display-name="Spring Boot WAR HTTPS Demo"
```

### **Step 14: Create TLS Secret**
```bash
# Create secret from keystore
oc create secret generic myapp-tls-secret \
  --from-file=keystore.p12=src/main/resources/keystore.p12 \
  --type=Opaque

# Verify secret
oc get secret myapp-tls-secret -o yaml
```

### **Step 15: Tag and Push Image**
```bash
# Tag for OpenShift internal registry
podman tag myapp-war-https:1.0 \
  image-registry.openshift-image-registry.svc:5000/springboot-war-demo/myapp-war-https:1.0

# Login to registry
podman login -u $(oc whoami) -p $(oc whoami -t) \
  image-registry.openshift-image-registry.svc:5000

# Push image
podman push \
  image-registry.openshift-image-registry.svc:5000/springboot-war-demo/myapp-war-https:1.0
```

### **Step 16: Deploy Application**
```yaml
# Create deployment.yaml
cat > deployment.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-war-https
  namespace: springboot-war-demo
  labels:
    app: myapp-war-https
    component: springboot
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp-war-https
  template:
    metadata:
      labels:
        app: myapp-war-https
    spec:
      containers:
      - name: myapp
        image: image-registry.openshift-image-registry.svc:5000/springboot-war-demo/myapp-war-https:1.0
        ports:
        - containerPort: 8443
          name: https
        env:
        - name: JAVA_OPTS
          value: "-Xmx512m -Xms256m -Djava.io.tmpdir=/tmp"
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        volumeMounts:
        - name: tls-secret
          mountPath: /etc/tls
          readOnly: true
        readinessProbe:
          httpGet:
            path: /api/health
            port: 8443
            scheme: HTTPS
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
        livenessProbe:
          httpGet:
            path: /api/health
            port: 8443
            scheme: HTTPS
          initialDelaySeconds: 45
          periodSeconds: 15
      volumes:
      - name: tls-secret
        secret:
          secretName: myapp-tls-secret
      securityContext:
        runAsUser: 1001
        fsGroup: 1001
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-war-https-service
  namespace: springboot-war-demo
spec:
  selector:
    app: myapp-war-https
  ports:
  - port: 8443
    targetPort: 8443
    protocol: TCP
    name: https
  type: ClusterIP
EOF

# Apply deployment
oc apply -f deployment.yaml
```

### **Step 17: Create HTTPS Route**
```bash
# Create passthrough route for HTTPS
oc create route passthrough myapp-https-route \
  --service=myapp-war-https-service \
  --port=https \
  --hostname=myapp-war-https.apps.ocp.example.com

# Get route URL
oc get route myapp-https-route -o jsonpath='{.spec.host}'
```

### **Step 18: Configure Auto-scaling (Optional)**
```yaml
# Create hpa.yaml
cat > hpa.yaml << EOF
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-war-hpa
  namespace: springboot-war-demo
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-war-https
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
EOF

oc apply -f hpa.yaml
```

---

## 🧪 Testing

### **Step 19: Test OpenShift Deployment**
```bash
# Get route URL
ROUTE_URL=$(oc get route myapp-https-route -o jsonpath='{.spec.host}')

# Test health endpoint
curl -k https://${ROUTE_URL}/api/health

# Test main endpoint
curl -k https://${ROUTE_URL}/api/util

# Watch pods
oc get pods -w

# Check logs
oc logs -f deployment/myapp-war-https

# Monitor resource usage
oc adm top pods -n springboot-war-demo
```

### **Step 20: Load Testing (Optional)**
```bash
# Install hey load testing tool
wget https://hey-release.s3.us-east-2.amazonaws.com/hey_linux_amd64 -O hey
chmod +x hey

# Run load test
./hey -n 1000 -c 50 -k https://${ROUTE_URL}/api/util
```

---

## 🔧 Troubleshooting

### **Common Issues and Solutions**

1. **TLS/SSL Handshake Errors**
   ```bash
   # Verify keystore exists in container
   oc exec deployment/myapp-war-https -- ls -la /etc/tls/
   
   # Check Java version
   oc exec deployment/myapp-war-https -- java -version
   
   # Verify keystore password
   oc exec deployment/myapp-war-https -- \
     keytool -list -keystore /etc/tls/keystore.p12 -storepass changeit
   ```

2. **System-Scope JARs Not Included**
   ```bash
   # Check WAR contents
   jar tf target/myapp.war | grep commons
   
   # Verify includeSystemScope in pom.xml
   grep -A2 -B2 includeSystemScope pom.xml
   ```

3. **OpenShift Permission Issues**
   ```bash
   # Check security context constraints
   oc describe scc restricted
   
   # Fix permission issues
   oc adm policy add-scc-to-user anyuid -z default -n springboot-war-demo
   ```

4. **Application Won't Start**
   ```bash
   # Get detailed logs
   oc logs deployment/myapp-war-https --previous
   
   # Check events
   oc get events --sort-by='.lastTimestamp'
   
   # Check resource limits
   oc describe pod <pod-name>
   ```

5. **Route Not Working**
   ```bash
   # Check route status
   oc describe route myapp-https-route
   
   # Test service directly
   oc port-forward service/myapp-war-https-service 8443:8443
   curl -k https://localhost:8443/api/health
   ```

### **Debug Commands**
```bash
# Comprehensive health check
oc get all -n springboot-war-demo
oc get routes,svc,deploy,pods -n springboot-war-demo
oc describe deploy/myapp-war-https
oc logs deployment/myapp-war-https --tail=100

# Shell into container
oc exec -it deployment/myapp-war-https -- /bin/sh
# Inside container:
#   ps aux
#   netstat -tulpn
#   cat /proc/1/environ
```

---

## 📊 Verification Checklist

- [ ] TLS certificates generated successfully
- [ ] System-scope JARs included in WAR (`WEB-INF/lib/`)
- [ ] WAR builds without errors
- [ ] HTTPS works locally (`curl -k https://localhost:8443/api/util`)
- [ ] Docker image builds and runs
- [ ] OpenShift project created
- [ ] TLS secret created
- [ ] Image pushed to registry
- [ ] Deployment created with 2 replicas
- [ ] Pods are running
- [ ] Service created
- [ ] HTTPS route created
- [ ] Application accessible via route
- [ ] Health checks passing
- [ ] Both JARs functional in OpenShift

---

## 🎯 Expected Final Output

### **API Response**
```json
{
  "trimmed": "Hello OpenShift with HTTPS",
  "randomToken": "a1B2c3D4e5F6g7H8",
  "containsOpenShift": true,
  "readBack": "commons-io stream operations working with HTTPS",
  "httpsEnabled": true,
  "timestamp": 1706812345678,
  "status": "SUCCESS"
}
```

### **Health Endpoint**
```json
{
  "status": "UP",
  "service": "Spring Boot WAR with HTTPS",
  "timestamp": "1706812345678"
}
```

---

## 📝 Summary

This implementation provides:
- ✅ Spring Boot 3.2.2 as WAR
- ✅ Custom system-scope JAR dependencies (commons-io, commons-lang3)
- ✅ HTTPS/TLS with OpenSSL certificates
- ✅ Docker container with non-root user
- ✅ OpenShift deployment with TLS passthrough
- ✅ Health checks and readiness probes
- ✅ Horizontal auto-scaling
- ✅ Complete end-to-end pipeline

The application demonstrates a production-ready Spring Boot WAR deployment with custom dependencies and HTTPS security, fully compatible with OpenShift enterprise environments.
