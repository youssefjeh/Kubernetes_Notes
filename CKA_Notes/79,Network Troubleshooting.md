# Network Troubleshooting in Kubernetes

## Network Plugins in Kubernetes

Several network plugins are available in Kubernetes. Below are some commonly used ones:

### 1. Weave Net
To install Weave Net:
```bash
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
Find more details in the official Kubernetes documentation:
[Networking and Network Policy](https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy)

### 2. Flannel
To install Flannel:
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
```
**Note:** Flannel does not support Kubernetes Network Policies as of now.

### 3. Calico
To install Calico:
```bash
curl https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml -O
kubectl apply -f calico.yaml
```
Calico is considered one of the most advanced CNI network plugins.

**Note for CKA and CKAD Exams:**
- You will not be asked to install the CNI plugin during the exams.
- If required, the exact URL will be provided.
- If multiple CNI configuration files are in the directory, kubelet uses the file that comes first in lexicographic order.

---

## DNS in Kubernetes
Kubernetes uses **CoreDNS**, a flexible and extensible DNS server, as its cluster DNS.

### Memory and Pods
In large-scale clusters, CoreDNS memory usage is affected by:
- Number of Pods and Services.
- Size of the DNS answer cache.
- Query rate (QPS) per CoreDNS instance.

CoreDNS resources in Kubernetes include:
- A service account: `coredns`
- Cluster roles: `coredns`, `kube-dns`
- Cluster role bindings: `coredns`, `kube-dns`
- A deployment: `coredns`
- A config map: `coredns`
- A service: `kube-dns`

### Corefile Configuration Example
```text
kubernetes cluster.local in-addr.arpa ip6.arpa {
   pods insecure
   fallthrough in-addr.arpa ip6.arpa
   ttl 30
}
```
This serves as the backend to `cluster.local` and reverse domains.

For forwarding out-of-cluster domains:
```text
proxy . /etc/resolv.conf
```

### Troubleshooting CoreDNS Issues
1. **CoreDNS Pods in Pending State**
   - Ensure the network plugin is installed.

2. **CoreDNS Pods CrashLoopBackOff or Error State**
   - If running SELinux with an older Docker version, consider:
     - Upgrading Docker.
     - Disabling SELinux.
     - Setting `allowPrivilegeEscalation` to true:
       ```bash
       kubectl -n kube-system get deployment coredns -o yaml | \
         sed 's/allowPrivilegeEscalation: false/allowPrivilegeEscalation: true/g' | \
         kubectl apply -f -
       ```
   - Handle loop detection issues by:
     - Updating kubelet config:
       ```yaml
       resolvConf: <path-to-real-resolv-conf>
       ```
     - Disabling local DNS cache on host nodes.
     - Editing Corefile to replace:
       ```text
       forward . /etc/resolv.conf
       ```
       with the IP of the upstream DNS (e.g., `forward . 8.8.8.8`).

3. **CoreDNS Pods and kube-dns Service Running but No Endpoints**
   - Verify the kube-dns service has valid endpoints:
     ```bash
     kubectl -n kube-system get ep kube-dns
     ```
   - Inspect the service and ensure correct selectors and ports.

---

## Kube-Proxy
`kube-proxy` is a network proxy running on each cluster node, maintaining network rules for communication between Pods and external clients.

### Configuration
In clusters configured with `kubeadm`, `kube-proxy` runs as a DaemonSet. Its configuration is specified in `/var/lib/kube-proxy/config.conf`.

Example command inside the `kube-proxy` container:
```bash
/usr/local/bin/kube-proxy \
  --config=/var/lib/kube-proxy/config.conf \
  --hostname-override=$(NODE_NAME)
```

The config file includes settings such as:
- `clusterCIDR`
- `kube-proxy` mode (`ipvs`, `iptables`)
- `bindAddress`
- `kube-config`

### Troubleshooting Kube-Proxy Issues
1. Ensure the `kube-proxy` pod is running in the `kube-system` namespace.
2. Check `kube-proxy` logs for errors.
3. Verify the config map and ensure the configuration file is accurate.
4. Confirm `kube-config` is properly defined.
5. Verify `kube-proxy` is running inside the container:
   ```bash
   netstat -plan | grep kube-proxy
   ```
   Example output:
   ```text
   tcp        0      0 0.0.0.0:30081           0.0.0.0:*               LISTEN      1/kube-proxy
   tcp        0      0 127.0.0.1:10249         0.0.0.0:*               LISTEN      1/kube-proxy
   tcp        0      0 172.17.0.12:33706       172.17.0.12:6443        ESTABLISHED 1/kube-proxy
   tcp6       0      0 :::10256                :::*                    LISTEN      1/kube-proxy
   ```

---

## References
- **Debugging Services:** [Kubernetes Debug Service](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/)
- **DNS Troubleshooting:** [Kubernetes DNS Debugging](https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/)
