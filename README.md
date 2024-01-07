# Virtual Service Route Controller

Construct Istio `VirtualService` resource from some components.

:writing_hand: This is study about custom controller.

## Usage
1. Create `VirtualServiceBase` resource.

```bash
$ cat <<EOF | kubectl apply -f -
apiVersion: virtualservicecomponent.shibataka000.com/v1alpha1
kind: VirtualServiceBase
metadata:
  name: virtualservicebase-sample
spec:
  gateways:
  - gateway
  hosts:
  - '*'
EOF
```

2. Create `HTTPRouteBinding` resources.

```bash
$ cat <<EOF | kubectl apply -f -
apiVersion: virtualservicecomponent.shibataka000.com/v1alpha1
kind: HTTPRouteBinding
metadata:
  name: httproutebinding-sample-1
spec:
  virtualServiceBaseRef:
    apiVersion: virtualservicecomponent.shibataka000.com/v1alpha1
    kind: VirtualServiceBase
    name: virtualservicebase-sample
    namespace: default
  httpRoute:
    match:
    - headers:
        key:
          exact: mysubset-1
    route:
    - destination:
        host: myhost
        subset: mysubset-1
EOF

$ cat <<EOF | kubectl apply -f -
apiVersion: virtualservicecomponent.shibataka000.com/v1alpha1
kind: HTTPRouteBinding
metadata:
  name: httproutebinding-sample-2
spec:
  virtualServiceBaseRef:
    apiVersion: virtualservicecomponent.shibataka000.com/v1alpha1
    kind: VirtualServiceBase
    name: virtualservicebase-sample
    namespace: default
  httpRoute:
    match:
    - headers:
        key:
          exact: mysubset-2
    route:
    - destination:
        host: myhost
        subset: mysubset-2
EOF
```

3. VirtualServiceRouteController construct `VirtualService` resource from `VirtualServiceBase` and `HTTPRouteBinding` . You can see created resource as follows.

```bash
$ kubectl get virtualservices.networking.istio.io virtualservicebase-sample -o yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: virtualservicebase-sample
  namespace: default
spec:
  gateways:
  - gateway
  hosts:
  - '*'
  http:
  - match:
    - headers:
        key:
          exact: mysubset-1
    route:
    - destination:
        host: myhost
        subset: mysubset-1
  - match:
    - headers:
        key:
          exact: mysubset-2
    route:
    - destination:
        host: myhost
        subset: mysubset-2
```

## Requirement
1. Install [Istio](https://istio.io/)

2. Install dependencies: controller-gen & kustomize 
```bash
go install sigs.k8s.io/controller-tools/cmd/controller-gen@v0.4.1
brew install kustomize
```

## Deploy to cluster
1. Build operator

```bash
make
```

2. Deploy VirtualServiceRouteController operator to your kubernetes cluster.

```bash
make install
make deploy
```

## Running locally
This can be done after

1. Delete deployment of the operator
```bash
NS=vsr-controller-system
kubectl delete deployments $(kubectl get deployments -o=jsonpath="{.items[*].metadata.name} -n $N" -n $NS)
```

2. Run in shell
```bash
make run
```