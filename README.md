# eks-karpenter-mng-custom-cni
Example for having Karpenter with custom CNI

URL : https://docs.aws.amazon.com/eks/latest/userguide/cni-custom-network-tutorial.html
Original template : https://github.com/aws-ia/terraform-aws-eks-blueprints/tree/main/patterns/karpenter-mng

### How to use this example
terraform init
kubectl run -it busybox --image=busybox --restart=Never -- /bin/sh #for testing deployment pods


### Normal step using cli :
Create security group : sg-02091518c4aaa44b1


Create subnet sg-02091518c4aaa44b1 ex-karpenter-mng-pod-ap-southeast-1a for 3 subnet


kubectl set env daemonset aws-node -n kube-system AWS_VPC_K8S_CNI_EXTERNALSNAT=false
kubectl set env daemonset aws-node -n kube-system AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG=true


---cluster_security_group_id=sg-02091518c4aaa44b1
new_subnet_id_1=subnet-01af55db9f0be951d
new_subnet_id_2=subnet-0c6463cfbca863624
new_subnet_id_3=subnet-0547cc4af454e4f3c

cluster_security_group_id=$(aws eks describe-cluster --name ex-karpenter-mng --query cluster.resourcesVpcConfig.clusterSecurityGroupId --output text)


cat >ap-southeast-1a.yaml <<EOF
apiVersion: crd.k8s.amazonaws.com/v1alpha1
kind: ENIConfig
metadata:
  name: ap-southeast-1a
spec:
  securityGroups:
    - $cluster_security_group_id
  subnet: $new_subnet_id_1
EOF

cat >ap-southeast-1b.yaml <<EOF
apiVersion: crd.k8s.amazonaws.com/v1alpha1
kind: ENIConfig
metadata:
  name: ap-southeast-1b
spec:
  securityGroups:
    - $cluster_security_group_id
  subnet: $new_subnet_id_2
EOF

cat >ap-southeast-1c.yaml <<EOF
apiVersion: crd.k8s.amazonaws.com/v1alpha1
kind: ENIConfig
metadata:
  name: ap-southeast-1c
spec:
  securityGroups:
    - $cluster_security_group_id
  subnet: $new_subnet_id_3
EOF
kubectl set env daemonset aws-node -n kube-system ENI_CONFIG_LABEL_DEF=topology.kubernetes.io/zone

kubectl describe daemonset aws-node -n kube-system | grep ENI_CONFIG_ANNOTATION_DEF


### Deploy pods for testing

kubectl run -it busybox --image=busybox --restart=Never -- /bin/sh

### Remove config from kubectl

#### Delete the context
kubectl config delete-context arn:aws:eks:ap-southeast-1:526169475020:cluster/ex-eks-mng-custom-cni
#### Delete the cluster
kubectl config delete-cluster arn:aws:eks:ap-southeast-1:526169475020:cluster/ex-eks-mng-custom-cni
#### Delete the user
kubectl config unset users.arn:aws:eks:ap-southeast-1:526169475020:cluster/ex-eks-mng-custom-cni
