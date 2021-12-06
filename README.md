# CKS-test-rbac
show difference between `list` and `get` calls on secrets.
* `kc create namespace test-rbac -oyaml > test-rbac.ns.yaml`

* `kc -n test-rbac create serviceaccount allow-list --dry-run -oyaml > allow-list.sa.yaml`
* `kc -n test-rbac create role allow-list --verb=list --resource=secrets --dry-run -oyaml > allow-list.role.yaml`
* `kc -n test-rbac create rolebinding allow-list --role=allow-list --serviceaccount=test-rbac:allow-list --dry-run > allow-list.rb.yaml

* `kc -n test-rbac create serviceaccount allow-get --dry-run -oyaml > allow-get.sa.yaml`
* `kc -n test-rbac create role allow-get --verb=get --resource=secrets --dry-run -oyaml > allow-get.role.yaml`
* `kc -n test-rbac create rolebinding allow-get --role=allow-get --serviceaccount=test-rbac:allow-get --dry-run > allow-get.rb.yaml`

* `kc -n test-rbac run --generator=run-pod/v1 --image bitnami/kubectl:latest --serviceaccount allow-list allow-list --dry-run -oyaml --command -- sleep 3600 > allow-list.pod.yaml`
* `kc -n test-rbac run --generator=run-pod/v1 --image bitnami/kubectl:latest --serviceaccount allow-get allow-get --dry-run -oyaml --command -- sleep 3600 > allow-get.pod.yaml`

## allow-list

```
$ kc -n test-rbac exec allow-list kubectl get secrets
NAME                     TYPE                                  DATA   AGE
allow-get-token-vzb9p    kubernetes.io/service-account-token   3      3m31s
allow-list-token-hrwcj   kubernetes.io/service-account-token   3      3m30s
default-token-9kw2v      kubernetes.io/service-account-token   3      9m7s
$ kc -n test-rbac exec allow-list -- kubectl get secrets default-token-9kw2v -oyaml
Error from server (Forbidden): secrets "default-token-9kw2v" is forbidden: User "system:serviceaccount:test-rbac:allow-list" cannot get resource "secrets" in API group "" in the namespace "test-rbac"
command terminated with exit code 1
```
## allow-get
```
$ kc -n test-rbac exec allow-get -- kubectl get secrets
Error from server (Forbidden): secrets is forbidden: User "system:serviceaccount:test-rbac:allow-get" cannot list resource "secrets" in API group "" in the namespace "test-rbac"
command terminated with exit code 1
$ kc -n test-rbac exec allow-get -- kubectl get secrets default-token-9kw2v -oyaml
apiVersion: v1
data:
  ca.crt: [...]
  namespace: dGVzdC1yYmFj
  token: [...]
kind: Secret
metadata:
  annotations:
    kubernetes.io/service-account.name: default
    kubernetes.io/service-account.uid: 9e00e332-2ca1-42f9-9903-adb37f85ad50
  creationTimestamp: "2021-12-06T13:08:50Z"
  name: default-token-9kw2v
  namespace: test-rbac
  resourceVersion: "2024253"
  uid: b516eae1-9180-45ea-941e-015d94d21edd
type: kubernetes.io/service-account-token
```
