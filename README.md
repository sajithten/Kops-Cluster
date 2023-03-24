# Kubernetes on AWS using Kops

# 1. Launch Linux EC2 instance in AWS (Kubernetes Client) or any linux machine

# 2. Install awscli version2
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install

    Confirm the installation with the following command.

    aws --version 

# 3. Create and attach IAM role to EC2 Instance.
	Kops need permissions to access
	  S3
	  EC2
	  VPC
	  Route53
	  Autoscaling
	  etc..
# 4. Install Kops on EC2
	curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
	chmod +x kops-linux-amd64
	sudo mv kops-linux-amd64 /usr/bin/kops
	kops version

# 5. Install kubectl 
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
	chmod +x ./kubectl
	mkdir -p $HOME/bin
	cp ./kubectl $HOME/bin/kubectl
	export PATH=$HOME/bin:$PATH
	echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
	source $HOME/.bashrc
	kubectl version --short --client

# 6. Install kubectl 1.14.6 version
	curl -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/kubectl
	chmod +x ./kubectl
	mkdir -p $HOME/bin
	cp ./kubectl $HOME/bin/kubectl
	export PATH=$HOME/bin:$PATH
	echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
	source $HOME/.bashrc
	kubectl version --short --client

# 7. Create S3 bucket in AWS
    S3 bucket is used by kubernetes to persist cluster state, lets create s3 bucket using aws cli Note: Make sure you choose bucket name that is uniqe accross all aws accounts

	aws s3 mb s3://unique-bucket-name --region ap-south-1

# 8. Create hosted zone in AWS Route53
	Head over to aws Route53 and create hostedzone
	Choose name for example (domainname.ml)
	Hit create

# Note:

    Add NS records for the domain where you register the domain 

# 9. Create ssh key pair
This keypair is used for ssh into kubernetes cluster

	ssh-keygen
# 10. Create a Kubernetes cluster definition
    
	kops create cluster --yes --state=s3://<bucketname> --zones=<AZname1, AZname2> --node-count=<no.of node, minimum 2> --name=<clustername>.k8s.local 

    eg:
	kops create cluster --yes --state=s3://9am-k8s --zones=eu-west-1c,eu-west-1b --node-count=2 --name=9am.k8s.local 

    eg:
	kops create cluster --yes --zones=us-east-1a,us-east-1b,us-east-1c --name=domainname.ml --state s3://domainname.ml.k8s

	Above command may take some time to create the required infrastructure resources on AWS. Execute the validate command to check its status and wait until the cluster becomes ready

	kops validate cluster --state s3://domainname.ml.k8s --wait 10m

    For the above above command, you might see validation failed error initially when you create cluster and it is expected behaviour, you have to wait for some more time and check again.

# 11. Check nodes of the cluster
	kubectl get nodes

# 12. Clean-UP
	kops delete cluster --name=domainname.ml --state s3://domainname.ml.k8s --yes
