Graviton2 Workshop - Optimizing cost with AWS Graviton2 and containers (L200)

Lab guide: https://graviton2-workshop.workshop.aws/en/amazoncontainers.html

* Create container build pipeline
	- provisioning is automated by AWS Cloud Development Kit (CDK)

* Multi-architecture application build
	- create docker images for x86-64 and arm64
	- create docker image of multi-architecture manifest 
	- with CloudFormation
	- commit and push to CodeCommit EC2 repository
	- can monitor build process with CodePipeline
	
* Amazon EKS: Deploy multi-architecture EKS cluster
	- mixed nodegroups: 2 nodes x86-64, 2 nodes arm64

* Amazon EKS: Running ASP.NET Core application on Graviton2 
	- deploy .NET Core5 using Elastic Kubernetes Service (EKS) on Graviton2 instance types
	- .NET 5 is arm64 architecture ready
	- deploy .NET Core build pipeline using CDK
	- clone .NET Core sample git repo
	- create empty AWS Code Commit git repo
	- copy build specification yaml file, Dockerfile and .NET Core 5 application source code to new empty repo
	- add, commit, push. monitor in AWS CodePipeline. Set environmental variables for deployment.
	- deploy .NET Core application to EKS cluster
	- "kubectl get svc" to check kubernetes cluster status

* Amazon ECS: Deploy Gaviton2 ECS cluster
