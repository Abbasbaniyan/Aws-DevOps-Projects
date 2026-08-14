☁️ Kubernetes Monitoring on AWS EKS



📌 Project Overview

This project implements a production-style Kubernetes monitoring stack on Amazon EKS using Terraform, Helm, Prometheus, Grafana, Metrics Server, and the AWS EBS CSI Driver.

The infrastructure is provisioned as code with Terraform, while Prometheus and Grafana are deployed into the EKS cluster using Helm.

The final setup provides:

Kubernetes cluster monitoring

Node CPU and memory monitoring

Pod-level resource monitoring

Kubernetes system component monitoring

Persistent storage for Prometheus and Alertmanager

Grafana dashboards for visualization

Kubernetes Metrics API through Metrics Server

AWS EBS-backed persistent volumes using gp3

🏗️ Architecture

                         AWS Cloud
                            │
                    ┌───────▼────────┐
                    │   VPC 10.0.0.0/16
                    │  2 AZs / NAT GW │
                    └───────┬────────┘
                            │
                    ┌───────▼────────┐
                    │    EKS 1.33    │
                    │ monitoring-eks │
                    └───────┬────────┘
                            │
              ┌─────────────┴─────────────┐
              │                           │
      ┌───────▼────────┐          ┌───────▼────────┐
      │ Managed Nodes  │          │ EKS Add-ons    │
      │  t3.small      │          │ VPC CNI        │
      │  1–2 nodes     │          │ CoreDNS        │
      └───────┬────────┘          │ kube-proxy     │
              │                   │ EBS CSI        │
              │                   │ Pod Identity   │
              │                   └────────────────┘
              │
      ┌───────┴──────────────────────────────┐
      │              monitoring namespace    │
      │                                      │
      │  ┌─────────────┐    ┌─────────────┐ │
      │  │ Prometheus  │───▶│   Grafana   │ │
      │  │             │    │             │ │
      │  └──────┬──────┘    └─────────────┘ │
      │         │                            │
      │  ┌──────▼──────────┐                 │
      │  │ EBS gp3 CSI PVC │                 │
      │  └─────────────────┘                 │
      │                                      │
      │  Metrics Server → Kubernetes Metrics │
      └──────────────────────────────────────┘

🛠️ Technologies Used

Technology

Purpose

AWS VPC

Network infrastructure

Amazon EKS

Managed Kubernetes cluster

EC2 / EKS Managed Node Groups

Kubernetes worker nodes

Terraform

Infrastructure as Code

Helm

Kubernetes application deployment

Prometheus

Metrics collection and monitoring

Grafana

Metrics visualization and dashboards

Metrics Server

Kubernetes resource metrics API

AWS EBS CSI Driver

Persistent EBS storage for Kubernetes

gp3

EBS storage class

kubectl

Kubernetes administration

AWS CLI

AWS resource management

🚀 Infrastructure Deployment

1. Clone the Repository

git clone <YOUR-GITHUB-REPOSITORY-URL>
cd kubernetes-monitoring-terraform-helm

Move into the Terraform directory:

cd terraform

2. Initialize Terraform

terraform init

Terraform downloads the required AWS modules and providers.

The project uses:

terraform-aws-modules/vpc/aws

terraform-aws-modules/eks/aws

3. Review the Terraform Plan

terraform plan

Review the resources before applying the infrastructure.

4. Create the Infrastructure

terraform apply

Enter:

yes

Terraform provisions the AWS networking and EKS infrastructure.

🌐 VPC Configuration

The project creates a custom VPC:

CIDR: 10.0.0.0/16

Private Subnets

10.0.1.0/24
10.0.2.0/24

Public Subnets

10.0.101.0/24
10.0.102.0/24

The VPC uses:

2 Availability Zones

Public subnets

Private subnets

NAT Gateway

DNS hostnames

DNS support

EKS worker nodes are deployed into the private subnets.

☸️ EKS Configuration

Cluster:

monitoring-eks

Kubernetes version:

1.33

Region:

eu-north-1

Managed node group:

monitoring_nodes

Instance type:

t3.small

Scaling configuration:

Minimum:  1
Desired:  1
Maximum:  2

Capacity type:

ON_DEMAND

🔌 EKS Add-ons

The cluster uses the following EKS add-ons:

Amazon VPC CNI

CoreDNS

kube-proxy

AWS EBS CSI Driver

EKS Pod Identity Agent

The EBS CSI Driver is configured with IAM permissions through:

AmazonEBSCSIDriverPolicy

This allows Kubernetes workloads to provision and attach Amazon EBS volumes.

💾 Persistent Storage

Prometheus and Alertmanager require persistent storage.

The project uses the AWS EBS CSI driver with the gp3-csi StorageClass.

Example:

StorageClass: gp3-csi
Provisioner: ebs.csi.aws.com
Volume type: gp3
Filesystem: ext4

