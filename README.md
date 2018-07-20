# DataPipeline-Kubeflow-DCOS
***This repository is intended for internal Mesosphere staff only***

Mesosphere repository for Data Pipeline workshops featuring DC/OS, Kubernetes, Kubeflow, TensorFlow, Jupyter, Spark, Portworx, HDFS, Kafka. 

You will need to provision a Mesosphere DC/OS Enterprise Edition cluster in `permissive` mode.

A DC/OS cluster with at least 3 Masters, 1 Public Agent, and 8 Private Agents is required for an aggregate of 36 CPU, 132 GiB Memory, and 1 TiB storage.

Deploy Kubernetes on DC/OS
```
dcos package install kubernetes
```

Deploy BeakerX on DC/OS
```
dcos package install beakerx
```

Deploy Spark on DC/OS
```
dcos package install spark
```

Deploy ksonnet
```
brew install ksonnet/tap/ks
```

[Deploy kubeflow](https://www.kubeflow.org/docs/about/user_guide/)


Deploy Portworx-Hadoop on DC/OS
```
dcos package install portworx-hadoop
```

Deploy Portworx-Kafka on DC/OS
```
dcos package install portworx-kafka
```




Resources, made possible by and/or for:
* [Mesosphere DC/OS](https://dcos.io)
* [Mesosphere DC/OS Enterprise](https://mesosphere.com/product)
* [KubeCon EU 2018, Bringing Your Data Pipeline into The Machine Learning Era](https://www.youtube.com/watch?v=f_-3rQoudnc)
* [Joerg Schad](https://github.com/joerg84)
* [Kubeflow](https://www.kubeflow.org)
* [Portworx](https://www.portworx.com)
