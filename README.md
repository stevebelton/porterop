# Porter Operator Installtion (June '23)
(https://github.com/getporter/operator/blob/main/docs/content/quickstart/_index.md


**Install Porter v1**
https://getporter.org/quickstart/

- export VERSION="v1.0.1"
- curl -L https://cdn.porter.sh/$VERSION/install-linux.sh | bash

**Install Kubectl**
https://kubernetes.io/docs/tasks/tools/

- curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
- sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

**Install KinD**
https://kind.sigs.k8s.io/docs/user/quick-start/

- [ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.19.0/kind-linux-amd64
- chmod +x ./kind
- sudo mv ./kind /usr/local/bin/kind

**Create New PorterOperator K8S Cluster**
*(file contents for porteroperator.yaml below*)

NOTE: In order for Porter to successfully connect to KinD/K8S you must add an IP address of the host otherwise it will default to 127.0.0.1 which Porter will not be able to connect to. Be sure to update the IP address shown below, which is an example.

- kind create cluster --config porteroperator.yaml --name porteroperator

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  apiServerAddress: "192.168.72.129"
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
- role: worker
```

**Generate Porter Credential Set**

- porter credentials generate porterops -r ghcr.io/getporter/porter-operator:v0.8.0

```
Generating new credential porterops3 from bundle porter-operator

==> 1 credentials required for bundle porter-operator

? How would you like to set credential "kubeconfig"
  file path
? Enter the path that will be used to set credential "kubeconfig"
  porterops.yaml

(Enter file path and porterops.yaml as responses to prompts)
```

**Create credentials file**

- porter credentials show porterops --output yaml > porterops.yaml

```
schemaType: CredentialSet
schemaVersion: 1.0.1
namespace: ""
name: porterops
credentials:
  - name: kubeconfig
    source:
      path: porterops.yaml
status:
  created: 2023-05-31T19:40:55.533354518-07:00
  modified: 2023-05-31T19:40:55.533354518-07:00

```

- Update the path to point to the full path of your kubectl config file so the file now looks like this:

```
schemaType: CredentialSet
schemaVersion: 1.0.1
namespace: ""
name: porterops
credentials:
  - name: kubeconfig
    source:
      path: /home/nonroot/.kube/config
status:
  created: 2023-05-31T19:40:55.533354518-07:00
  modified: 2023-05-31T19:40:55.533354518-07:00
```

Make sure to update the nonroot to your username.

**Force Set the credential set to use**

- porter credentials apply /home/nonroot/porterops.yaml

**Install Porter Operator**
- porter install porterops -c porterops -r ghcr.io/getporter/porter-operator:v0.8.0

**Check Porter Operator Installation**

- kubectl -n "porter-operator-system" get pods -o wide

```
porter-operator-system   mongodb-5ccfcb958c-jdg8c                             1/1     Running            0               4m32s
porter-operator-system   porter-operator-controller-manager-d79d958fc-z4wsv   1/2     CrashLoopBackOff   4 (61s ago)     4m34s
```

**Create namespace/RBAC/etc**
- porter invoke porterops --action configureNamespace --param namespace=mynamespace

