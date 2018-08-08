# demo_DataPipeline
**This repository is intended for internal Mesosphere staff only**

Mesosphere repository for Data Pipeline workshops featuring DC/OS, Kubernetes, Kubeflow, TensorFlow, Jupyter, Spark, Portworx, HDFS, Kafka. 

You will need to provision a Mesosphere DC/OS Enterprise Edition cluster in `permissive` mode.

A DC/OS cluster with at least 3 Masters, 1 Public Agent, and 8 Private Agents is required for an aggregate of 36 CPU, 132 GiB Memory, and 1 TiB storage.


## High level steps
1. (1,2) Prepare/Cleanse & Explore data
    - Bring up Jupyter notebook
    - Run Spark job on Portworx-HDFS data from Jupyter notebook
2. (3) Model training
    - Run TensorFlow single-node job
3. (4,5) Monitoring, Debugging
4. (6) Model serving 
    - Run kubeflow job on K8s on DC/OS
5. (7) Streaming of Requests
    - Portworx-Kafka for streams


### DC/OS
  - Authenticate CLI using MAWS
```
maws login [AWS user account]
```
ref: https://github.com/mesosphere/terraform-dcos-enterprise/blob/master/aws/README.md

  - Deploy a DC/OS cluster 12 pvt nodes 1 public using Terraform
```
cd dcos-installer
terraform apply -var-file desired_cluster_profile.tfvars
```

### Portworx
  - AWS: use Tags, and create & attach EBS volumes to all participating private nodes.
  - Install Portworx: enable etcd, lighthouse. Set node count to quantity of nodes in your cluster.
  - Deploy Repoxy (https://docs.portworx.com/scheduler/mesosphere-dcos/lighthouse-marathon.html#accessing-lighthouse)
  - UI: 
    - http://PublicAgentIP:9999
    - portworx@yourcompany.com / admin


### Packages
  - Deploy Kubernetes on DC/OS, HA, 3 nodes.
```
dcos package install kubernetes
```
  - Deploy MLB.
```
dcos package install marathon-lb
```
  - Deploy Portworx-Hadoop on DC/OS
```
dcos package install portworx-hadoop
```


### Kubernetes
  - Expose k8s API
```
dcos marathon app add kubectl-proxy.json
```
  - Find public agent IP
```
dcos kubernetes kubeconfig --apiserver-url https://PubAgentIP:6443 --insecure-skip-tls-verify
```

### Kubeflow
#### Deploy Jupyterlab
```
dcos package install jupyterlab
```
    - Networking: use the public agent ELB address
    - Access: http://**publicAgentIP**:10104/jupyterlab-notebook/login
    - password: jupyter
    - enable Start Tensorboard

#### Jupyterlab - run test Spark job in Terminal
  - Terminal
  - [Github repo](https://github.com/dcos-labs/dcos-jupyterlab-service/blob/master/DEPLOY-STRICT.md)
    - Paste into Jupyterlab Terminal: cmd under Submit a test SparkPi Job section
    - Clone the Yahoo TensorFlowOnSpark Github Repo
    - Prepare MNIST Dataset in CSV format and store on S3
    - Prepare MNIST Dataset in CSV format and store on HDFS
    - Show available endpoints in portworx-hadoop, using the Jupyterlab Terminal
    - DCOS > Services > Environment > Jupyter Conf Urls
      - Jupyter Terminal: 
      - curl http://api.portworx-hadoop.marathon.l4lb.thisdcos.directory/v1/endpoints


### Deploy ksonnet
```
brew install ksonnet/tap/ks
```

### Deploy Kubeflow
https://www.kubeflow.org/docs/about/user_guide/

### TensorBoard
* https://www.tensorflow.org/guide/summaries_and_tensorboard


### Deploy Portworx-Kafka on DC/OS
```
dcos package install portworx-kafka
```


### Clean Up
```
terraform destroy -var-file desired_cluster_profile.tfvars
```
* Manual AWS cleanup for EBS volumes

















### Resources, made possible by and/or for:
* [Mesosphere DC/OS](https://dcos.io)
* [Mesosphere DC/OS Enterprise](https://mesosphere.com/product)
* [Joerg Schad](https://github.com/joerg84)
* [Jupyter Notebooks with the BeakerX JVM Kernels on Mesosphere DC/OS](https://github.com/dcos-labs/dcos-jupyterlab-service/blob/master/DEPLOY-STRICT.md)
* [Kubeflow](https://www.kubeflow.org)
* [Portworx](https://www.portworx.com)
