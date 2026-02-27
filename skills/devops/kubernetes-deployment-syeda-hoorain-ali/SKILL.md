---
name: kubernetes-deployment
description: Comprehensive Kubernetes deployment and scaling for containerized applications from hello world to professional production systems. Use this skill when deploying, managing, and scaling containerized applications on Kubernetes clusters, particularly with Minikube for local development and testing.
---

# Kubernetes Deployment and Scaling Skill

## Purpose
This skill provides guidance for deploying and scaling containerized applications on Kubernetes from basic concepts to professional production systems. It focuses on practical workflows for developers, particularly for local development using Minikube. The skill covers complete deployment lifecycle from initial setup to production-grade configurations.

## When to Use This Skill
Use this skill when:
- Deploying containerized applications to Kubernetes clusters
- Setting up local Kubernetes environments with Minikube
- Creating Helm charts for application deployment
- Scaling applications on Kubernetes
- Managing application lifecycle on Kubernetes
- Troubleshooting Kubernetes deployments
- Implementing service discovery and networking in Kubernetes
- Setting up persistent storage for applications
- Configuring Ingress for external access

## Prerequisites
- Docker containers of your applications
- Basic understanding of containerization concepts
- kubectl installed and configured
- Helm installed for chart management
- Minikube or other Kubernetes cluster available

## Core Workflows

### 1. Setting up Minikube Environment
When starting with local Kubernetes development:

```bash
# Start Minikube with sufficient resources
minikube start --cpus=4 --memory=8192 --disk-size=20g

# Verify cluster status
minikube status

# Enable necessary addons
minikube addons enable ingress
minikube addons enable metrics-server
minikube addons enable default-storageclass
minikube addons enable storage-provisioner
```

### 2. Preparing Container Images for Minikube
Before deploying applications to Minikube:

```bash
# For local images, load them into Minikube's Docker environment
eval $(minikube docker-env)
docker build -t your-app:latest .
# Or load existing images
minikube image load your-existing-image:tag
```

### 3. Basic Deployment Process
For deploying a simple application to Kubernetes:

1. **Verify your container images are available**
   - Ensure your Docker images are built and accessible in the Kubernetes cluster
   - For local development, use `minikube image load <image-name>` or `eval $(minikube docker-env)` before building

2. **Create Kubernetes deployment YAML**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: app-name
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: app-name
     template:
       metadata:
         labels:
           app: app-name
       spec:
         containers:
         - name: app-container
           image: your-image:tag
           ports:
           - containerPort: 8080
             name: http
             protocol: TCP
           env:
           - name: ENV_VAR
             value: "value"
           resources:
             requests:
               memory: "64Mi"
               cpu: "250m"
             limits:
               memory: "128Mi"
               cpu: "500m"
           livenessProbe:
             httpGet:
               path: /health
               port: http
             initialDelaySeconds: 30
             periodSeconds: 10
           readinessProbe:
             httpGet:
               path: /ready
               port: http
             initialDelaySeconds: 5
             periodSeconds: 5
   ```

3. **Create Service to expose the deployment**
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: app-service
   spec:
     selector:
       app: app-name
     ports:
       - name: http
         protocol: TCP
         port: 80
         targetPort: 8080
     type: ClusterIP  # Use NodePort for local access
   ```

4. **Apply the configuration**
   ```bash
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   ```

5. **Verify deployment**
   ```bash
   kubectl get deployments
   kubectl get pods
   kubectl get services
   ```

### 4. Using Helm Charts for Professional Deployments
For more complex or production-ready deployments:

1. **Initialize Helm chart structure**
   ```bash
   helm create app-chart
   ```

2. **Customize the chart templates**
   - Edit `templates/deployment.yaml`, `templates/service.yaml`, `templates/ingress.yaml`, etc.
   - Update `values.yaml` with configurable parameters
   - Add `templates/pvc.yaml` for persistent storage if needed
   - Add `templates/secret.yaml` for sensitive configuration

3. **Install the Helm chart**
   ```bash
   helm install app-release ./app-chart --namespace app-namespace --create-namespace
   ```

4. **Upgrade and manage releases**
   ```bash
   helm upgrade app-release ./app-chart
   helm list -n app-namespace
   helm uninstall app-release -n app-namespace
   ```

### 5. Service Discovery and Networking
For applications with multiple interconnected services:

1. **Use Kubernetes DNS names for inter-service communication**
   - Format: `http://<service-name>.<namespace>.svc.cluster.local:<port>`
   - For same namespace: `http://<service-name>:<port>`
   - Example: Backend connecting to MCP server: `http://mcp-server-service:8080`

2. **Configure NetworkPolicies for security**
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: app-network-policy
   spec:
     podSelector:
       matchLabels:
         app: app-name
     policyTypes:
       - Ingress
       - Egress
     ingress:
       - from:
         - podSelector:
             matchLabels:
               app: other-app
         ports:
         - protocol: TCP
           port: 8080
   ```

### 6. Persistent Storage Configuration
For data that needs to persist across pod restarts:

1. **Create PersistentVolumeClaim**
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: app-storage
   spec:
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 10Gi
     storageClassName: standard  # Use "" for default, or specific class
   ```

