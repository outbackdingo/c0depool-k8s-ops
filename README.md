# c0depool-k8s-ops

k3s Cluster GitOps managed by Flux + Sops.

## Prerequisites

* Helm v3.10+
* Flux v0.41+
* Sops v3.7+
* Age 1.0+
* kubectl v1.20+
* Tested on k3 kubernetes v1.25.7

## Deployment

- Create age key.

    ```bash
    age-keygen -o sops-key.txt
    ```
- Add the age private key to flux namespace as secret.
    ```bash
    kubectl create ns flux-system
    kubectl -n flux-system create secret generic sops-age --from-file=age.agekey=sops-key.txt
    ```
- Clone the repo.
    ```bash
    git clone https://github.com/c0depool/c0depool-k8s-ops.git
    cd c0depool-k8s-ops
    ```
- Update the repo according to your liking, check apps/ and infrastructure/ and add/remove components. I used [this](https://github.com/fluxcd/flux2-kustomize-helm-example) repo to build the cluster. 
- Update the .sops.yaml with your age pulic key
- Create cluster-secrets.yaml similar to clusters/config/cluster-secrets-encrypted.yaml with the values as non-encrypted base64 encoded values.
    ```bash
    # Example
    echo "mysecretpassword" | base64 -w 0
    # And update cluster-secrets.yaml
    DB_PASSWORD: bXlzZWNyZXRwYXNzd29yZAo=
    ```
- Encrypt your cluster-secrets.yaml using Sops.
    ```bash
    sops --config ~/your-repo/.sops.yaml -e ~/cluster-secrets.yaml | tee ~/your-repo/clusters/config/cluster-secrets-encrypted.yaml
    ```
- Create a new repo and push the code.
    > **Warning**
    > Make sure that your non-encrypted secrets are not pushed to repo
- Bootstrap Flux.
    ```bash
    export GITHUB_TOKEN=<your GitHub token>
    flux bootstrap github \
        --owner=yourgithubname \
        --repository=your-repo \
        --path=clusters/your-cluster \
        --personal
    ```
- Done!