# learn-consul-canary-deployments

A companion repository for the 'Deploy seamless canary deployments with service splitters' tutorial

# Runbook

```
terraform init
terraform apply
```

```
Plan: 51 to add, 0 to change, 0 to destroy.

Changes to Outputs:

cluster_endpoint = "https://F00E5DAC55DA150F5305D04E78732BA4.gr7.us-east-2.eks.amazonaws.com"
cluster_id = "learn-canary-b5RK3phA"
cluster_name = "learn-canary-b5RK3phA"
cluster_security_group_id = "sg-0fe7e166b1335c22b"
region = "us-east-2"
```

```
aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -raw cluster_id)
```

```
consul-k8s install -config-file=k8s-yamls/consul-helm.yaml
```

```
kubectl apply --filename api-gw/consul-api-gateway.yaml && \
  kubectl wait --for=condition=accepted gateway/api-gateway --namespace=consul --timeout=90s && \
  kubectl apply --filename api-gw/referencegrant.yaml && \
  kubectl apply --filename api-gw/rbac.yaml && \
  kubectl apply --filename api-gw/ingress.yaml
```

```
kubectl get svc -n consul # open the consul UI
```


```
export CONSUL_HTTP_ADDR="https://$(kubectl get service consul-ui --output jsonpath='{.status.loadBalancer.ingress[0].hostname}' -n consul)"
```

```
kubectl apply -f hashicups
```

```
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
frontend-84999dc698-hz9cq         2/2     Running   0          33s
frontend-v2-6d896675bf-24bjn      2/2     Running   0          33s
nginx-84985894bd-xmq2p            2/2     Running   0          31s
payments-b4f5c6c58-wpmwn          2/2     Running   0          30s
product-api-74c5f98f64-rwshf      2/2     Running   0          29s
product-api-db-6c49b5dcb4-kcrk4   2/2     Running   0          30s
public-api-5dc47dd74-skc2d        3/3     Running   0          28s
```

```
export APIGW_URL=$(kubectl get services --namespace=consul api-gateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}') && echo $APIGW_URL
```

```
$ curl --insecure $CONSUL_HTTP_ADDR/v1/catalog/service/frontend?pretty
[
  {
    "Node": "ip-10-0-1-222.us-east-2.compute.internal-virtual",
    "Address": "10.0.1.222",
    "Datacenter": "dc1",
    "ServiceID": "frontend-84999dc698-hz9cq-frontend",
    "ServiceName": "frontend",
    "ServiceTags": [
      "v1"
    ],
    "ServiceAddress": "10.0.1.95",
    "ServiceTaggedAddresses": {
      "virtual": {
        "Address": "172.20.73.164",
        "Port": 3000
      }
    }
  },
  {
    "Node": "ip-10-0-3-239.us-east-2.compute.internal-virtual",
    "Address": "10.0.3.239",
    "Datacenter": "dc1",
    "ServiceID": "frontend-v2-6d896675bf-24bjn-frontend",
    "ServiceName": "frontend",
    "ServiceTags": [
      "v2"
    ],
    "ServiceAddress": "10.0.3.209",
    "ServiceTaggedAddresses": {
      "virtual": {
        "Address": "172.20.73.164",
        "Port": 3000
      }
    }
  }
]
```

```
kubectl port-forward deployments/nginx 8000:80
```

```
kubectl apply -f k8s-yamls/service-resolver.yaml
```

```
kubectl apply -f k8s-yamls/service-splitter-v1-only.yaml
```

```
kubectl apply -f k8s-yamls/service-splitter-50-50.yaml
```

```
kubectl apply -f k8s-yamls/service-splitter-v2-only.yaml
```

kubectl delete -f hashicups/frontend-v1.yaml
kubectl delete -f k8s-yamls/service-splitter-v2-only.yaml
kubectl delete -f k8s-yamls/service-resolver.yaml


for i in `seq 1 10`; do echo -n "$i. " && curl -s http://localhost:8000/ | sed -n 's/.*\(HashiCups-v1\).*/\1/p;s/.*\(HashiCups-v2\).*/\1/p' && echo ""; done