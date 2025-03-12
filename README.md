# ovn_zt

Enable Istio Ambient to run on specific ovn-k8s UDNs

## Caveats

- This does not work on WSL(2) due to missing `openvswitch` kernel module
 and module loading. So you're better off not wasting time and doing
 development on a physical (or virtual) machine.

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
1. Miscellaneous tools that are used during the build, such as `python3`, 
   `pip`, and `jq` are often preinstalled.  

### Cluster creation with ovn-k8s CNI

Install the [ovn-kubernetes in kind](https://ovn-kubernetes.io/installation/launching-ovn-kubernetes-on-kind/) development environment.
You mush enable `-mne` and `-nse` (short for `--multi-network-enable` and
`--network-segmentation-enable`, respectively) when running the
`contrib/kind.sh` command. The below is a shortcut of the steps - refer
to the original document linked above.

```sh
git clone github.com/ovn-kubernetes/ovn-kubernetes; 
cd ovn-kubernetes
# validate local changes, if any
pushd go-controller
make
popd
# build container image
pushd dist/images
make ubuntu-image
popd
# bring up development environment
pushd contrib
export KUBECONFIG=${HOME}/.kube/ovn.conf
./kind.sh -mne -nse
# or: KUBECONFIG=${HOME}/.kube/ovn.conf ./kind.sh -mne -nse
# if you built the image before, add -ov ovn-kube-ubuntu
# you may also use --deploy for tighter feedback loops when making
# ovn-k8s changes
popd
```

To validate the environment is up:

```sh
# export KUBECONFIG=~/.kube/ovn.conf - if not already set...
kubectl run test-pod --image=busybox --restart=Never --command -- sleep 3600
kubectl run test-pod-2 --image=busybox --restart=Never --command -- sleep 3600
POD2="$(kubectl get pod test-pod --template '{{.status.podIP}}')"
kubectl exec -it test-pod -- ping $POD2
```

#### Error: Externally managed Python environment

As part of the bring up process, `contrib/kind.sh` attempts to install
 the `Jinja2` and `jinjanate` Python modules. This could error on newer
 distributions (especially recent Ubuntu's), with an error message:
 `error: externally-managed-environment ... See PEP 668 for the detailed specification`.

You can workaround it by manually installing the packages before script
 invocation, adding `--break-system-packages` to the `pip` command line.
 A safer alternative is to use Python virtual environments:

```sh
python -m venv ~/.pyvenv
source ~/.pyvenv/bin/activate
# or, if you wish to make this permanent:
# echo "export VIRTUAL_ENV_DISABLE_PROMPT=true" >> ~/.bashrc
# echo "source $HOME/.pyvenv/bin/activate" >> ~/.bashrc
```

## Create UDNs

TBD

## Deploy Ambient

TBD

## Shutdown

```sh
# from ovn-kubernetes/ovn-kubernetes Git directory:
pushd contrib
./kind.sh --delete
popd
```
