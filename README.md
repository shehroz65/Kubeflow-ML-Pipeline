# Setting Up Kubeflow on Azure VM

## Prerequisites
- Azure VM with 32 GB RAM, 4 VCPUs, and 64 GB storage (Standard SSD)
- SSH access to the VM using a private key provided by Azure

## Steps

### 1. Create Azure VM
Create an Azure VM with the following specifications:
- RAM: 32 GB
- VCPUs: 4
- Storage: 64 GB (Standard SSD)

### 2. SSH into the VM
Use your private key to SSH into the VM:
```sh
ssh -i /path/to/your/private-key.pem username@your-vm-ip
```

### 3. Install MicroK8s and Kubeflow
Follow the guide at https://charmed-kubeflow.io/docs/get-started-with-charmed-kubeflow to install MicroK8s and Kubeflow.

### 4. Deploy Kubeflow Ingress
Create a file named kubeflow-ingress.yaml. My file is given in this repo. Apply using the below command
# microk8s kubectl apply -f kubeflow-ingress.yaml

### 5. Port Forward Ingress to Local Machine
# microk8s kubectl port-forward -n kubeflow svc/istio-ingressgateway-workload 8080:80
Access the Kubeflow dashboard on your local PC using http://localhost:8080.

### 6. Get ClusterIP for Minio
```sh
microk8s kubectl get svc -n kubeflow
```
Note the ClusterIP for Minio (used for saving the model).

### 7. Create Minio Service with NodePort
Create a file named minionew.yaml . My file is given in this repo.Apply the Minio service configuration:
```sh
microk8s kubectl apply -f minionew.yaml
```
### 8. Port Forward Minio to Local Machine
```sh
microk8s kubectl port-forward -n kubeflow svc/minio 9000:9000 9001:9001
```
In a separate terminal, forward the ports via SSH:
```sh
ssh -i /path/to/your/private-key.pem -L 9000:localhost:9000 -L 9001:localhost:9001 username@your-vm-ip
```

### 9. Get Minio Access and Secret Keys
```sh
microk8s kubectl get secret mlpipeline-minio-artifact -n kubeflow -o jsonpath="{.data.accesskey}" | base64 --decode
microk8s kubectl get secret mlpipeline-minio-artifact -n kubeflow -o jsonpath="{.data.secretkey}" | base64 --decode
```
### 10. Access Minio Console Dashboard
Access the Minio console dashboard at http://localhost:9001/login.

Enter your decoded access key as the username and the decoded secret key as the password.

For example, in Python:
```sh
import base64
access_key = base64.b64decode("bWluaW8=").decode('utf-8')
secret_key = base64.b64decode("RDZOWTFEN1JSVVc3NklQUjNHOTJGNUNUVUZCTUhG").decode('utf-8')
```
