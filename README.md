# learn-consul-canary-deployments
A companion repository for the 'Deploy seamless canary deployments with service splitters' tutorial

# Runbook

terraform init
terraform apply
aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -raw cluster_id)

consul-k8s install -config-file=k8s-yamls/consul-helm.yaml

kubectl get svc -n consul # open the consul UI

for deployment in {frontend-service.yaml,frontend-v1.yaml,intentions.yaml,nginx.yaml}; do kubectl apply -f helloworld/$deployment; done; kubectl wait --for=condition=ready pod -l app=frontend

kubectl port-forward deployments/nginx 8000:80
curl http://localhost:8000/

kubectl apply -f k8s-yamls/service-resolver.yaml

kubectl apply -f k8s-yamls/service-splitter-v1-only.yaml

kubectl apply -f helloworld/frontend-v2.yaml
curl http://localhost:8000/

kubectl apply -f k8s-yamls/service-splitter-50-50.yaml
curl http://localhost:8000/

kubectl apply -f k8s-yamls/service-splitter-v2-only.yaml
curl http://localhost:8000/

kubectl delete -f helloworld/frontend-v1.yaml
kubectl delete -f k8s-yamls/service-splitter-v2-only.yaml
kubectl delete -f k8s-yamls/service-resolver.yaml
