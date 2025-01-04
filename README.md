# Automate Kubernetes Deployment

## Overview
This project demonstrates the automation of Kubernetes deployments using Terraform and Ansible. It provisions an AWS EKS cluster using Terraform and deploys an application in a new Kubernetes namespace using an Ansible playbook.

## Technologies Used
- **Terraform**: For provisioning the EKS cluster.
- **Ansible**: For deploying Kubernetes resources.
- **Kubernetes 12.0.0**: Container orchestration platform.
- **AWS EKS**: Managed Kubernetes service on AWS.
- **Python**: Dependency for Kubernetes-related modules in Ansible.
- **Linux**: Development environment.

---

## Project Structure
```
├── ansible.cfg
├── deploy-to-k8s.yaml
├── hosts
├── nginx.yaml
├── terraform/
│   ├── eks-cluster.tf
│   ├── providers.tf
│   ├── terraform.tfvars
```

---

## Prerequisites
1. **AWS CLI** installed and configured.
2. **Terraform** installed (`>=1.0` recommended).
3. **Ansible** installed with required plugins:
   - `kubernetes.core`
4. Python dependencies:
   - `pyyaml`, `kubernetes`, `jsonpatch` (install using `pip3 install pyyaml kubernetes jsonpatch`).

---

## Step 1: Provision the EKS Cluster with Terraform
1. Navigate to the `terraform` directory:
   ```bash
   cd terraform
   ```

2. Initialize the Terraform project:
   ```bash
   terraform init
   ```

3. Apply the Terraform configuration to create the EKS cluster:
   ```bash
   terraform apply --auto-approve
   ```

4. Verify that the EKS cluster is created and update the kubeconfig:
   ```bash
   aws eks update-kubeconfig --region us-east-1 --name myapp-eks-cluster --kubeconfig ~/terraform/kubeconfig_myapp-eks-cluster
   export KUBECONFIG=/Users/.../_myapps-eks-cluster
   kubectl cluster-info
   ```

---

## Step 2: Configure Ansible
1. Update the `ansible.cfg` file with the following settings:
   ```ini
   [defaults]
   host_key_checking = False
   inventory = hosts
   enable_plugins = aws_ec2
   remote_user = ec2-user
   private_key_file = ~/.ssh/id_ed22519_ec2
   ```

2. Verify Ansible connectivity to the EC2 instance:
   ```bash
   ansible -i hosts all -m ping
   ```

---

## Step 3: Deploy Application to Kubernetes with Ansible
1. Ensure the kubeconfig is set up:
   ```bash
   export K8S_AUTH_KUBECONFIG=/Users/.../_myapps-eks-cluster
   ```

2. Navigate to the `ansible` directory and run the playbook:
   ```bash
   ansible-playbook deploy-to-k8s.yaml
   ```

### Sample Output:
```
PLAY [Deploy app in new namespace] *******************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************
ok: [localhost]

TASK [Create a k8s namespace] ************************************************************************************************************************
ok: [localhost]

TASK [Deploy nginx app] ******************************************************************************************************************************
changed: [localhost]

PLAY RECAP *******************************************************************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

---

## Application Deployment Details
### Namespace Creation
The playbook creates a Kubernetes namespace `my-app` using the `kubernetes.core.k8s` module.

### NGINX Deployment
The `nginx.yaml` manifest deploys an NGINX application with the following configuration:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: LoadBalancer
```

---

## Verify Deployment
1. Check pods in the `my-app` namespace:
   ```bash
   kubectl get pods -n my-app
   ```
   Sample Output:
   ```
   NAME                     READY   STATUS    RESTARTS   AGE
   nginx-7769f8f85b-5nmjk   1/1     Running   0          2m27s
   nginx-7769f8f85b-qlrnm   1/1     Running   0          2m27s
   ```

2. Check the service details:
   ```bash
   kubectl get svc -n my-app
   ```
   Sample Output:
   ```
   NAME    TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)        AGE
   nginx   LoadBalancer   172.20.243.186   a2f541fc7ed85446384f5746378bd803-299308098.us-east-1.elb.amazonaws.com   80:30335/TCP   22s
   ```

3. Access the application in the browser using the external IP of the service.

---

## Troubleshooting
1. If you encounter the error:
   ```
   Unhandled Error" err="couldn't get current server API group list: the server has asked for the client to provide credentials"
   ```
   Ensure you have the correct IAM policies attached to your user or role:
   - **AmazonEKSAdminViewPolicy**
   - **AmazonEKSClusterAdminPolicy**

2. Ensure all Python dependencies are installed:
   ```bash
   pip3 install pyyaml kubernetes jsonpatch
   ```

3. Verify connectivity between Ansible and the Kubernetes cluster using the `kubeconfig`.

---

## Cleanup
1. To remove the deployed application, change `state: present` to `state: absent` in the playbook and re-run it:
   ```bash
   ansible-playbook deploy-to-k8s.yaml
   ```

2. Destroy the EKS cluster to release resources:
   ```bash
   cd terraform
   terraform destroy --auto-approve
   ```

---

## Conclusion
This project automates the provisioning of Kubernetes infrastructure and application deployment, showcasing the power of Terraform and Ansible for DevOps workflows.
