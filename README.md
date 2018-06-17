# Spawnd Infrastructure 

A Kubernetes cluster up and running in ~10 minutes (provisioned with kubeadm)
Flannel networking
Traefik HTTP reverse proxy and load balancer
Cloudflare dynamic DNS configuration
GlusterFS distributed file system

## Instance Types

master: it runs the Kubernetes master, and it optionally acts as an ingress controller proxying from the Internet to the application services through its public IP.
node: it runs a Kubernetes node and it hosts application services.
edge: it is a specialized kind of node with a public IP associated, it acts as an ingress controller proxying from the Internet to the application services. It can run application services as well. Edge nodes are optional.
glusternode: it is a specialized kind of node that runs only a GlusterFS server. One or more glusternodes can be used to provide distributed storage for the application services. Glusternodes are optional.
Cloudflare can be optionally used to setup DNS records and SSL/TSL (HTTPS) encryption.

# Cluster configuration

cluster_prefix: every resource in your tenancy will be named with this prefix
aws_region: the region where your cluster will be bootstrapped (e.g. eu-west-1)
availability_zone: an availability zone for your cluster (e.g. eu-west-1a)
Credentials

aws_access_key_id: your access key id
aws_secret_access_key: your secret access key
Master configuration

master_instance_type: an instance type for the master (e.g. t2.medium)
master_disk_size: edges disk size in GB
Node configuration

node_count: number of Kubernetes nodes to be created
node_instance_type: an instance type for the Kubernetes nodes (e.g. t2.medium)
node_disk_size: edges disk size in GB

# Initialize

Once you are done with your settings you are ready deploy your cluster running:

spawnd apply
To check that your cluster is up and running you can run:

spawnd kubectl get nodes
As long as you are in the my_deployment directory you can use kubectl over SSH to control Kubernetes. If you want to open an interactive SSH terminal onto the master then you can use the kn ssh command:

spawnd ssh

# Deploy your first application

In this guide we are going to deploy a simple application: cheese. We will deploy 3 web pages with a 2 replication factor. The master node will act as a reverse proxy, load balancing the requests among the replicas in the Kubernetes nodes.

The simple cluster that we just deployed uses nip.io as base domain for incoming HTTP traffic. First, we need to figure out our cluster domain by running:

grep domain inventory
The command will return something like domain=37.153.138.137.nip.io, meaning that our cluster domain name in this case would be 37.153.138.137.nip.io.

In Spawnd we encourage to deploy and define SaaS-layer applications using Helm.

spawnd helm install --name --set domain=<your-domain> 
If everything goes well you should be able to access the web pages.


## Traefik reverse proxy

Spawnd uses the Traefik reverse proxy as ingress controller for your cluster. Traefik is installed on one or more nodes, namely edge nodes, which have a public IP associated. In this way, we can access services with a few floating IP without needing LBaaS, which may not be available on certain cloud providers.

In the default setting Spawnd doesn’t deploy any edge node, and it runs Traefik in the master node.

Accessing the Traefik UI

One simple and quick way to access the Traefik UI is to tunnel via SSH to one of the edge nodes with the following command:

ssh -N -f -L localhost:8081:localhost:8081 ubuntu@<your-domain>
Using SSH tunnelling, the Traefik UI should be reachable at http://localhost:8081, and it should look something like this:

../_images/traefik_UI_example.png
In the left side you can find your deployed frontends URLs, whereas on the right side the backend services.

# Clean up

Cloud resources are typically pay-per-use, hence it is good to release them when they are not used. Here we show how to destroy a KubeNow cluster.

To release the resources, please run:

spawnd destroy
