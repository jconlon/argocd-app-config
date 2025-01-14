# Superset 

*TODO: Rework this readme to reflect changes related to using ArgoCD.*

*Note: The Superset metadata store: `postgresql` requires a storage service with persistent volumns.  Install the `OpenEBS` helm charts prior to this Superset install.* 

[OpenEBS](https://openebs.io/)

## Install OpenEBS

Helm installation is just using the: [Local PV provisioner](https://github.com/openebs/dynamic-localpv-provisioner/blob/HEAD/deploy/helm/charts/values.yaml) 

```bash
cd custom

helm repo add openebs https://openebs.github.io/openebs

helm repo update

helm upgrade --install openebs openebs/openebs --namespace openebs --create-namespace -f openebs-custom-values.yaml

```
Check the storageclass

```bash
$: k get storageclass 
NAME               PROVISIONER        RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-hostpath   openebs.io/local   Delete          WaitForFirstConsumer   false                  15h

$ kubectl describe storageclass openebs-hostpath 
Name:            openebs-hostpath
IsDefaultClass:  No
Annotations:     cas.openebs.io/config=- name: StorageType
  value: "hostpath"
- name: BasePath
  value: "/var/openebs/local"
,meta.helm.sh/release-name=openebs,meta.helm.sh/release-namespace=openebs,openebs.io/cas-type=local
Provisioner:           openebs.io/local
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>
```

If it is not default set it to be:

```bash

kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## Superset Installation

```bash
helm install my-superset superset/superset --namespace superset -f superset-custom-values.yaml --create-namespace # --dry-run --debug

helm upgrade my-superset superset/superset --namespace superset -f custom/superset-custom-values.yaml 

helm upgrade --wait --timeout 30m my-superset superset/superset --namespace superset -f custom/superset-custom-values.yaml
```

## Troubleshooting

### Network Connectivity

```bash
kubectl apply -f troubleshoot-pod.yaml

kubectl exec -it troubleshoot-pod -- /bin/bash

```
Once inside:

To test a service, you can use commands like:

```bash
curl http://service-name:port

curl http://my-superset-postgresql:5432

nc -zv service-name port

nc -zv my-superset-postgresql 5432

```

#### Setup listerner and tester

```bash

kubectl apply -f listener-deployment.yaml
kubectl apply -f listener-service.yaml

```



#### Commands

```bash
# install ArgoCD in k8s
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# access ArgoCD UI
kubectl get svc -n argocd
kubectl port-forward svc/argocd-server 8080:443 -n argocd

# login with admin user and below token (as in documentation):
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo

# you can change and delete init password

```
</br>

#### Links

* Config repo: [https://gitlab.com/nanuchi/argocd-app-config](https://gitlab.com/nanuchi/argocd-app-config)

* Docker repo: [https://hub.docker.com/repository/docker/nanajanashia/argocd-app](https://hub.docker.com/repository/docker/nanajanashia/argocd-app)

* Install ArgoCD: [https://argo-cd.readthedocs.io/en/stable/getting_started/#1-install-argo-cd](https://argo-cd.readthedocs.io/en/stable/getting_started/#1-install-argo-cd)

* Login to ArgoCD: [https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli](https://argo-cd.readthedocs.io/en/stable/getting_started/#4-login-using-the-cli)

* ArgoCD Configuration: [https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/)


