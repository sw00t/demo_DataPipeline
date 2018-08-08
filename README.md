# demo_DataPipeline
**This repository is for internal Mesosphere staff only**

## High level steps
![overview](https://i.imgur.com/BEpyVYS.png)
1. Stage 1, 2: Prepare/Cleanse & Explore Data
    - Run a Spark job on Portworx-HDFS data from Jupyter notebook
2. Stage 3: Model Training
    - Run TensorFlow on a job single CPU only node (GPU ideal if available)
3. Stage 4, 5: Monitoring, Debugging
    - TensorBoard, [TFDBG](https://www.tensorflow.org/guide/debugger)
4. Stage 6: Model Serving 
    - Run kubeflow job on Kubernetes on DC/OS
5. Stage 7: Streaming of Requests
    - Portworx-Kafka

This is a Mesosphere repository to be used for Data Pipeline workshops and demos. Featured components include DC/OS, Kubernetes, Kubeflow, TensorFlow, Jupyter, Spark, Portworx, HDFS, Kafka. 

The purpose of this demo is to showcase DC/OS features and capabilities as a platform, to appeal to data scientists, infrastructure, operations, and devops personnel. 

You will need the appropriate permissions in AWS to provision a Mesosphere DC/OS Enterprise cluster using Terraform. 
The DC/OS cluster should have at minimum 3 Masters, 1 Public Agent, and 12 Private Agents.

This is currently in progress, and will continue to be updated until indicated otherwise in this README. Enjoy!


### DC/OS
  - Authenticate CLI using MAWS
```
maws login [AWS user account]
```
  - Deploy a DC/OS cluster 12 pvt nodes 1 public using Terraform
```
dcos_version = "1.11.4"
num_of_masters = "3"
num_of_private_agents = "12"
num_of_public_agents = "1"
#
os = "centos_7.4"
aws_profile = "<Mesosphere AWS user account name>"
aws_region = "us-west-2"
aws_bootstrap_instance_type = "m3.large"
aws_master_instance_type = "m4.xlarge"
aws_agent_instance_type = "m4.xlarge"
aws_public_agent_instance_type = "m4.xlarge"
ssh_key_name = "default"
# Inbound Master Access
admin_cidr = "0.0.0.0/0"
dcos_exhibitor_storage_backend = "aws_s3"
dcos_exhibitor_explicit_keys = "false"
dcos_master_discovery = "master_http_loadbalancer"
dcos_license_key_contents = "<DC/OS license>"
```
```
cd dcos-installer
terraform apply -var-file desired_cluster_profile.tfvars
```

### Portworx
  - Search for your AWS EC2 instances in the AWS Management Console using your GitHub username. 
  - Create an AWS EBS volume of any size (e.g. 100 Gb) for the quantity of participating DC/OS Private Agents.
    - Identify your AWS EC2 instances using a unique Tag.
  - Attach the AWS EBS volumes to all participating DC/OS Private Agents.
  
  - Deploy Portworx
    - Set node count to the quantity of nodes in your DC/OS cluster.
    - Enable etcd.
    - Enable Lighthouse.

    - Deploy [Repoxy](https://docs.portworx.com/scheduler/mesosphere-dcos/lighthouse-marathon.html#accessing-lighthouse)
    - Access Portworx Lighthouse: 
      - http://PublicAgentIP:9999
      - portworx@yourcompany.com / admin


### Packages
  - Deploy Kubernetes on DC/OS, with HA enabled, and 3 worker nodes.
```
dcos package install kubernetes
```
  - Deploy Marathon-LB.
```
dcos package install marathon-lb
```
  - Deploy Portworx-hadoop on DC/OS
```
dcos package install portworx-hadoop
```


### Kubernetes
  - [Expose the Kubernetes API using Marathon-LB without TLS Certificate Verification](https://docs.mesosphere.com/services/kubernetes/1.2.0-1.10.5/exposing-the-kubernetes-api-marathonlb/)
```
dcos marathon app add kubectl-proxy.json
```
  - Find public agent IP
    - Use the Public Agent IP indicated on commpletion of applying the Terraform configuration, or:
    ```
    for id in $(dcos node --json | jq --raw-output '.[] | select(.attributes.public_ip == "true") | .id'); do dcos node ssh --option StrictHostKeyChecking=no --option LogLevel=quiet --master-proxy --mesos-id=$id "curl -s ifconfig.co" ; done 2>/dev/null
    ``` 
  -Configure kubectl to access the Kubernetes API without validating the presented TLS certificate:
```
dcos kubernetes kubeconfig --apiserver-url https://PubAgentIP:6443 --insecure-skip-tls-verify
```

### Deploy ksonnet
```
brew install ksonnet/tap/ks
```

### Kubeflow
#### Deploy Jupyterlab
```
dcos package install jupyterlab
```

  - Networking: use the public agent ELB address
  - Enable checkbox: 'Start Tensorboard'
- Access Jupyter: 
    - http://**publicAgentIP**:10104/jupyterlab-notebook/login
    - UI password: jupyter



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


### Deploy Kubeflow
* https://www.kubeflow.org/docs/about/user_guide/

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
* [Install Mesosphere DC/OS Enterprise on AWS](https://github.com/mesosphere/terraform-dcos-enterprise/blob/master/aws/README.md)
* [Kubeflow](https://www.kubeflow.org)
* [Portworx](https://www.portworx.com)
