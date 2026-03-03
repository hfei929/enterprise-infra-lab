# Kubernetes Cluster Architecture Design

## 1. Background

Build a 3-node Kubernetes cluster using kubeadm for production simulation.

## 2. Architecture Diagram

(Insert draw.io diagram here)

- 1 Master
- 2 Worker
- etcd local
- Calico network

## 3. Component Explanation

- kube-apiserver
- controller-manager
- scheduler
- etcd
- kubelet

## 4. High Availability Consideration

- Single master risk
- etcd backup strategy
- Future HA control-plane plan

## 5. Risk Analysis

- Node failure
- etcd corruption
- Network misconfiguration

## 6. Optimization Direction

- Upgrade to multi-master
- External etcd
- Resource tuning
