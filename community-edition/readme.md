
# what this is
a free tool to deploying a hubs system on kubernetes
- enables anyone to host hubs on a variety of infrastructures, from closet server to public close and anything in between
- can be productionized with some modifications (read -- suggestions to make it production ready)
 
# what it is not 
a "turn key" solution for a production ready hubs system.
- flexibility/compatibility and production ready don't mix, it's not possible for one architectural design to simultaneously support average joe/jane's laptop and their company's production environment. 
- Designing, building, hosting, and maintaining production software systems goes beyond the scope of a single codebase. It encompasses both business aspects, such as budgeting and scoping, and engineering factors like security, reliability, scalability, and efficiency. Many books have been written on this subject.

# why kubernetes
- [portable, extensible, open source](https://kubernetes.io/docs/concepts/overview/)
- [available in many clouds](https://kubernetes.io/docs/setup/production-environment/turnkey-solutions/)
- many single server options as well (ie. minikube, k3s, microk8s, kind)

# how to deploy
- pre requisites
    - kubernetes
    - DNS service (to reach hubs services on a domain)
    - ports: (expose services to client)
        - tcp: 80,443,4443,5349
        - udp: 35000 - 60000
    - smtp service (for login emails, ie. [use gmail as smtp](https://support.google.com/a/answer/176600?hl=en))
- deploy to kubernetes
    - edit configs into `render_hcce.sh` with a text editor
    - `bash render_hcce.sh && kubectl apply -f hcce.yaml`
- expose the services
    - use `kubectl -n <hcce_namespace> get svc lb` to find it's external ip
    - on your dns service, create A-records to route below domains to the external ip of lb service in hcce namespace
        - <root_domain>
        - assets.<root_domain>
        - stream.<root_domain>
        - cors.<root_domain>

- https certs
    - option#1: bring your own
        - package the certs into kubernetes secrets named `cert-<domain>` under the deploy namespace
    - option#2: use certbotbot
        - edit configs into `cbb.sh` with a text editor
        - `bash cbb.sh`

# example -- a "hello-world" instance with vm on gcp
### make a kubernetes environment
replace `hcce-vm-1` and `us-central1-a` with your desired name and zone, check [official doc](https://cloud.google.com/sdk/gcloud/reference/compute/instances/create) for more options
```
### login gcp
gcloud auth login
### create a vm
`gcloud compute instances create hcce-vm-1 --zone=us-central1-a`
### ssh to the vm
`gcloud compute ssh --project=hubs-dev-333333 --zone=us-central1-a geng-test-2`
### prepare the vm
`sudo apt update && sudo apt install npm && sudo npm install pem-jwk -g`
### install k3s without traefik -- read https://docs.k3s.io/ for more info
- `curl https://get.k3s.io/ | INSTALL_K3S_EXEC="--disable=traefik" sh -`
```

## deploy to kubernetes
`bash render_hcce.sh && sudo k3s kubectl apply -f hcce.yaml`

## connect the ingress
- find the vm's external ip
- dns
- firewall

# example -- a "hello-world" instance with managed kubernetes on gcp
### make a kubernetes environment
replace `hcce-gke-1` and `us-central1-a` with your desired name and zone, check [official doc](https://cloud.google.com/sdk/gcloud/reference/container/clusters/create) for more options
```
# login gcp
gcloud auth login
# create gke cluster
gcloud container clusters create hcce-gke-1 --zone=us-central1-a
# get creds for kubectl
gcloud container clusters get-credentials --region us-central1-a hcce-gke-1
```

## deploy to kubernetes
`bash render_hcce.sh && sudo k3s kubectl apply -f hcce.yaml`

### connect the ingress
- find the external ip with `kubectl -n hcce get svc lb`
- dns and firewall steps are the same <link to above>


# considerations for production environment
- infra
    - easy -- use a managed kubernetes
    - hard -- https://kubernetes.io/docs/setup/production-environment/
- security
    - password and keys overview
    - add a waf
- scalability
    - stateful services
        - pgsql 
            - use a managed pgsql ie. rds on aws or cloudsql on gcp
            - roll your own
                - <links to some guides to run pgsql in k8s>
        - reticulum
            - use a network/shared storage for reticulum's /storage mount
    - stateless services (all except reticulum and pgsql)
        - just run multiple replicas
        - use hpa
- devops
    - the yaml file is the entire infra on kubernetes, use a git to track changes and an ops pipeline to auto deploy
        - ie put the yaml file on a github repo and use github action to deploy to your hosting env
    - use dev env for staging/testing
        - use spot instances for nodes to save $
        - develop and integrate automated testing scripts into the ops pipeline

# example -- a "potentially-somewhat-production-ready" instance on aws
- comming soon
