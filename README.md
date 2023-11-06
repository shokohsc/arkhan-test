# k3d osx m1 silicon

## Requirements

- [k3d](https://k3d.io/v5.6.0/#install-script)
- [jq](https://jqlang.github.io/jq/download/)

#### create k3d cluster from configuration
    $ k3d cluster create --config config.yaml

#### install dependencies
    $ helm repo add hashicorp https://helm.releases.hashicorp.com
    $ helm repo add traefik https://traefik.github.io/charts
    $ helm repo add crossplane-stable https://charts.crossplane.io/stable

    $ helm install vault hashicorp/vault --version 0.26.0 --namespace vault --create-namespace -f vault-values.yaml
    $ kubectl exec -n vault vault-0 -- vault operator init \
        -key-shares=1 \
        -key-threshold=1 \
        -format=json > cluster-keys.json
    $ VAULT_UNSEAL_KEY=$(jq -r ".unseal_keys_b64[]" cluster-keys.json)
    $ kubectl exec -n vault vault-0 -- vault operator unseal $VAULT_UNSEAL_KEY
    $ VAULT_ROOK_TOKEN=$(jq -r ".root_token" cluster-keys.json)

    $ helm install crossplane crossplane-stable/crossplane --version 1.13.2 --namespace crossplane-system --create-namespace -f crossplane-values.yaml

    $ helm install traefik traefik/traefik --version 25.0.0 --namespace traefik --create-namespace -f traefik-values.yaml

    $ kubectl apply -k ./

#### delete k3d cluster
    $ k3d cluster delete -a

## Helper commmands

    $ k9s
    $ kube cluster-info
