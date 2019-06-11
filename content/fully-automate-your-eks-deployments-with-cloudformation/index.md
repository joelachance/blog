---
title: "Fully Automate your EKS Deployments with CloudFormation"
description: "With the release of EKS this last summer, our shop decided to give EKS a shot and create a fully automated deployment using CloudFormation…"
date: "2019-06-11T15:27:28.419Z"
categories: []
published: false
---

With the release of EKS this last summer, our shop decided to give EKS a shot and create a fully automated deployment using CloudFormation. Previously, like most (if not all) k8 shops, we were using `kops` . `kops` gives you something you forfeit with AWS — agnostic deployments on the cloud service of your choice. If this is a dealbreaker, CloudFormation isn’t your answer. We decided to drink the Kool-aid and join AWS. 

The benefit of EKS: EKS removes any management of your clusters. Your nodes will still live as EC2 instances, and with `heptio-authenticator`, you’ll still have `kubectl` to navigate your nodes & pods. So while not entirely hands-free (yet), reducing cluster management is still helpful.¹ EKS is still in its infant stages, and since we’ve automated our k8 deployments the EKS docs have changed a lot.

#### An EKS Review

EKS for us is simply one step in the right direction. We can’t spend days on Devops, so one less resource to manage is good. Even with EKS, we still need to manage our VPC’s, Subnets, Elastic IP’s… the list goes on. The spin-up time for clusters is still fairly slow compared to GCP (or so I hear). The saving grace here is truly CloudFormation. Everything is provisioned in our deployment yaml, so being able to ‘black box’ our resources have been helpful to speed up Devops tasks.

---

#### Why CloudFormation?

If you’re unaware of CloudFormation, this is a way to describe your infrastructure in yaml format so AWS can properly provision these resources. In other words, it’s a way to describe your infrastructure so AWS can build it for you. There are extensive docs to support the CloudFormation API found [here](https://docs.aws.amazon.com/cloudformation/index.html#lang/en_us). Hopefully, this starts to answer why we would want to use CloudFormation— although EKS is provisioning our clusters for us, we still need to provision a VPN, subnets, k8 nodes, the list goes on. If you need to re-build a cluster, manually entering all of this would take a while.² Think of it like killing about 20 birds with one stone.

#### Getting Started

Before we begin, it’s worth going through the EKS [Getting Started](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html) Doc. Here you’ll make sure your `cli` is setup properly and you have `heptio-authenticator` working properly. AWS also has two tutorials in the [Getting Started](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html) that will help understand the workings of an EKS deployment.

After you have a basic setup, we can start looking at CloudFormation. AWS gives us a starter CloudFormation template [here](https://github.com/awslabs/amazon-eks-ami/blob/master/amazon-eks-nodegroup.yaml). This is a good starting point, however, it only deploys our nodes. In the spirit of automation, we really want to deploy all resources in one shot, which we will get to shortly. There are two sections worth noting: `Parameters` & `Resources` .

Think of Parameters as variables for CloudFormation. This is your configuration for your deployment. You’ll notice in the starter template there is a lot of config. It doesn’t hurt to leave it in there, but I find it much easier to read to remove the unnecessary configuration.

The `Resources` are the meat and potatoes of our template. As the name suggests, this is where our deployment resources are provisioned when your CloudFormation template is run. For example:

```
NodeInstanceProfile: 
  Type: AWS::IAM::InstanceProfile 
  Properties: 
    Path: “/” 
    Roles: 
      - !Ref NodeInstanceRole
```

Note the `!Ref NodeInstanceRole` here. This is referencing our `NodeInstanceRole` Parameter we have set in the `Parameters` section of our template. For illustration’s sake, here’s an EKS resource:

```
EKS:
  Type: AWS::EKS::Cluster
  Properties:
    Name: !Sub "${AWS::StackName}"
    Version: "1.10"
    RoleArn: <your role ARN goes here>
    ResourcesVpcConfig:
      SubnetIds: [ !Ref PublicSubnet1, !Ref PublicSubnet2, !Ref PrivateSubnet1, !Ref PrivateSubnet2 ]
```

Keep in mind your resources may be dependant on other resources. These dependencies need to come first in your `Resources` block.

After running this template, we should now have a provisioned EKS Cluster and the associated Nodes. These are not connected to each other yet! You’ll need to run a yaml in `kubectl` to point your new Nodes (EC2 Instances) to your Cluster. See AWS’s walkthrough [here](https://docs.aws.amazon.com/eks/latest/userguide/launch-workers.html). 

#### Final Thoughts

This is a very brief overview of CloudFormation and how you can use it with EKS. In a future post, I’ll go further into the weeds of how the CloudFormation template we’re using connects all of our resources for a very quick & hands free deployment.

---

#### Foot Notes

1.  I’ve read many articles that GCP has a much better implementation with their Kubernetes Engine, which I don’t doubt is true. I haven’t had the time to try it out, so forgive my GCP ignorance.
2.  A shoutout to AWS Support. Those guys have gone above and beyond to help out on troubleshooting and have an excellent knowledge base. A great way to submit feature requests too.t