The final deployment successfully created persistent volumes for:

prometheus-server
storage-prometheus-alertmanager-0

Both PVCs reached:

STATUS: Bound

Check them with:

kubectl get pvc -n monitoring

📊 Prometheus Deployment

Prometheus is deployed using Helm in the monitoring namespace.

Verify:

kubectl get pods -n monitoring

Prometheus components include:

Prometheus Server

Alertmanager

kube-state-metrics

node-exporter

Pushgateway

Check Helm-managed monitoring workloads:

kubectl get pods -n monitoring

📈 Grafana Deployment

Grafana is deployed using Helm.

Verify:

kubectl get pods -n monitoring

The Grafana service is exposed internally as a Kubernetes ClusterIP.

For local access:

kubectl port-forward svc/grafana 3000:80 -n monitoring

Open:

http://localhost:3000

Grafana health can be checked with:

curl http://localhost:3000/api/health

Expected response contains:

{
  "database": "ok"
}

🔗 Prometheus as Grafana Data Source

Inside Grafana:

Connections
    ↓
Data Sources
    ↓
Prometheus

The Prometheus service is used as the Grafana data source.

After configuring the URL, use:

Save & Test

to verify connectivity.

📡 Metrics Server

Metrics Server provides Kubernetes resource metrics.

Verify:

kubectl get pods -n kube-system | grep metrics-server

Check node metrics:

kubectl top nodes

Check pod metrics:

kubectl top pods -A

This provides CPU and memory consumption for Kubernetes nodes and workloads.

📊 Grafana Monitoring Dashboard

The Grafana dashboard was configured to monitor:

Cluster CPU

sum(
  rate(container_cpu_usage_seconds_total{
    container!="",
    container!="POD",
    image!=""
  }[1m])
)

Cluster Filesystem Usage

100 * (
  1 - (
    sum(node_filesystem_avail_bytes{
      mountpoint="/",
      fstype!="rootfs"
    })
    /
    sum(node_filesystem_size_bytes{
      mountpoint="/",
      fstype!="rootfs"
    })
  )
)

Pod CPU Usage

sum by (namespace, pod) (
  rate(container_cpu_usage_seconds_total{
    container!="",
    container!="POD",
    image!=""
  }[1m])
)

Kubernetes System Pod CPU

sum by (namespace, pod) (
  rate(container_cpu_usage_seconds_total{
    namespace="kube-system",
    container!="",
    container!="POD",
    image!=""
  }[1m])
)

Pod Network I/O

sum by (namespace, pod) (
  rate(container_network_receive_bytes_total{pod!=""}[1m])
)
+
sum by (namespace, pod) (
  rate(container_network_transmit_bytes_total{pod!=""}[1m])
)

🔍 Kubernetes Verification Commands

Check cluster nodes

kubectl get nodes -o wide

Check all pods

kubectl get pods -A

Check monitoring workloads

kubectl get pods -n monitoring

Check services

kubectl get svc -n monitoring

Check persistent volumes

kubectl get pvc -n monitoring

Check StorageClasses

kubectl get storageclass

Check resource usage

kubectl top nodes
kubectl top pods -A

📸 Project Screenshots

Store the project screenshots inside a screenshots/ directory using the filenames below.

1. EKS Cluster

<img width="1901" height="900" alt="Screenshot 2026-08-11 231512" src="https://github.com/user-attachments/assets/96eb1b7d-91df-40f2-82ee-31aef15562cd" />


2. EKS Managed Node Group

<img width="1523" height="195" alt="Screenshot 2026-08-11 231736" src="https://github.com/user-attachments/assets/33d1af40-ebe7-4ea2-81a7-dc29d9b32157" />


3. EKS Networking

<img width="1582" height="241" alt="Screenshot 2026-08-11 231810" src="https://github.com/user-attachments/assets/e645f5a8-121f-4f8f-a5cc-eb03da46ab61" />



4. EKS Add-ons

<img width="1017" height="680" alt="Screenshot 2026-08-11 231845" src="https://github.com/user-attachments/assets/6f414b37-69be-451a-b0f1-315868dad6e6" />


5. Kubernetes Nodes

<img width="1192" height="342" alt="Screenshot 2026-08-11 231945" src="https://github.com/user-attachments/assets/4d3f30bc-19c3-4641-a4e8-93a8d7ba4cdc" />



6. Kubernetes Pods

<img width="1090" height="522" alt="Screenshot 2026-08-11 232022" src="https://github.com/user-attachments/assets/604b4e48-aaf6-4fad-8109-4b5fc2f386e6" />



7. Persistent Volume Claims

<img width="1167" height="187" alt="Screenshot 2026-08-11 232100" src="https://github.com/user-attachments/assets/156de65d-b6d0-4aa0-8db9-fcbd415779dd" />



