# Kubernetes Microservices Deployment

## Project Overview

This project demonstrates a cloud-native microservices application deployed on a local Kubernetes cluster using **Minikube, Docker, Kubernetes Deployments, Services, Horizontal Pod Autoscaling (HPA), and persistent storage**.

The implementation simulates a production-like microservices environment for ABVC Solutions, focusing on containerization, service discovery, scalability, external access, and persistent application data.

## Objectives

- Set up and configure a Kubernetes cluster using Minikube.
- Containerize independent microservices with Docker.
- Deploy microservices using Kubernetes Deployments.
- Implement Kubernetes Service discovery and load balancing.
- Configure Horizontal Pod Autoscaling.
- Implement persistent storage using Kubernetes PV/PVC.
- Demonstrate data persistence after a pod restart.
- Expose a microservice externally using NodePort.

## Architecture

```text
                         Minikube Kubernetes Cluster
                                  |
             +--------------------+--------------------+
             |                    |                    |
             v                    v                    v
       User Service         Product Service       Order Service
        ClusterIP              NodePort              ClusterIP
             |                    |                    |
          2 Pods              2-5 Pods               1 Pod
                                  |
                                 HPA
                           CPU Target: 50%
                                  |
                           Scale: 2 -> 5
                                                        |
                                                       PVC
                                                        |
                                                        v
                                                       PV
                                                        |
                                                Persistent Data
```

## Technologies Used

- Kubernetes
- Minikube
- kubectl
- Docker
- Python / Flask
- Kubernetes YAML
- HPA
- Persistent Volumes and Persistent Volume Claims
- Git / GitHub

## Microservices

### User Service
- Flask microservice
- Port 5000
- 2 replicas
- ClusterIP
- Endpoints: `/`, `/users`, `/health`

### Product Service
- Flask microservice
- Port 5000
- 2 initial replicas
- HPA: minimum 2, maximum 5
- CPU target: 50%
- NodePort for external access
- Endpoints: `/`, `/products`, `/health`, `/cpu`

The `/cpu` endpoint was used to generate controlled CPU load for the HPA demonstration.

### Order Service
- Flask microservice
- Port 5000
- 1 replica
- ClusterIP
- Persistent storage mounted at `/data`
- Endpoints: `/`, `/orders`, `/health`

## Project Structure

```text
kubernetes-microservices-deployment/
├── services/
│   ├── user-service/
│   ├── product-service/
│   └── order-service/
├── k8s/
│   ├── user-deployment.yaml
│   ├── user-service.yaml
│   ├── product-deployment.yaml
│   ├── product-service.yaml
│   ├── product-hpa.yaml
│   ├── order-deployment.yaml
│   ├── order-service.yaml
│   ├── order-pv.yaml
│   ├── order-pvc.yaml
│   └── load-generator.yaml
├── screenshots/
└── README.md
```

## 1. Kubernetes Cluster Setup

The Minikube cluster was created using Docker:

```bash
minikube start --driver=docker
```

Cluster verification:

```bash
kubectl get nodes
```

![Kubernetes Node](screenshots/05-kubernetes-node.png)

## 2. Containerization

Each microservice has its own Dockerfile and image.

```bash
docker build -t user-service:1.0 ./services/user-service
docker build -t product-service:1.0 ./services/product-service
docker build -t order-service:1.0 ./services/order-service
```

## 3. Kubernetes Deployments

Three Deployments were created:

```bash
kubectl get deployments -n microservices
```

![Deployments](screenshots/07-deployments.png)

| Service | Replicas | Purpose |
|---|---:|---|
| User Service | 2 | User operations |
| Product Service | 2 initial | Product operations + HPA |
| Order Service | 1 | Orders + persistent storage |

## 4. Kubernetes Services and Service Discovery

| Service | Type | Purpose |
|---|---|---|
| user-service | ClusterIP | Internal communication |
| product-service | NodePort | Internal + external access |
| order-service | ClusterIP | Internal communication |

![Services](screenshots/08-services.png)

Service discovery was verified using Kubernetes DNS names such as:

```bash
curl http://user-service
curl http://product-service/products
curl http://order-service/orders
```

## 5. Horizontal Pod Autoscaling

Product Service was configured with:

```yaml
minReplicas: 2
maxReplicas: 5
averageUtilization: 50
```

Artificial CPU load was generated against Product Service. Kubernetes successfully scaled Product Service from **2 replicas to 5 replicas**.

![HPA Scaling Timeline](screenshots/01-hpa-scaling-timeline.png)

![Pod Scaling Timeline](screenshots/02-pod-scaling-timeline.png)

![HPA Maximum Replicas](screenshots/03-hpa-max-replicas.png)

The HPA was verified using:

```bash
kubectl get hpa -n microservices
```

Observed behavior:

```text
CPU utilization: 14% -> 252%+
Replicas:         2  -> 4 -> 5
```

## 6. Persistent Storage

Persistent storage was configured for the Order Service using Kubernetes PV/PVC resources.

```bash
kubectl get pv
kubectl get pvc -n microservices
```

![PV and PVC](screenshots/10-pv-pvc.png)

The Order Service mounts storage at:

```text
/data
```

and stores order information in:

```text
/data/orders.txt
```

The PVC successfully bound to a persistent volume provided by the Minikube storage provisioner.

## 7. Persistence Test

An order was stored and retrieved through the Order Service:

```bash
curl -X POST http://order-service/orders   -H "Content-Type: application/json"   -d '{"order":"ORD-001-Abbas"}'

curl http://order-service/orders
```

The Order Service pod was then deleted and recreated by its Deployment. The order data remained available after the pod restart, demonstrating persistent storage.

![Persistence Test](screenshots/09-persistence-test.png)

## 8. External Access Using NodePort

Product Service was exposed using NodePort:

```text
product-service -> NodePort 30774
```

External access was tested successfully through the Minikube service tunnel.

```bash
curl http://127.0.0.1:<forwarded-port>
curl http://127.0.0.1:<forwarded-port>/products
```

![NodePort External Access](screenshots/11-nodeport-external-access.png)

## 9. Final Cluster State

The final environment included:

- Minikube node in `Ready` state
- User Service: 2/2 running
- Product Service: 2/2 running after HPA scale-down
- Order Service: 1/1 running
- HPA configured for 2–5 replicas
- Order PVC in `Bound` state
- Product Service exposed through NodePort

![Final Pods](screenshots/06-final-pods.png)

## 10. Challenges and Solutions

### Docker and Minikube
Local Docker images were made available to Minikube with:

```bash
eval $(minikube docker-env)
```

### Git Bash Path Conversion
A load-generator command initially converted `/bin/sh` into a Windows Git Bash path.

**Solution:** The load generator was defined in Kubernetes YAML with the container command specified explicitly.

### HPA Metrics
HPA initially showed `<unknown>/50%`.

**Solution:** Waited for Metrics Server to collect CPU metrics. The HPA then reported live utilization.

### HPA Load Testing
A single load generator was insufficient to create sustained CPU pressure.

**Solution:** Multiple load-generator pods were used. Product Service scaled from 2 to 5 replicas.

### Persistent Storage
The PVC was successfully bound and used by Order Service. Minikube's standard storage provisioner supplied the bound volume.

## 11. Verification Commands

```bash
kubectl get nodes
kubectl get pods -n microservices
kubectl get deployments -n microservices
kubectl get services -n microservices
kubectl get hpa -n microservices
kubectl get pv
kubectl get pvc -n microservices
kubectl get all -n microservices
```

## 12. Conclusion

This project demonstrates a complete Kubernetes-based microservices deployment with Docker containerization, service discovery, external access, automatic scaling, and persistent application data.

The implementation successfully demonstrated:

- Microservice containerization
- Kubernetes Deployments
- ClusterIP service discovery
- NodePort external access
- HPA scaling from 2 to 5 replicas
- Persistent storage using PVC/PV
- Data survival after pod recreation

This project provides practical experience with core Kubernetes concepts used in cloud-native DevOps environments.