2. **Use in deployment**
   ```yaml
   spec:
     template:
       spec:
         containers:
         - name: app-container
           volumeMounts:
           - name: app-storage
             mountPath: /data
         volumes:
         - name: app-storage
           persistentVolumeClaim:
             claimName: app-storage
   ```

### 7. Working with Multi-Service Applications (Taskflow Example)
For the specific Taskflow application with frontend, backend, and MCP server:

1. **Create a Helm chart with multiple components**
   - Frontend: Next.js application
   - Backend: Python FastAPI application
   - MCP Server: Model Context Protocol server
   - PostgreSQL: Database with persistent storage

2. **Define inter-service communication**
   - Backend connects to MCP server using service name: `http://{{ include "app.fullname" . }}-mcp-server-service:8080`
   - Backend connects to PostgreSQL using service name: `postgresql://user:pass@{{ include "app.fullname" . }}-postgres-service:5432/dbname`
   - Frontend connects to backend using service name: `http://{{ include "app.fullname" . }}-backend-service:80`

3. **Configure persistent storage for data persistence**
   - Use StatefulSets for PostgreSQL with PersistentVolumeClaims
   - Configure appropriate storage classes for Minikube

4. **Environment variables in deployments**
   ```yaml
   # In backend deployment
   env:
     - name: MCP_SERVER_URL
       value: "http://{{ include "app.fullname" . }}-mcp-server-service:8080"
     - name: DATABASE_URL
       value: "postgresql://postgres:password@{{ include "app.fullname" . }}-postgres-service:5432/task_db"
   ```

### 8. Ingress Configuration for External Access
For exposing applications to external traffic:

1. **Create Ingress resource**
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: app-ingress
     annotations:
       kubernetes.io/ingress.class: "nginx"
       nginx.ingress.kubernetes.io/rewrite-target: /
   spec:
     rules:
     - host: app.local
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: frontend-service
               port:
                 number: 80
   ```

2. **Access the application**
   - Add to hosts file: `<minikube-ip> app.local`
   - Access via: `http://app.local`

### 9. Scaling Applications
For scaling applications based on demand:

1. **Horizontal Pod Autoscaling (HPA)**
   ```yaml
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
     name: app-hpa
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: app-name
     minReplicas: 1
     maxReplicas: 10
     metrics:
     - type: Resource
       resource:
         name: cpu
         target:
           type: Utilization
           averageUtilization: 70
   ```

2. **Manual scaling**
   ```bash
   kubectl scale deployment app-name --replicas=3
   ```

## Best Practices

### Security
- Use resource requests and limits for all containers
- Implement liveness and readiness probes
- Use non-root users in containers when possible
- Apply appropriate RBAC policies
- Store sensitive data in Secrets, not ConfigMaps
- Use NetworkPolicies to restrict unnecessary network traffic

### Configuration Management
- Use ConfigMaps for non-sensitive configuration
- Use Secrets for sensitive information (API keys, passwords)
- Parameterize configurations using Helm values
- Use Helm template functions for dynamic naming: `{{ include "app.fullname" . }}`
- Follow consistent naming conventions

### Monitoring and Observability
- Include health check endpoints in applications
- Use Kubernetes native logging (kubectl logs)
- Implement proper labeling for resource identification
- Use readiness/liveness probes for service availability
- Include resource utilization metrics

### Production Readiness
- Use PersistentVolumeClaims for stateful applications
- Implement proper backup strategies for persistent data
- Use StatefulSets for applications requiring stable identities
- Configure appropriate resource limits to prevent resource exhaustion
- Implement proper error handling and graceful degradation

## Troubleshooting Common Issues

### Pod fails to start
- Check image name and tag are correct
- Verify image is accessible (for Minikube, ensure it's loaded in the correct environment)
- Check resource requests aren't too high for the cluster
- Use `kubectl describe pod <pod-name>` to see detailed events
- Check logs with `kubectl logs <pod-name> --previous` for crashed containers

### Service not accessible
- Verify service selector matches pod labels
- Check if the application is listening on the correct port
- For external access, ensure Ingress or NodePort is properly configured
- Verify DNS resolution with `kubectl exec` into a pod and test connectivity

### Scaling issues
- Ensure metrics-server is running for HPA
- Verify application can handle multiple instances
- Check for any shared state that might prevent scaling
- Monitor resource utilization to ensure appropriate scaling triggers

### Persistent storage issues
- Check PVC status with `kubectl get pvc`
- Verify storage class exists and is available
- Ensure proper permissions for storage access
- Check if PV/PVC binding is successful

### Secret/Configuration issues
- Verify secret keys match what's referenced in deployments
- Check base64 encoding for secrets
- Use `kubectl get secrets <name> -o yaml` to verify content
- Ensure proper permissions for reading secrets

## References
- Kubernetes official documentation
- Helm charts best practices
- Minikube documentation
- Production Kubernetes deployment patterns
- Service discovery in Kubernetes
- Persistent storage best practices