8. Monitoring Workloads

<img width="817" height="210" alt="Screenshot 2026-08-11 232144" src="https://github.com/user-attachments/assets/5d58209f-75eb-431c-b36c-c2a52a50a8ae" />

<img width="1052" height="167" alt="Screenshot 2026-08-11 232205" src="https://github.com/user-attachments/assets/ec5e8043-49ce-44e8-9e85-b44a87d5ff81" />


9. Kubernetes Metrics

<img width="872" height="130" alt="Screenshot 2026-08-11 232246" src="https://github.com/user-attachments/assets/b28e4c58-5ce7-4b85-a250-22ad6cf32109" />

<img width="935" height="527" alt="Screenshot 2026-08-11 232306" src="https://github.com/user-attachments/assets/793d85a6-43cc-43e1-a5ec-671ad257f4d4" />


10. Prometheus Targets
<img width="1891" height="877" alt="Screenshot 2026-08-11 232505" src="https://github.com/user-attachments/assets/14441368-030c-4da1-a6ae-8c65d7eb3661" />

<img width="1883" height="780" alt="Screenshot 2026-08-11 232522" src="https://github.com/user-attachments/assets/df2876a5-9f46-4163-b764-39454e0702ed" />


11. Grafana Dashboard

<img width="1891" height="807" alt="Screenshot 2026-08-11 232637" src="https://github.com/user-attachments/assets/e38b1f94-2cf9-4453-9959-9ca318950b68" />

<img width="1567" height="566" alt="Screenshot 2026-08-11 232705" src="https://github.com/user-attachments/assets/11b2d28b-971f-44c1-869f-2829a34efd91" />

<img width="1562" height="667" alt="Screenshot 2026-08-11 232723" src="https://github.com/user-attachments/assets/a5a57c4f-7ef5-4d74-9efd-f261905f8a1e" />


🧪 Final Validation

The final monitoring environment was validated by checking:

kubectl get nodes

All active worker nodes were confirmed as Ready.

Monitoring workloads were verified with:

kubectl get pods -n monitoring

Prometheus and Grafana workloads were running successfully.

Persistent storage was verified with:

kubectl get pvc -n monitoring

The Prometheus and Alertmanager PVCs reached Bound.

Metrics Server was verified using:

kubectl top nodes
kubectl top pods -A

Grafana was verified through its health endpoint and the monitoring dashboard.

🧹 Cleanup

When the project is no longer required, remove the Helm releases:

helm uninstall grafana -n monitoring
helm uninstall prometheus -n monitoring

Then destroy the Terraform infrastructure:

cd terraform
terraform destroy

Confirm with:

yes

⚠️ terraform destroy removes the AWS resources managed by this Terraform configuration, including the EKS cluster and networking infrastructure. Use it only when the environment is no longer needed.

🎯 Key Learning Outcomes

Through this project, I gained practical experience with:

Infrastructure as Code using Terraform

AWS VPC design

Amazon EKS cluster provisioning

EKS managed node groups

Kubernetes add-ons

IAM permissions for Kubernetes workloads

AWS EBS CSI integration

Kubernetes PersistentVolumeClaims

EBS gp3 storage

Helm-based application deployment

Prometheus monitoring

Grafana visualization

Kubernetes Metrics Server

PromQL queries

Kubernetes troubleshooting

Resource utilization monitoring

Debugging pod scheduling and storage issues

🔄 End-to-End Flow

Terraform
   │
   ├── VPC
   │   ├── Public Subnets
   │   ├── Private Subnets
   │   └── NAT Gateway
   │
   └── EKS
       ├── Managed Node Group
       ├── VPC CNI
       ├── CoreDNS
       ├── kube-proxy
       ├── EBS CSI Driver
       └── Pod Identity Agent
                │
                ▼
          Kubernetes Cluster
                │
        ┌───────┴────────┐
        │                │
   Prometheus          Grafana
        │                │
        └───────┬────────┘
                │
        Monitoring Dashboard
                │
        ┌───────┴────────┐
        │                │
   CPU / Memory      Network / FS
   Pod Metrics       Kubernetes Metrics

⭐ Project Summary

This project demonstrates an end-to-end Kubernetes monitoring platform on AWS EKS, where Terraform provisions the cloud infrastructure, Helm deploys the monitoring stack, Prometheus collects metrics, Grafana visualizes them, Metrics Server provides Kubernetes resource metrics, and the AWS EBS CSI Driver provides persistent storage for monitoring workloads.

The result is a reusable AWS/Kubernetes monitoring environment that can be extended with additional dashboards, alerts, notification channels, and production workloads.

👨‍💻 Author

Abbas Baniyan

AWS & DevOps Engineer | Cloud & Kubernetes Enthusiast

⭐ If you found this project useful, feel free to star the repository.
