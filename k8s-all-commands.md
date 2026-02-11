# EKS/Kubernetes Context Management & Daily Operations Guide

## Table of Contents
- [Kubeconfig Management](#kubeconfig-management)
- [Context Management](#context-management)
- [Pod Operations](#pod-operations)
- [Deployment Management](#deployment-management)
- [Service & Networking](#service--networking)
- [Logs & Debugging](#logs--debugging)
- [Resource Management](#resource-management)
- [ConfigMaps & Secrets](#configmaps--secrets)
- [Namespace Operations](#namespace-operations)
- [Cluster Information](#cluster-information)
- [Troubleshooting](#troubleshooting)

---

### View Kubeconfig File Location
```bash
# Default location
~/.kube/config

# View current config
kubectl config view

---

## Context Management

### List and View Contexts
```bash
# List all configured contexts
kubectl config get-contexts

# Show current context
kubectl config current-context

# View specific context details
kubectl config get-contexts <context-name>

# Set default namespace for context
kubectl config set-context --current --namespace=<namespace-name>

```

### Add EKS Clusters to Kubeconfig
```bash
# Add Dev cluster (Cluster1)
aws eks update-kubeconfig --region <region> --name cluster1 --alias dev-cluster

# Add UAT cluster (Cluster2)
aws eks update-kubeconfig --region <region> --name cluster2 --alias uat-cluster

# Example: Add clusters in ap-south-1 region
aws eks update-kubeconfig --region ap-south-1 --name ekswithavinash --alias dev-cluster
aws eks update-kubeconfig --region ap-south-1 --name ekswithavinash-uat --alias uat-cluster

# Add cluster with specific profile
aws eks update-kubeconfig --region <region> --name <cluster-name> --alias <alias> --profile <aws-profile>
```

### Switch Between Contexts
```bash
# Switch to Dev/Cluster1
kubectl config use-context dev-cluster

# Switch to UAT/Cluster2
kubectl config use-context uat-cluster
```

### Run Commands on Specific Context
```bash
# Query specific context without switching
kubectl --context=dev-cluster get pods
kubectl --context=uat-cluster get pods

# Get all resources in specific context
kubectl --context=dev-cluster get all -A
```

### Rename/Delete Contexts
```bash
# Rename context
kubectl config rename-context old-name new-name

# Delete context
kubectl config delete-context <context-name>

# Delete cluster from config
kubectl config delete-cluster <cluster-name>

# Delete user from config
kubectl config delete-user <user-name>
```

---

## Pod Operations

### List Pods
```bash
# List pods in current namespace
kubectl get pods

# List pods in all namespaces
kubectl get pods -A
kubectl get pods --all-namespaces

# List pods in specific namespace
kubectl get pods -n <namespace>

# List pods with more details
kubectl get pods -o wide

# Watch pods in real-time
kubectl get pods -w

# List pods with labels
kubectl get pods --show-labels

# Filter pods by label
kubectl get pods -l app=nginx
kubectl get pods -l environment=production
```

### Describe and Inspect Pods
```bash
# Describe pod (detailed info)
kubectl describe pod <pod-name>
kubectl describe pod <pod-name> -n <namespace>

# Get pod YAML
kubectl get pod <pod-name> -o yaml

# Get pod JSON
kubectl get pod <pod-name> -o json

# Get specific field from pod
kubectl get pod <pod-name> -o jsonpath='{.status.podIP}'
```

### Execute Commands in Pods
```bash
# Execute command in pod
kubectl exec <pod-name> -- <command>

# Interactive shell in pod
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -- /bin/sh

# Execute in specific container (multi-container pod)
kubectl exec -it <pod-name> -c <container-name> -- /bin/bash

# Execute in specific namespace
kubectl exec -it <pod-name> -n <namespace> -- /bin/bash
```

### Delete Pods
```bash
# Delete specific pod
kubectl delete pod <pod-name>

# Delete pod in specific namespace
kubectl delete pod <pod-name> -n <namespace>

# Force delete pod
kubectl delete pod <pod-name> --force --grace-period=0

# Delete all pods with label
kubectl delete pods -l app=nginx

# Delete all pods in namespace
kubectl delete pods --all -n <namespace>
```

---

## Deployment Management

### List Deployments
```bash
# List deployments
kubectl get deployments
kubectl get deploy

# List in all namespaces
kubectl get deployments -A

# List with details
kubectl get deployments -o wide
```

### Create/Update Deployments
```bash
# Create deployment from file
kubectl apply -f deployment.yaml

# Create deployment from command
kubectl create deployment nginx --image=nginx:latest

# Update deployment image
kubectl set image deployment/<deployment-name> <container-name>=<new-image>

# Edit deployment
kubectl edit deployment <deployment-name>
```

### Scale Deployments
```bash
# Scale deployment
kubectl scale deployment <deployment-name> --replicas=3

# Autoscale deployment
kubectl autoscale deployment <deployment-name> --min=2 --max=10 --cpu-percent=80
```

### Rollout Management
```bash
# Check rollout status
kubectl rollout status deployment/<deployment-name>

# View rollout history
kubectl rollout history deployment/<deployment-name>

# Undo rollout
kubectl rollout undo deployment/<deployment-name>

# Rollback to specific revision
kubectl rollout undo deployment/<deployment-name> --to-revision=2

# Pause rollout
kubectl rollout pause deployment/<deployment-name>

# Resume rollout
kubectl rollout resume deployment/<deployment-name>

# Restart deployment (rolling restart)
kubectl rollout restart deployment/<deployment-name>
```

### Delete Deployments
```bash
# Delete deployment
kubectl delete deployment <deployment-name>

# Delete deployment with associated resources
kubectl delete deployment <deployment-name> --cascade=foreground
```

---

## Service & Networking

### List Services
```bash
# List services
kubectl get svc
kubectl get services

# List in all namespaces
kubectl get svc -A

# Get service details
kubectl describe svc <service-name>
```

### Create Services
```bash
# Expose deployment as service
kubectl expose deployment <deployment-name> --port=80 --target-port=8080 --type=LoadBalancer

# Create service from file
kubectl apply -f service.yaml
```

### Ingress Management
```bash
# List ingress
kubectl get ingress
kubectl get ing -A

# Describe ingress
kubectl describe ingress <ingress-name>

# Create/update ingress
kubectl apply -f ingress.yaml
```

### Network Policies
```bash
# List network policies
kubectl get networkpolicies
kubectl get netpol

# Describe network policy
kubectl describe networkpolicy <policy-name>
```

---

## Logs & Debugging

### View Logs
```bash
# View pod logs
kubectl logs <pod-name>

# View logs from specific container
kubectl logs <pod-name> -c <container-name>

# Follow logs (real-time)
kubectl logs -f <pod-name>

# View logs from previous pod instance
kubectl logs <pod-name> --previous

# View last N lines
kubectl logs <pod-name> --tail=100

# View logs since timestamp
kubectl logs <pod-name> --since=1h
kubectl logs <pod-name> --since-time=2024-01-01T00:00:00Z

# View logs from all containers in pod
kubectl logs <pod-name> --all-containers=true

# View logs from deployment
kubectl logs deployment/<deployment-name>
```

### Port Forwarding
```bash
# Forward local port to pod
kubectl port-forward <pod-name> 8080:80

# Forward to service
kubectl port-forward service/<service-name> 8080:80

# Forward to deployment
kubectl port-forward deployment/<deployment-name> 8080:80

# Listen on all interfaces
kubectl port-forward --address 0.0.0.0 <pod-name> 8080:80
```

### Debugging Tools
```bash
# Run debug container alongside pod
kubectl debug <pod-name> -it --image=busybox

# Create debug pod in same node
kubectl debug node/<node-name> -it --image=ubuntu

# Copy files from pod
kubectl cp <pod-name>:/path/to/file /local/path

# Copy files to pod
kubectl cp /local/path <pod-name>:/path/to/file

# Run temporary pod for debugging
kubectl run debug --rm -it --image=busybox -- /bin/sh
```

---

## Resource Management

### Get Resource Usage
```bash
# Get node resource usage
kubectl top nodes

# Get pod resource usage
kubectl top pods

# Get pod resource usage in namespace
kubectl top pods -n <namespace>

# Get pod resource usage with containers
kubectl top pods --containers
```

### Resource Quotas
```bash
# List resource quotas
kubectl get resourcequota
kubectl get quota

# Describe quota
kubectl describe resourcequota <quota-name>
```

### Limit Ranges
```bash
# List limit ranges
kubectl get limitrange

# Describe limit range
kubectl describe limitrange <limitrange-name>
```

### Horizontal Pod Autoscaler (HPA)
```bash
# List HPA
kubectl get hpa

# Describe HPA
kubectl describe hpa <hpa-name>

# Create HPA
kubectl autoscale deployment <deployment-name> --cpu-percent=50 --min=1 --max=10
```

---

## ConfigMaps & Secrets

### ConfigMaps
```bash
# List configmaps
kubectl get configmaps
kubectl get cm

# Create configmap from literal
kubectl create configmap <name> --from-literal=key1=value1 --from-literal=key2=value2

# Create configmap from file
kubectl create configmap <name> --from-file=config.txt

# Create configmap from directory
kubectl create configmap <name> --from-file=/path/to/dir

# Describe configmap
kubectl describe configmap <name>

# Edit configmap
kubectl edit configmap <name>

# Delete configmap
kubectl delete configmap <name>
```

### Secrets
```bash
# List secrets
kubectl get secrets

# Create generic secret
kubectl create secret generic <name> --from-literal=password=mypassword

# Create secret from file
kubectl create secret generic <name> --from-file=ssh-key=/path/to/key

# Create docker registry secret
kubectl create secret docker-registry <name> \
  --docker-server=<server> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>

# Create TLS secret
kubectl create secret tls <name> --cert=/path/to/cert --key=/path/to/key

# Describe secret
kubectl describe secret <name>

# View secret data (base64 encoded)
kubectl get secret <name> -o yaml

# Decode secret
kubectl get secret <name> -o jsonpath='{.data.password}' | base64 --decode
```

---

## Namespace Operations

### List and Manage Namespaces
```bash
# List namespaces
kubectl get namespaces
kubectl get ns

# Create namespace
kubectl create namespace <namespace-name>

# Delete namespace
kubectl delete namespace <namespace-name>


# View current namespace
kubectl config view --minify | grep namespace:
```

### Get Resources Across Namespaces
```bash
# Get all pods across namespaces
kubectl get pods -A

# Get specific resource type across namespaces
kubectl get deployments -A
kubectl get services -A
kubectl get configmaps -A
```

---

## Cluster Information

### Cluster Details
```bash
# Get cluster info
kubectl cluster-info

# Get cluster info dump (detailed)
kubectl cluster-info dump

# List nodes
kubectl get nodes

# Get node details
kubectl get nodes -o wide

# Describe node
kubectl describe node <node-name>

# Get node labels
kubectl get nodes --show-labels

# Get API resources
kubectl api-resources

# Get API versions
kubectl api-versions
```

### Component Status
```bash
# Get events
kubectl get events

# Get events sorted by timestamp
kubectl get events --sort-by='.lastTimestamp'

# Watch events
kubectl get events -w
```

---

## Troubleshooting

### Common Troubleshooting Commands
```bash
# Check pod status and events
kubectl describe pod <pod-name>

# Check pod logs for errors
kubectl logs <pod-name> --previous

# Check resource constraints
kubectl top pods
kubectl top nodes

# Check service endpoints
kubectl get endpoints <service-name>

# Verify DNS resolution
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup <service-name>

# Check network connectivity
kubectl run -it --rm debug --image=nicolaka/netshoot --restart=Never -- /bin/bash

# Validate YAML before applying
kubectl apply -f manifest.yaml --dry-run=client

# Explain resource fields
kubectl explain pod.spec.containers

# Get pod conditions
kubectl get pods <pod-name> -o jsonpath='{.status.conditions[*].type}'
```

### EKS-Specific Troubleshooting
```bash
# Check EKS cluster status
aws eks describe-cluster --name <cluster-name> --region <region>

# List EKS node groups
aws eks list-nodegroups --cluster-name <cluster-name> --region <region>

# Describe node group
aws eks describe-nodegroup --cluster-name <cluster-name> --nodegroup-name <nodegroup-name> --region <region>

# Check EKS add-ons
aws eks list-addons --cluster-name <cluster-name> --region <region>

# Describe add-on
aws eks describe-addon --cluster-name <cluster-name> --addon-name <addon-name> --region <region>

# Update kubeconfig with verbose output
aws eks update-kubeconfig --name <cluster-name> --region <region> --verbose
```

### Performance and Optimization
```bash
# Find pods using most CPU
kubectl top pods -A | sort --reverse --key 3 --numeric

# Find pods using most memory
kubectl top pods -A | sort --reverse --key 4 --numeric

# Get pods not in Running state
kubectl get pods -A --field-selector=status.phase!=Running

# Get pods with restart count
kubectl get pods -A -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.containerStatuses[*].restartCount}{"\n"}{end}'

# Delete all failed pods
kubectl delete pods --field-selector=status.phase=Failed -A

# Delete all evicted pods
kubectl get pods -A | grep Evicted | awk '{print $2, $1}' | xargs -n2 kubectl delete pod -n
```

---

## Useful Aliases and Shortcuts

Add these to your `~/.bashrc` or `~/.zshrc`:

```bash
# Kubectl aliases
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgpa='kubectl get pods -A'
alias kgd='kubectl get deployments'
alias kgs='kubectl get svc'
alias kgn='kubectl get nodes'
alias kdp='kubectl describe pod'
alias kdd='kubectl describe deployment'
alias kl='kubectl logs'
alias klf='kubectl logs -f'
alias kex='kubectl exec -it'
alias kctx='kubectl config use-context'
alias kns='kubectl config set-context --current --namespace'

# EKS aliases
alias eks-dev='kubectl config use-context dev-cluster'
alias eks-uat='kubectl config use-context uat-cluster'
alias eks-update-dev='aws eks update-kubeconfig --region ap-south-1 --name ekswithavinash --alias dev-cluster'
alias eks-update-uat='aws eks update-kubeconfig --region ap-south-1 --name ekswithavinash-uat --alias uat-cluster'
```

---

**Note**: Replace `<placeholders>` with actual values when executing commands.
