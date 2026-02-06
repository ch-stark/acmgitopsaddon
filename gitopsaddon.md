Here is the complete guide formatted specifically for a Markdown-based document. You can copy and paste this directly into your .md file.

Red Hat Advanced Cluster Management (RHACM) GitOps Addon
The gitops-addon is a lifecycle manager that facilitates the Argo CD Pull Model. This shifts the architectural responsibility of GitOps from a centralized "Push" model on the Hub to a decentralized "Pull" model on managed clusters.

How the GitOps Addon Works
Instead of the Hub cluster maintaining connections to every managed cluster to push changes, the addon decentralizes the operation:

Operator Management: When enabled, the addon automatically installs the OpenShift GitOps Operator on the managed cluster.

Local Instance: It deploys a lightweight Argo CD instance (Application Controller, Repo Server, and Redis) directly on the spoke.

Local Reconciliation: The managed cluster pulls manifests directly from Git. This allows for reconciliation to continue even if the connection to the Hub is lost.

Scalability: By offloading the sync process to the spokes, the Hub can manage a significantly higher number of clusters without performance bottlenecks.

Installation Guide
The installation is declarative and managed via resources on the Hub cluster.

1. Enable the Addon
Apply the ManagedClusterAddOn to the Hub in the namespace of your managed cluster.

YAML

apiVersion: addon.open-cluster-management.io/v1alpha1
kind: ManagedClusterAddOn
metadata:
  name: gitops-addon
  namespace: <managed-cluster-name> 
spec:
  installNamespace: open-cluster-management-agent-addon
2. Verification
Check the status on the Managed Cluster:

Addon Agent: oc get pods -n open-cluster-management-agent-addon

Argo CD Components: oc get pods -n openshift-gitops

Configuration & Customization
Use an AddOnDeploymentConfig to customize settings like the reconciliation scope or specific image versions.

Step A: Create the Configuration
Create this resource on the Hub:

YAML

apiVersion: addon.open-cluster-management.io/v1alpha1
kind: AddOnDeploymentConfig
metadata:
  name: gitops-addon-config
  namespace: open-cluster-management
spec:
  customizedVariables:
  - name: RECONCILE_SCOPE
    value: All-Namespaces # Alternatives: 'Single-Namespace'
Step B: Bind the Configuration
Associate this config with the ClusterManagementAddOn on the Hub:

YAML

apiVersion: addon.open-cluster-management.io/v1alpha1
kind: ClusterManagementAddOn
metadata:
  name: gitops-addon
spec:
  supportedConfigs:
  - group: addon.open-cluster-management.io
    resource: addondeploymentconfigs
    defaultConfig:
      name: gitops-addon-config
      namespace: open-cluster-management
Advanced: The Argo CD Agent (Pull Model Evolution)
For high-scale environments, the "new" architecture uses the GitOpsCluster resource to automate the registration of the agent via mTLS.

YAML

apiVersion: apps.open-cluster-management.io/v1alpha1
kind: GitOpsCluster
metadata:
  name: my-gitops-cluster
  namespace: open-cluster-management
spec:
  placementRef:
    kind: Placement
    name: cluster-selection-policy
  argoCDAgentAddon:
    mode: managed
