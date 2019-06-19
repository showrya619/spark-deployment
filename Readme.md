# **Kubernetes Clusters on AWS Using Kops**

## Install kops

### Linux:
```
wget https://github.com/kubernetes/kops/releases/download/1.6.1/kops-linux-amd64
sudo chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 kops
sudo mv kops /usr/local/bin/
```
### Windows:
Download the offical release from official Windows releases : [kops-windows](https://github.com/kubernetes/kops/releases)

Rename ```kops-windows-amd64``` to ```kops.exe``` 

Move it to a directory of your preference and add it to PATH. eg. ```C:\Users\Showrya\kops\kops.exe```

## Install kubectl

### Linux:
```
curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```
### Windows:
[kubectl-windows](https://docs.aws.amazon.com/cli/latest/userguide/install-windows.html)

## Install aws-cli

### Linux:
```pip install awscli```

### Windows:
[aws-cli-windows](https://docs.aws.amazon.com/cli/latest/userguide/install-windows.html)


## IAM user permission
The IAM user to create the Kubernetes cluster must have the following permissions:
```
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
```
Alternatively, a new IAM user may be created and the policies attached as explained [here](github.com/kubernetes/kops/blob/master/docs/aws.md#setup-iam-user)

## Create an Amazon S3 bucket for the Kubernetes state store

Bucket Name : kubernetes-aws-iot

#### Create an S3 bucket
```aws s3api create-bucket --bucket kubernetes-aws-iot```


#### Bucket Versioning

For reverting or recovering a previous version of the cluster. 

```aws s3api put-bucket-versioning --bucket kubernetes-aws-iot --versioning-configuration Status=Enabled```


#### Point environment variable to the S3 bucket

Linux:
```export KOPS_STATE_STORE=s3://kubernetes-aws-iot```

Windows:
```set KOPS_STATE_STORE=s3://kubernetes-aws-iot```

## DNS Configuration:

Note: If you are using Kops 1.6.2 or later, then DNS configuration is optional. Instead, a gossip-based cluster can be easily created. The only requirement to trigger this is to have the cluster name end with .k8s.local. If a gossip-based cluster is created then you can skip this section.


## Create Cluster:
```
kops create cluster \
--name cluster.kubernetes-aws.iot.k8s.local. \ 
--zones ap-south-1a \   
--state s3://kubernetes-aws-iot \
--yes
```
```--name``` : name of the cluster.

```--zones```: Defines the zones in which the cluster is going to be created.

```--state```: Points to the S3 bucket that is the state store.

This starts a single master and two worker node Kubernetes cluster. The master is in an Auto Scaling group and the worker nodes are in a separate group. By default, the master node is m3.medium and the worker node is t2.medium. 
Master and worker nodes are assigned separate IAM roles as well.

![create cluster](https://user-images.githubusercontent.com/40289521/59744248-c4213500-928f-11e9-9f84-db23414f4db7.PNG)

The cluster can be verified by

```kops validate cluster --state=s3://kubernetes-aws-iot```


More details about the cluster can be seen using the commands

```kubectl cluster-info```

![cluster-info](https://user-images.githubusercontent.com/40289521/59744517-5f1a0f00-9290-11e9-97fc-1e04cb142f87.PNG)


```kubectl get svc```

![get-svc](https://user-images.githubusercontent.com/40289521/59744630-a0aaba00-9290-11e9-9a4a-e6eaed71b9ed.PNG)


```kubectl get nodes```

![get-nodes](https://user-images.githubusercontent.com/40289521/59744635-a30d1400-9290-11e9-9456-89e20e60943d.PNG)


## Delete Cluster

```kops delete cluster cluster.kubernetes-aws.iot.k8s.local --state=s3://kubernetes-aws-iot --yes```

![kops-delete-1](https://user-images.githubusercontent.com/40289521/59744807-fd0dd980-9290-11e9-9011-9311f94f9cfe.PNG)

![kops-delete-2](https://user-images.githubusercontent.com/40289521/59744809-fe3f0680-9290-11e9-8f6a-bb7630714f15.PNG)

## Conclusion

This post explained how to manage a Kubernetes cluster on AWS using kops.
