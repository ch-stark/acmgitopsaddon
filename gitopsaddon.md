# Red Hat Advanced Cluster Management (RHACM) GitOps Addon

The **RHACM GitOps Addon** is a lifecycle manager that facilitates the transition from a centralized "Push" model to a decentralized **Pull Model**. This shift moves the architectural responsibility of reconciliation from the central Hub cluster to the managed clusters (spokes), significantly improving scalability and resilience.

RHACM currently supports two primary pull-based architectures:
1. **The Basic Pull Model** (via `gitops-addon`)
2. **The Advanced Pull Model** (via the Argo CD Agent)

---

## 1. How the GitOps Addon Works

Instead of the Hub cluster maintaining persistent outbound connections to every managed cluster to "push" changes, the addon decentralizes the operation through the following mechanisms:

* **Operator Management:** When enabled, the addon automatically installs and manages the **OpenShift GitOps Operator** on the managed cluster.
* **Local Instance:** It deploys a lightweight Argo CD instance (Application Controller, Repo Server, and Redis) directly on the spoke.
* **Local Reconciliation:** The managed cluster pulls manifests directly from the Git source. This ensures that reconciliation continues even if the network connection to the Hub is lost (disconnected/edge scenarios).
* **Scalability:** By offloading the synchronization process to the spokes, the Hub can manage thousands of clusters without the performance bottlenecks associated with centralized processing.

---

## 2. Installation Guide (Basic Pull Model)

The installation is declarative and managed via resources defined on the Hub cluster.

### Step 1: Enable the Addon
Apply the `ManagedClusterAddOn` resource to the Hub in the namespace corresponding to your managed cluster.

```yaml
apiVersion: addon.open-cluster-management.io/v1alpha1
kind: ManagedClusterAddOn
metadata:
  name: gitops-addon
  namespace: <managed-cluster-name> 
spec:
  installNamespace: open-cluster-management-agent-addon
