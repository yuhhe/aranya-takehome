Aranya Infrastructure Challenge
1. Overview
This repository contains the infrastructure code and operational notes for the Aranya take-home assignment. The goal was to provision a high-availability Kubernetes cluster on DigitalOcean, implement GitOps via ArgoCD, and ensure secure, private network communication between nodes.

2. Infrastructure & Provisioning
Tooling: Provisioned via Kubespray for a production-grade Kubernetes distribution.

Node Topology:

1 Control Plane / Worker (node1)

2 Workers (node2, node3)

Local Environment: Successfully resolved Python 3.12 dependency conflicts and Ansible collection requirements to ensure a stable deployment pipeline.

3. Network Architecture & CNI Migration (Key Technical Decisions)
During the initial deployment, the environment faced challenges with the default CNI and node taints. I made the strategic decision to migrate from Cilium to Calico to ensure network stability.

Private Network Compliance
As per the requirement for nodes to communicate via private networks:

Interface Binding: Explicitly configured Calico to use the eth1 interface (DigitalOcean's VPC/Private Network).

Configuration:

Bash
# Command applied to ensure inter-node traffic stays off the public internet:
kubectl set env daemonset/calico-node -n kube-system IP_AUTODETECTION_METHOD=interface=eth1
Result: Reduced public bandwidth costs, enhanced security, and optimized BGP peering latency.

4. GitOps Implementation
ArgoCD: Deployed in the argocd namespace to manage the cluster state.

ClusterdOS Application: Configured to sync with the Aranya GitLab repository.

Operational Note: Due to the CNI transition, the initial synchronization might experience transient timeouts (deadline exceeded). I have verified the manifests and adjusted MTU settings (1440) to accommodate DigitalOcean's encapsulation overhead.

5. Security & Access
SSH Access: Authorized Christian's public key (condaatje) across all nodes. Access was verified using the provided id_rsa_aranya identity.

Credential Management: The cluster kubeconfig has been exported and encrypted using GPG (aranya-kubeconfig.gpg) for secure delivery.

6. Verification Commands
To audit the cluster status, you can use the following:

Bash
# Verify Web Application (NodePort)
curl -I http://138.197.209.135:31080

# Verify CNI Private Interface Binding
kubectl get pods -n kube-system -l k8s-app=calico-node -o jsonpath='{.items[*].spec.containers[*].env}'

# Verify ArgoCD Applications
kubectl get applications -n argocd
7. Post-Mortem / Future Improvements
Automated VPC Provisioning: In a full production scenario, I would use Terraform to pre-provision the DigitalOcean VPC and inject private IPs directly into the Kubespray inventory.

Monitoring: Integration of Prometheus/Grafana would be the next step to monitor BGP peering health and network convergence times.