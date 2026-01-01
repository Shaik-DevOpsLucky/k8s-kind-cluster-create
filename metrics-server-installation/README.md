# Kubernetes Metrics Server Installation & Configuration

This guide explains how to install and configure **Metrics Server** on a Kubernetes cluster (tested on `kind` multi-node cluster) to enable **Horizontal Pod Autoscaler (HPA)** and resource metrics.

---

## **1️⃣ Prerequisites**

* Kubernetes cluster running (control-plane + workers)
* `kubectl` configured to access the cluster
* Sufficient disk space for containers/images

> ⚠️ Note for **kind clusters**: Metrics Server needs the `--kubelet-insecure-tls` flag because kind nodes use self-signed certificates.

---

## **2️⃣ Install Metrics Server**

Apply the official Metrics Server manifest:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Verify pods:

```bash
kubectl get pods -n kube-system | grep metrics-server
```

* Initially, pods may be in `Pending` or `CrashLoopBackOff` if disk space is low or image pull fails.

---

## **3️⃣ Configure Metrics Server for kind clusters**

Edit the deployment to allow insecure TLS:

```bash
kubectl edit deployment metrics-server -n kube-system
```

Add the following arguments under `containers.args`:

```yaml
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
```

Save and exit.

---

### **4️⃣ Restart Metrics Server**

```bash
kubectl rollout restart deployment metrics-server -n kube-system
kubectl get pods -n kube-system -w
```

* Wait until all pods show `1/1 Running`.

---

## **5️⃣ Verify Metrics Server**

Check node metrics:

```bash
kubectl top nodes
```

Check pod metrics:

```bash
kubectl top pods
```

> If metrics appear correctly, Metrics Server is fully operational and HPA can now scale workloads.

---

## **6️⃣ Common Issues**

| Issue                       | Solution                                                                            |
| --------------------------- | ----------------------------------------------------------------------------------- |
| `ImagePullBackOff`          | Ensure host has enough disk space. Delete and recreate kind cluster if needed.      |
| `Metrics API not available` | Check Metrics Server pod status; ensure `--kubelet-insecure-tls` is added for kind. |
| Disk full errors            | Clean Docker images/containers or increase EC2 root volume.                         |

---

## **7️⃣ References**

* [Metrics Server GitHub](https://github.com/kubernetes-sigs/metrics-server)
* [HPA Documentation](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
* [Kind Quick Start](https://kind.sigs.k8s.io/docs/user/quick-start/)

---

✅ Now your Kubernetes cluster has a working **Metrics Server**, ready for **HPA** and resource monitoring.
