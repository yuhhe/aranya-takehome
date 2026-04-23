# Aranya Kubernetes Cluster Build Guide

## Overview
This document outlines the steps to reproduce the Kubernetes cluster for the Aranya technical test.

## Prerequisites
- Ansible installed on the control machine.
- Python 3.11+ (Upgraded from 3.7 to support modern Kubespray/Ansible).

## Deployment Steps
1. **Infrastructure Prep**: 
   - DigitalOcean droplets (Ubuntu 22.04).
   - SSH keys injected for `root`.
2. **Kubespray Configuration**:
   - Inventory defined in `inventory/hosts.yaml`.
   - CNI: Initially attempted Cilium, but switched to **Calico** for better compatibility with the target environment and to ensure 24h SLA.
3. **Cluster Provisioning**:
   ```bash
   ansible-playbook -i inventory/hosts.yaml cluster.yml --become
