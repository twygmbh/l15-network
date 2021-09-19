# Network - GitOps

> :warning: **Docs are WIP**

## GitOps

This repo is used for the GitOps of the [L15/l16 clusters] running in the respective GCP projects.

[FluxCD](https://fluxcd.io/) is running on the cluster and is applying the changes of this repository to the clusters.

### Usefull commands

Install the [FluxCLI](https://fluxcd.io/docs/cmd/)

Get release statuses: `flux get helmreleases --all-namespaces`

## Secrets

> :warning: You will need to have the kubeseal cli installed:
> https://fluxcd.io/docs/guides/sealed-secrets/#prerequisites

When a team member wants to create a kubernetes secret, they use kubeseal and the public key located in the root directory `./pub-sealed-secrets.pem` (not yet here, coming soon)

Assuming a team member wants to deploy an application that needs to connect to a database using a username and password, they’ll be doing the following:

- create a kubernetes secret manifest locally with the db credentials e.g. credentials.yaml

  ```bash
  kubectl -n default create secret generic credentials \
  --from-literal=user=admin \
  --from-literal=password=change-me \
  --dry-run=true \
  -o yaml > credentials.yaml
  ```

- encrypt the secret with kubeseal as credentials-sealed.yaml
  ```
  kubeseal --format=yaml --cert=pub-sealed-secrets.pem \
  < credentials.yaml > credentials-sealed.yaml
  ```
- delete the original secret file credentials.yaml
- create a kubernetes deployment manifest for the app e.g. app-deployment.yaml
- add the Secret to the deployment manifest as a volume mount or env var using the original name credentials
- commit the manifests credentials-sealed.yaml and app-deployment.yaml to a Git repository that’s being synced by the GitOps toolkit controllers

Once the manifests have been pushed to the Git repository, the following happens:

- source-controller pulls the changes from Git
- kustomize-controller applies the SealedSecret and the deployment manifests
- sealed-secrets controller decrypts the SealedSecret and creates a kubernetes secret
- kubelet creates the pods and mounts the secret as a volume or env variable inside the app container
