# 🚀 Deploying a Production-Grade Kubernetes Cluster on AWS using kOps

This guide will walk you through creating a production-ready Kubernetes cluster on AWS using kOps, the powerful cluster management tool.

**kOps** ("Kubernetes Operations") makes it easy to create, destroy, upgrade, and maintain clusters on AWS and other cloud providers.

## 📋 Prerequisites

- AWS Account with appropriate permissions
- Linux/macOS machine for running kOps commands
- Basic understanding of Kubernetes and AWS services

## 🛠️ Installation and Setup

### Step 1: Install kOps

```bash
# Download and install kOps (Linux)
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x ./kops
sudo mv ./kops /usr/local/bin/
```

### Step 2: Install kubectl

kubectl is the command-line tool used to manage Kubernetes clusters.

```bash
# Download and install kubectl (Linux)
curl -Lo kubectl https://dl.k8s.io/release/$(curl -s -L https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

## 🔐 AWS Configuration

### Step 3: Create IAM User for kOps

Create a dedicated IAM user in AWS and attach the following policies:

- `AmazonEC2FullAccess`
- `AmazonRoute53FullAccess`
- `AmazonS3FullAccess`
- `IAMFullAccess`
- `AmazonVPCFullAccess`
- `AmazonSQSFullAccess`
- `AmazonEventBridgeFullAccess`

**Important:** Record your AccessKeyID and SecretAccessKey for configuration.

### Step 4: Configure AWS CLI

```bash
# Configure AWS CLI with your credentials
aws configure    # Enter Access key, Secret key, region, and output format

# Verify configuration
aws iam list-users

# Export the keys for kOps
export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)

# Verify environment variables
printenv
```

## 🪣 State Store Setup

### Step 5: Create an S3 Bucket for kOps State

kOps uses an S3 bucket as the state store — the single source of truth for your cluster configuration.

```bash
# Create S3 bucket (replace 'prefix-example-com-state-store' with your desired name)
aws s3api create-bucket \
    --bucket prefix-example-com-state-store \
    --region us-east-1
```

**Enable versioning (highly recommended):**

```bash
aws s3api put-bucket-versioning \
    --bucket prefix-example-com-state-store \
    --versioning-configuration Status=Enabled
```

## ⚙️ Environment Configuration

### Step 6: Set Environment Variables

For a gossip-based cluster, ensure your cluster name ends with `.k8s.local`.

```bash
# Set cluster name and state store
export NAME=myfirstcluster.k8s.local
export KOPS_STATE_STORE=s3://prefix-example-com-state-store

# Generate SSH keys for accessing cluster nodes
ssh-keygen
```

## 🚀 Cluster Deployment

### Step 7: Create the Kubernetes Cluster

```bash
# Create cluster configuration
kops create cluster --name=${NAME} --cloud=aws --zones=ap-southeast-1a
```

**Optional:** Edit the cluster configuration before deployment:

```bash
kops edit cluster --name ${NAME}
```

### Step 8: Deploy the Cluster

```bash
# Deploy the cluster
kops update cluster --name ${NAME} --yes --admin
```

Wait a few minutes for resources (EC2, VPC, S3, Route53, etc.) to be created.

## ✅ Validation and Management

### Step 9: Validate the Cluster

```bash
# Check if your cluster is ready
kops validate cluster
```

### Step 10: Cleanup (Delete Cluster)

```bash
# Delete the entire cluster
kops delete cluster --name ${NAME} --yes
```

## 🏁 Production Recommendations

- **Always enable versioning** on your S3 state store for rollback and recovery capabilities
- **Use multiple availability zones** for production-grade resilience
- **Consider enabling** autoscaling, monitoring, and backups after initial setup
- **Regularly update** your cluster components for security and performance
- **Implement proper RBAC** policies for cluster access control

## 📚 Additional Resources

- [kOps Documentation](https://kops.sigs.k8s.io/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [AWS EKS vs kOps Comparison](https://aws.amazon.com/eks/)

## 🤝 Contributing

Feel free to submit issues and enhancement requests!

## 📄 License

This project is open source and available under the [MIT License](LICENSE).
