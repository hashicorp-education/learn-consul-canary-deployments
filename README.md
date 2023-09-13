# Learn Consul - Canary deployments with service splitters

This is a companion repo to the [Deploy seamless canary deployments with service splitters](https://developer.hashicorp.com/consul/tutorials/developer-mesh/service-splitters-canary-deployment), containing sample configuration to:

- Deploy an Elastic Kubernetes Service (EKS) cluster with Terraform
- Deploy Consul to EKS cluster
- Deploy example applications
- Create service subsets with service resolvers
- Send partial traffic to new service
- Send all traffic to new service