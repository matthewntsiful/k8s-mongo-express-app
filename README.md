# Kubernetes MongoDB Express Application

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-%234ea94b.svg?style=for-the-badge&logo=mongodb&logoColor=white)
![Mongo Express](https://img.shields.io/badge/Mongo%20Express-%23ff6b35.svg?style=for-the-badge&logo=mongodb&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![YAML](https://img.shields.io/badge/yaml-%23ffffff.svg?style=for-the-badge&logo=yaml&logoColor=151515)
![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)

A production-ready Kubernetes deployment of MongoDB with Mongo Express web interface, featuring secure configuration management, resource allocation, and ingress routing.

## 🖼️ Application Preview

![MongoDB Express Web Interface](./assets/mongo-express-screenshot.png)
*MongoDB Express web interface running on Kubernetes*

## 🏗️ Architecture Overview

```mermaid
graph TD
    subgraph Kubernetes Cluster
        subgraph mongo-db[Namespace: mongo-db]
            subgraph MongoDB
                mdb1[(MongoDB Pod 1)]
                mdb2[(MongoDB Pod 2)]
                mdbSvc[MongoDB Service\n27017]
                mdb1 --> mdbSvc
                mdb2 --> mdbSvc
            end
            
            subgraph Mongo Express
                me1[Mongo Express Pod 1]
                me2[Mongo Express Pod 2]
                meSvc[Mongo Express Service\n8081 (NodePort: 32000)]
                me1 --> meSvc
                me2 --> meSvc
            end
            
            subgraph Configuration
                secret[Secret\n- Credentials\n- Passwords]
                config[ConfigMap\n- Database URL\n- Auth Settings]
            end
            
            me1 -->|Connects to| mdbSvc
            me2 -->|Connects to| mdbSvc
            
            me1 -->|Uses| config
            me1 -->|Uses| secret
            me2 -->|Uses| config
            me2 -->|Uses| secret
        end
        
        ingress[NGINX Ingress\nmongodb.local] --> meSvc
    end
    
    User[User] -->|Accesses| ingress
    
    style mongo-db fill:#f9f9f9,stroke:#333,stroke-width:2px
    style MongoDB fill:#e1f5fe,stroke:#039be5
    style Mongo Express fill:#e8f5e9,stroke:#43a047
    style Configuration fill:#f3e5f5,stroke:#8e24aa
    style ingress fill:#fff3e0,stroke:#fb8c00
```

### Why This Deployment Matters in Production

In enterprise environments, database management requires robust, scalable, and secure solutions. This Kubernetes deployment addresses critical production needs:

- **High Availability**: Multi-replica deployments ensure zero-downtime database operations
- **Resource Isolation**: Dedicated namespace and resource limits prevent resource contention
- **Security**: Encrypted secrets management and network isolation protect sensitive data
- **Controlled Database Access**: GUI-based interaction via Mongo Express prevents direct database access and unauthorized operations
- **Access Layer Protection**: Web interface acts as a secure gateway, eliminating need for direct MongoDB client connections
- **Scalability**: Horizontal pod autoscaling handles varying workloads efficiently
- **Observability**: Centralized logging and monitoring enable proactive issue resolution
- **DevOps Integration**: Infrastructure-as-code approach enables CI/CD pipeline integration

### Architecture Components

This project demonstrates a complete Kubernetes deployment stack including:

- **MongoDB Database**: Scalable NoSQL database with persistent storage
- **Mongo Express**: Web-based MongoDB administration interface
- **Kubernetes Resources**: Comprehensive K8s manifests with security best practices
- **Ingress Controller**: NGINX-based routing for external access
- **Resource Management**: CPU and memory allocation for optimal performance

## 📋 Prerequisites

- Kubernetes cluster (v1.20+)
- kubectl configured and connected to your cluster
- NGINX Ingress Controller installed
- Docker runtime environment

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone <repository-url>
cd k8s-mongo-express-app
```

### 2. Deploy to Kubernetes

```bash
# Apply all manifests in order
kubectl apply -f manifests/

# Or apply individually
kubectl apply -f manifests/01-namespace.yaml
kubectl apply -f manifests/02-secret.yaml
kubectl apply -f manifests/03-configmap.yaml
kubectl apply -f manifests/04-mongodb.yaml
kubectl apply -f manifests/05-mongo-express.yaml
kubectl apply -f manifests/06-ingress.yaml
```

### 3. Verify Deployment

```bash
# Check all resources in mongo-db namespace
kubectl get all -n mongo-db

# Check pod status
kubectl get pods -n mongo-db

# Check services
kubectl get svc -n mongo-db
```

### 4. Access the Application

```bash
# Add to /etc/hosts (Linux/macOS) or C:\Windows\System32\drivers\etc\hosts (Windows)
echo "127.0.0.1 mongodb.local" >> /etc/hosts

# Access via browser
http://mongodb.local
```

## 📁 Project Structure

```text
k8s-mongo-express-app/
├── manifests/
│   ├── 01-namespace.yaml      # Kubernetes namespace
│   ├── 02-secret.yaml         # MongoDB credentials
│   ├── 03-configmap.yaml      # Application configuration
│   ├── 04-mongodb.yaml        # MongoDB deployment & service
│   ├── 05-mongo-express.yaml  # Mongo Express deployment & service
│   └── 06-ingress.yaml        # Ingress routing configuration
├── assets/                    # Documentation assets
├── README.md                  # Project documentation
└── LICENSE                    # License file
```

## 🔧 Configuration Details

### Resource Allocation

| Component | CPU Request | Memory Request | CPU Limit | Memory Limit |
|-----------|-------------|----------------|-----------|--------------|
| MongoDB | 250m | 256Mi | 500m | 512Mi |
| Mongo Express | 100m | 128Mi | 200m | 256Mi |

### Default Credentials

| Service | Username | Password |
|---------|----------|----------|
| MongoDB | root | passwd |
| Mongo Express | admin | pass |

> ⚠️ **Security Notice**: Change default credentials before production deployment

### Network Configuration

- **MongoDB Service**: ClusterIP on port 27017
- **Mongo Express Service**: LoadBalancer on port 8081 (NodePort: 32000)
- **Ingress**: HTTP access via `mongodb.local`

## 🛠️ Customization

### Scaling Replicas

```bash
# Scale MongoDB
kubectl scale deployment mongodb-deploy -n mongo-db --replicas=3

# Scale Mongo Express
kubectl scale deployment mongo-express-deploy -n mongo-db --replicas=3
```

### Updating Credentials

```bash
# Create new secret with custom credentials
kubectl create secret generic mongo-secret \
  --from-literal=mongo-root-username=<your-username> \
  --from-literal=mongo-root-password=<your-password> \
  -n mongo-db --dry-run=client -o yaml | kubectl apply -f -
```

### Resource Adjustment

Edit the `resources` section in deployment manifests:

```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1000m"
```

## 🔍 Monitoring & Troubleshooting

### Check Pod Logs

```bash
# MongoDB logs
kubectl logs -f deployment/mongodb-deploy -n mongo-db

# Mongo Express logs
kubectl logs -f deployment/mongo-express-deploy -n mongo-db
```

### Debug Connection Issues

```bash
# Test MongoDB connectivity
kubectl exec -it deployment/mongodb-deploy -n mongo-db -- mongo --eval "db.adminCommand('ismaster')"

# Check service endpoints
kubectl get endpoints -n mongo-db
```

### Common Issues

1. **Pods not starting**: Check resource availability and image pull status
2. **Connection refused**: Verify service names and port configurations
3. **Ingress not working**: Ensure NGINX Ingress Controller is installed

## 🔒 Security Best Practices

- ✅ Secrets stored in Kubernetes Secret objects
- ✅ Base64 encoded sensitive data
- ✅ Namespace isolation
- ✅ Resource limits configured
- ✅ Non-root container execution
- ⚠️ Default credentials (change for production)
- ⚠️ HTTP ingress (consider HTTPS for production)

## 📈 Production Considerations

### High Availability

- Deploy across multiple nodes
- Use persistent volumes for MongoDB data
- Implement backup strategies
- Configure health checks

### Security Hardening

- Enable TLS/SSL encryption
- Use custom certificates
- Implement network policies
- Regular security updates

### Performance Optimization

- Adjust resource limits based on workload
- Configure MongoDB replica sets
- Implement caching strategies
- Monitor resource usage

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- MongoDB team for the excellent database
- Mongo Express contributors for the web interface
- Kubernetes community for the orchestration platform
- NGINX team for the ingress controller

---

## Built with ❤️ for the Kubernetes community
