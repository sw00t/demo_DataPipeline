# demo_DataPipeline
**This repository is for internal Mesosphere staff only**

The goal of this demo is to show audiences in which DC/OS facilitates ease of lifecycle mgmt for data services, leverage kubeflow independently or on Kubernetes on DC/OS, and other components to build out your data pipeline, whether it's open source, public cloud, or hybrid of both.

Featured components include DC/OS, Kubernetes, TensorFlow, Jupyter, Spark, Portworx, HDFS. Future updates to include Kubeflow, Kafka, more.

You will need the appropriate permissions in AWS to provision a Mesosphere DC/OS Enterprise cluster using Terraform. 
The DC/OS cluster should have at minimum 3 Masters, 1 Public Agent, and 12 Private Agents.

This repository is a **Work In Progress**, until indicated otherwise in this README. Enjoy!


## Demo: Data Science Pipeline on DC/OS
![.](https://i.imgur.com/gzgMOAg.png)


### DC/OS
  - Authenticate CLI using MAWS
```
maws login [AWS user account]
```
  - Update **desired_cluster_profile.tfvars** with correct values for **aws_profile**, and **dcos_license_key_contents**
  - Use Terraform and the above tfvars file to deploy a DC/OS cluster of 3 masters, 12 private agents, and 1 public agent.
```
mkdir dcos-installer
cd dcos-installer
terraform apply -var-file desired_cluster_profile.tfvars
```
  - On completion, take note of all IPs in the summary.

### Portworx preparation
  - Identify your EC2 instances in the AWS Management Console. 
  - Create EBS volumes in the same AWS Region as your EC2 instances (e.g. 100 Gb, 12 EBS volumes).
    - Identify your AWS EC2 instances using a unique Tag.
  - Attach the 12 AWS EBS volumes to all 12 private agents.

### DC/OS CLI
  - Configure your CLI to access the DC/OS cluster, and install Marathon-LB.
```
dcos cluster setup <MasterIP>
dcos package install marathon-lb --yes
```

### Kubernetes 1.1.1-1.10.4
  - Install kubectl-proxy.json
```
dcos marathon app add kubectl-proxy.json
```
  - Install Kubernetes on DC/OS with HA enabled and 3 worker nodes, via the DC/OS GUI
  - Alternately, use the k8s-options.json to deploy by CLI
```
dcos package install kubernetes --package-version=1.1.1-1.10.4 --options=k8s-options.json
```
  - Watch deployment progress
```
brew install watch
watch -n1 dcos kubernetes plan status deploy
```
  - Configure kubectl with the Public Agent IP without TLS verification
```
dcos kubernetes kubeconfig --apiserver-url https://**PubAgentIP**:6443 --insecure-skip-tls-verify
```

### Portworx 1.3.1-4.2.1
  - Install Portworx
    - Set node count to the quantity of nodes in your DC/OS cluster.
    - Enable etcd.
    - Enable Lighthouse.
  -  Alternately, use the px-options.json to deploy by CLI
```
dcos package install portworx --package-version=1.3.1-4.2.1 --options=px-options.json
```
  - Install the Portworx CLI
```
dcos package install portworx --package-version=1.3.1-4.2.1 --cli --yes
```
  - Watch deployment progress
```
watch -n1 dcos portworx plan status deploy
```
  - Deploy Repoxy
```
dcos marathon app add repoxy.json
```

### Portworx Lighthouse
  - Browse to http://**Public Agent IP**:9999
    - admin / Password1
  - If required, add the Portworx cluster by providing the IP address of any one of the nodes in the cluster.


### Kubeflow (optional)
  - Install Kubeflow components on Kubernetes either independently, or on Kubernetes on DC/OS.
### Install ksonnet (optional)
```
brew install ksonnet/tap/ks
```

### HDFS / Hadoop
  - Deploy Portworx-hadoop on DC/OS for use with JupyterLab
```
dcos package install portworx-hadoop
```

### JupyterLab 1.2.0-0.33.7
  - Install JupyterLab via the DC/OS GUI
    - Networking: External Access: external public agent hostname: **Public Agent ELB Address**
    - Environment: Environment: jupyter conf urls: http://api.portworx-hadoop.marathon.l4lb.thisdcos.directory/v1/endpoints
    - Enable checkbox 'Start Tensorboard'
    - Click on Review & Run

### Access Jupyter
```
http://**Public Agent IP**:10104/jupyterlab-notebook/login
```
  - Password: jupyter

### SparkPi Job
  - Once logged in to Jupyter, launch Terminal.
  - In another browser window, open http://**Master IP**/mesos/ and show Spark task that are about to be run.
  - Run the following Spark job:
  ```
  eval spark-submit ${SPARK_OPTS} --verbose --class org.apache.spark.examples.SparkPi /opt/spark/examples/jars/spark-examples_2.11-2.2.1.jar 100
  ```
  - Highlight where the value of Pi is calculated, and the Spark teardown log messages.

### SparkPi with Apache Toree
  - Launch the **Apache Toree Scala** kernel in a new notebook
  - Run the SparkPi example to compute Pi:
  ```
  val NUM_SAMPLES = 10000000
  val count2 = spark.sparkContext.parallelize(1 to NUM_SAMPLES).map{i =>
  val x = Math.random()
  val y = Math.random()
  if (x*x + y*y < 1) 1 else 0
  }.reduce(_ + _)

  println("Pi is roughly " + 4.0 * count2 / NUM_SAMPLES)
  ```
  - click the Play button, observe the results.
  

### TensorBoard
  - Access TensorBoard to show visualization via the UI: https://**Public Agent ELB Address**/jupyterlab-notebook/tensorboard/


### MNIST TensorFlowOnSpark
  - In the JupyterLab Terminal, clone the following repository:
  ```
  git clone https://github.com/yahoo/TensorFlowOnSpark
  ```
  - Retrieve and extract the raw MNIST dataset:
  ```
  cd $MESOS_SANDBOX
  curl -fsSL -O https://s3.amazonaws.com/vishnu-mohan/tensorflow/mnist/mnist.zip
  unzip mnist.zip
  ```
  - Check HDFS to show the directory is empty:
  ```
  hdfs dfs -ls  mnist/
  ```
  - Prepare the MNIST dataset:
  ```
  eval spark-submit ${SPARK_OPTS} --verbose $(pwd)/TensorFlowOnSpark/examples/mnist/mnist_data_setup.py --output mnist/csv --format csv
  ```
  - Check the results of trained model on HDFS:
  ```
  hdfs dfs -ls -R  mnist/
  ```
  - Train the MNIST model with CPUs from the Terminal:
  ```
  eval spark-submit ${SPARK_OPTS} --verbose --conf spark.mesos.executor.docker.image=dcoslabs/dcos-jupyterlab:1.2.0-0.33.7 --py-files $(pwd)/TensorFlowOnSpark/examples/mnist/spark/mnist_dist.py $(pwd)/TensorFlowOnSpark/examples/mnist/spark/mnist_spark.py --cluster_size 5 --images mnist/csv/train/images --labels mnist/csv/train/labels --format csv --mode train --model mnist/mnist_csv_model
  ```
  - Check for the trained model on HDFS:
  ```
  hdfs dfs -ls -R mnist/mnist_csv_model
  ```

### Clean Up
  - Destroy the DC/OS cluster using Terraform:
```
terraform destroy -var-file desired_cluster_profile.tfvars
```
  - Remove all AWS EBS volumes via the AWS Management Console.






## Resources
* [Mesosphere DC/OS](https://dcos.io)
* [Mesosphere DC/OS Enterprise](https://mesosphere.com/product)
* [Joerg Schad](https://github.com/joerg84)
* [Fast Data: Data Analytics with JupyterLab, Spark and TensorFlow](https://github.com/dcos/demos/tree/822a64143bcd7df2f6027c053f664eeb038e9693/jupyterlab/1.11)
* [How to use JupyterLab on DC/OS](https://github.com/dcos/examples/tree/master/jupyterlab/1.11)
* [Install Mesosphere DC/OS Enterprise on AWS](https://github.com/mesosphere/terraform-dcos-enterprise/blob/master/aws/README.md)
* [Kubeflow](https://www.kubeflow.org/docs/about/user_guide/)
* [Portworx](https://www.portworx.com)
