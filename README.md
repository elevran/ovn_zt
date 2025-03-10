# ovn_zt

Enable Istio Ambient to run on specific ovn-k8s UDNs

## Prerequisites

### Requitred tool instllation
Ensure you have the following installed on your system:

1. Docker: Install based on instructions [here](https://docs.docker.com/engine/install/)
1. Kind: Install via:
    ```sh
    curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
    chmod +x kind
    sudo mv kind /usr/local/bin/
    ```
1. kubectl: Install via:
    ```bash
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x kubectl
    sudo mv kubectl /usr/local/bin/
    ```
1. Install CNI Plugins (optional, but recommended):
    ```sh
    sudo mkdir -p /opt/cni/bin
    curl -L -o /tmp/cni-plugins.tgz https://github.com/containernetworking/plugins/releases/latest/download/cni-plugins-linux-amd64.tgz
    sudo tar -C /opt/cni/bin -xzf /tmp/cni-plugins.tgz
    ```
    > if downloads fails (or tar complains about not finding a zip format), it's likely
    that `latest` release is not correctly configured on github and you should download
    a specific release number (e.g., `v1.6.2` at the time of writing) instead of `latest`.

### Cluster creation with ovn-k8s CNI

1. Create a `kind-config.yaml` file with the following content to create a 2 worker
 cluster, disabling the default CNI (we'll install ovn-k8s afterwards):
    ```yaml
    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    name: ovn-zt
    networking:
    disableDefaultCNI: true
    kubeProxyMode: "none"
    nodes:
    - role: control-plane
    - role: worker
    - role: worker
    ```
1. Run `kind` to create the cluster:
    ```sh
    kind create cluster --config kind-config.yaml
    Creating cluster "ovn-zt" ...
    ✓ Ensuring node image (kindest/node:v1.32.2) 🖼
    ✓ Preparing nodes 📦 📦 📦
    ✓ Writing configuration 📜
    ✓ Starting control-plane 🕹️
    ✓ Installing StorageClass 💾
    ✓ Joining worker nodes 🚜
    Set kubectl context to "kind-ovn-zt"
    You can now use your cluster with:

    kubectl cluster-info --context kind-ovn-zt

    Have a nice day! 👋
    ```
1. Verify the cluster is up and running:
    ```sh
    kubectl get nodes
    NAME                   STATUS     ROLES           AGE     VERSION
    ovn-zt-control-plane   NotReady   control-plane   10m     v1.32.2
    ovn-zt-worker          NotReady   <none>          9m58s   v1.32.2
    ovn-zt-worker2         NotReady   <none>          9m58s   v1.32.2
    ```
1. Install [ovn-kubernetes in kind](https://ovn-kubernetes.io/installation/launching-ovn-kubernetes-on-kind/)
