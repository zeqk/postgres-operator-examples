## Install Operator

https://access.crunchydata.com/documentation/postgres-operator/v5/installation/helm/

```bash
kubectl apply -f 01-namespace.yml
helm install --namespace postgres-operator postgres-operator -f ./pgo-values.yml helm/install
kubectl get all -n postgres-operator
kubectl -n postgres-operator get pods \
  --selector=postgres-operator.crunchydata.com/control-plane=pgo \
  --field-selector=status.phase=Running
```

```bash
helm uninstall --namespace postgres-operator postgres-operator
```

## Create PG Cluster

```bash
kubectl apply -k kustomize/postgres
kubectl describe postgresclusters.postgres-operator.crunchydata.com hippo
```

```bash
PG_CLUSTER_PRIMARY_POD=$(kubectl get pod -o name \
  -l postgres-operator.crunchydata.com/cluster=hippo,postgres-operator.crunchydata.com/role=master)
PG_CLUSTER_USER_SECRET_NAME=hippo-pguser-hippo
kubectl get secrets "${PG_CLUSTER_USER_SECRET_NAME}" -o go-template='{{.data.user | base64decode}}'
kubectl get secrets "${PG_CLUSTER_USER_SECRET_NAME}" -o go-template='{{.data.password | base64decode}}'
kubectl get secrets "${PG_CLUSTER_USER_SECRET_NAME}" -o go-template='{{.data.dbname | base64decode}}'

kubectl port-forward "${PG_CLUSTER_PRIMARY_POD}" 5432:5432

docker run -it --rm postgres psql -h host.docker.internal -U hippo
```

https://access.crunchydata.com/documentation/postgres-operator/5.2.0/architecture/pgadmin4/

```bash
kubectl port-forward svc/hippo-pgadmin 5050:5050
```

