# Knowledge prepartion
1. [What is ENI](https://www.youtube.com/watch?v=uICq0rkOgyw)
2. [VPC communication](https://www.youtube.com/watch?v=0YG3vo78gSM)
3. [Service vs Ingress](https://www.youtube.com/watch?v=NPFbYpb0I7w)
4. [EKS structure](https://aws.amazon.com/tw/blogs/containers/de-mystifying-cluster-networking-for-amazon-eks-worker-nodes/)


# Before EKS creation
Before the creation, you need to decide with account to create cluster since K8S only grant system:masters for the creator account. Others account need to use `RoleBinding` / `ClusterRoleBinding` for account creation. https://docs.aws.amazon.com/zh_tw/eks/latest/userguide/add-user-role.html


# AWS roles preparation (if you create without eksctl)
1. Create an AWS role with `AmazonEKSClusterPolicy` (ROLE_A). This is used for EKS cluster creation.
2. Create an AWS role with `AmazonEKSWorkerNodePolicy`, `AmazonEKS_CNI_Policy`, `AmazonEC2ContainerRegistryReadOnly` (ROLE_B). This is used for work node group creation inside EKS cluster.


# EKS Network Mode

## Public 
1. By manually

    Create VPC with public subnet and put all node group inside public subnet.

2. By eksctl

    Execute `eksctl/public-cluster.yml` (You need to create VPC with public subnet first). OR modify script to create VPC automatically.

## Private
1. By manually

    You need to setup VPC Endpoint for 3 private subnets
    https://docs.aws.amazon.com/zh_tw/eks/latest/userguide/private-clusters.html

```
com.amazonaws.ap-southeast-1.s3
com.amazonaws.ap-southeast-1.ec2
com.amazonaws.ap-southeast-1.ecr.dkr
com.amazonaws.ap-southeast-1.ecr.api
com.amazonaws.ap-southeast-1.sts
```

2. By eksctl

    Execute `eksctl/private-cluster.yml` (You need to create VPC with private subnet first). OR modify script to create VPC automatically.


## Public + Private (Fully Private)
1. By manually

    Create public and private subnet with NAT gateway. Put worker node inside private subnet.

2. By eksctl

    Execute `eksctl/public+private-cluster.yml` (You need to create VPC with NAT first). OR modify script to create VPC automatically.



# After EKS creation
1. Create `kubeconfig` by `aws eks update-kubeconfig --region ap-southeast-1 --name eks-demo`
2. Test config by `kubectl get svc` to get cluster information


# Additional Information
1. AWS credential and config stored in `~/.aws`
2. `kubeconfig` stored in `~/.kube/config`
3. K8S service account: https://www.cnblogs.com/panwenbin-logs/p/10029834.html


# Before integrate with AWS
1. Install AWS load balancer controller for K8S
https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html
2. Refer to this [link](https://www.gushiciku.cn/pl/2dUW/zh-tw) to see more structure

# Step to integrate with API gateway + VPC Link + Private NLB + Private Service
<img src="https://mdimg.wxwenku.com/getimg/6b990ce30fa9193e296dd37902816f4bbecae856f749da2cddc527de0d8a9287408aaf084e019c3ed3efef5970187cc5.jpg" />

1. Deploy deployment and service to K8S. The `k8s/nlb-service.yml` contain `service.beta.kubernetes.io` will auto deploy NLB in AWS
2. Create VPN link for NLB
3. Create API gateway connect to this VPN link


# Step to integrate with Public ALB + Private Ingress + Private Service
<img src="https://mdimg.wxwenku.com/getimg/ccdf080c7af7e8a10e9b88444af9839322fc2e5e02a4713a27a81b0f199bf55a9e2369cbc0602fc4a461538c538c77fc.jpg" />

1. Deploy deployment, `k8s/simple-service.yml`, `k8s/alb-ingress.yml`. It will automatically create ALB for you.