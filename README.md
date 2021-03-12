# flux-crossplane
Demo



## pre-requisites

Install `flux`

mac os
```
brew install fluxcd/tap/flux
```

other install [options](https://github.com/fluxcd/flux2)


* have GITHUB_TOKEN if not create one 
Flux is installed in a GitOps way and its manifest will be pushed to the repository, so you will also need a GitHub account and a personal access token that can create repositories (check all permissions under repo) to enable Flux do this.
[documentation](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token)

## Create GCP Account  Keyfile [link](https://crossplane.io/docs/v1.1/getting-started/install-configure.html)

```
# replace this with your own gcp project id and the name of the # service account
# that will be created.
PROJECT_ID=my-project
NEW_SA_NAME=test-service-account-name

# create service account
SA="${NEW_SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"
gcloud iam service-accounts create $NEW_SA_NAME --project $PROJECT_ID

# enable cloud API
SERVICE="sqladmin.googleapis.com"
gcloud services enable $SERVICE --project $PROJECT_ID

# grant access to cloud API
ROLE="roles/cloudsql.admin"
gcloud projects add-iam-policy-binding --role="$ROLE" $PROJECT_ID --member "serviceAccount:$SA"

# create service account keyfile
gcloud iam service-accounts keys create creds.json --project $PROJECT_ID --iam-account $SA
```

## activate gcloud service account.

```
gcloud auth activate-service-account --key-file=credentials.json
```

## initial Flux setup

have a working k8s cluster working can even be local cluster.



#### check state of cluster with flux

```
flux check --pre
```

###### example working 
```
flux check --pre
► checking prerequisites
✔ kubectl 1.20.0 >=1.18.0
✔ Kubernetes 1.20.2 >=1.16.0
✔ prerequisites checks passed
```


#### export this as variables to quick command use.

```
export GITHUB_TOKEN=XXXXXXXXXXXX \ 
export GITHUB_USER=jrab66  \
export GITHUB_REPO=flux-crossplane
```

### run bootstrap flux command:
```

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=$GITHUB_REPO \
  --branch=main \
  --path=./clusters/demo \
  --personal
```


###### example working 
```
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=$GITHUB_REPO \
  --branch=main \
  --path=./clusters/demo \
  --personal
► connecting to github.com
✔ repository cloned
✚ generating manifests
✔ components manifests pushed
► installing components in flux-system namespace
namespace/flux-system created
customresourcedefinition.apiextensions.k8s.io/alerts.notification.toolkit.fluxcd.io created
customresourcedefinition.apiextensions.k8s.io/buckets.source.toolkit.fluxcd.io created
customresourcedefinition.apiextensions.k8s.io/gitrepositories.source.toolkit.fluxcd.io created
customresourcedefinition.apiextensions.k8s.io/helmcharts.source.toolkit.fluxcd.io created
customresourcedefinition.apiextensions.k8s.io/helmreleases.helm.toolkit.fluxcd.io created
customresourcedefinition.apiextensions.k8s.io/helmrepositories.source.toolkit.fluxcd.io created
customresourcedefinition.apiextensions.k8s.io/kustomizations.kustomize.toolkit.fluxcd.io created
customresourcedefinition.apiextensions.k8s.io/providers.notification.toolkit.fluxcd.io created
customresourcedefinition.apiextensions.k8s.io/receivers.notification.toolkit.fluxcd.io created
serviceaccount/helm-controller created
serviceaccount/kustomize-controller created
serviceaccount/notification-controller created
serviceaccount/source-controller created
clusterrole.rbac.authorization.k8s.io/crd-controller-flux-system created
clusterrolebinding.rbac.authorization.k8s.io/cluster-reconciler-flux-system created
clusterrolebinding.rbac.authorization.k8s.io/crd-controller-flux-system created
service/notification-controller created
service/source-controller created
service/webhook-receiver created
deployment.apps/helm-controller created
deployment.apps/kustomize-controller created
deployment.apps/notification-controller created
deployment.apps/source-controller created
networkpolicy.networking.k8s.io/allow-scraping created
networkpolicy.networking.k8s.io/allow-webhooks created
networkpolicy.networking.k8s.io/deny-ingress created
Waiting for deployment "source-controller" rollout to finish: 0 of 1 updated replicas are available...
deployment "source-controller" successfully rolled out
Waiting for deployment "kustomize-controller" rollout to finish: 0 of 1 updated replicas are available...
deployment "kustomize-controller" successfully rolled out
deployment "helm-controller" successfully rolled out
deployment "notification-controller" successfully rolled out
✔ install completed
► configuring deploy key
✔ deploy key configured
► generating sync manifests
✔ sync manifests pushed
► applying sync manifests
◎ waiting for cluster sync
✔ bootstrap finished
```



# add GCP Secret!

```
kubectl create secret generic gcp-creds -n crossplane-system --from-file=creds=credentials.json
```
for now created from command line using `credentials.json`
This can propably need to be automated or handle via mozilla SOPS,etc...
