# AWS Practices 

## <a name="1"></a> VPC/Subnet/Internet Gateway (1 instance)
[1_1_vpc_1_public_subnet_1_instance_1_internet_gateway.json:](../master/1_1_vpc_1_public_subnet_1_instance_1_internet_gateway.json)
   Simple stack that contains `one VPC`, `one public subnet`, `Internet gateway` and `one instance` connected to the internet through the gateway.

<p align="center">
    <img src="https://github.com/mohammedalics/aws_practices/blob/master/1_1_vpc_1_public_subnet_1_instance_1_internet_gateway.png" alt="1_1_vpc_1_public_subnet_1_instance_1_internet_gateway.json"/>
</p>

## <a name="2"></a> VPC/Subnet/Internet Gateway (2 instances) 
[2_1_vpc_1_public_subnet_2_instances_1_internet_gateway.json:](../master/2_1_vpc_1_public_subnet_2_instances_1_internet_gateway.json)
   Simple stack that contains `one VPC`, `one public subnet`, `Internet gateway` and `two instances` connected to the internet through the gateway.

The above sample has been updated to: 
- Include one more instance in the same public subnet but without assigning a publicIP. 
- Add IpProtocol `icmp` from port `8` to all `-1` in `SecurityGroup` to enable `ping` request between the machines. 

<p align="center">
    <img src="https://github.com/mohammedalics/aws_practices/blob/master/2_1_vpc_1_public_subnet_2_instances_1_internet_gateway.png" alt="2_1_vpc_1_public_subnet_2_instances_1_internet_gateway.json"/>
</p>

## <a name="3"></a> VPC/2 Subnets/Internet Gateway (2 instances) 
[3_1_vpc_1_public_subnet_1_private_subnet_2_instances_1_internet_gateway.json:](../master/3_1_vpc_1_public_subnet_1_private_subnet_2_instances_1_internet_gateway.json)
   Simple stack that contains `one VPC`, `one public subnet`, `one private subnet`, `Internet gateway` and `two instances`. The `public subnet` connected to the internet through the `CustomRouteTable` route the traffic to the `Internet gateway`. The `private subnet` is not connected to the internet and use the `MainRouteTable`. 

The above sample has been updated to: 
- Move the private instance to a private subnet. 

<p align="center">
    <img src="https://github.com/mohammedalics/aws_practices/blob/master/3_1_vpc_1_public_subnet_1_private_subnet_2_instances_1_internet_gateway.png" alt="3_1_vpc_1_public_subnet_1_private_subnet_2_instances_1_internet_gateway.png"/>
</p>

## <a name="4"></a> VPC/Subnet/Internet Gateway with recovery alarm (1 instance)
[4_1_vpc_1_public_subnet_1_instance_1_internet_gateway_recovery_alarm.json:](../master/1_1_vpc_1_public_subnet_1_instance_1_internet_gateway_recovery_alarm.json)
   Same as [VPC/Subnet/Internet Gateway (1 instance)](#1) but a recovery alarm `CloudWatch` was added. 
_`UserData` added to start jenkins server on 8080 for later use_

<p align="center">
    <img src="https://github.com/mohammedalics/aws_practices/blob/master/4_1_vpc_1_public_subnet_1_instance_1_internet_gateway_recovery_alarm.png" alt="4_1_vpc_1_public_subnet_1_instance_1_internet_gateway_recovery_alarm.json"/>
</p>

## <a name="5"></a> VPC/2 Subnets/Internet Gateway/AutoScaling (1 instance max/min) 
[5_1_vpc_2_public_subnet_1_internet_gateway_autoscaling.json:](../master/5_1_vpc_2_public_subnet_1_internet_gateway_autoscaling.json)
   Simple stack that contains `one VPC`, `two public subnets`, `Internet gateway`, `launch configruation` and `autoscaling for one instance min/max` `two instances`. The `subnets` connected to the internet through the `CustomRouteTable` route the traffic to the `Internet gateway`.

<p align="center">
    <img src="https://github.com/mohammedalics/aws_practices/blob/master/5_1_vpc_2_public_subnet_1_internet_gateway_autoscaling.png" alt="5_1_vpc_2_public_subnet_1_internet_gateway_autoscaling.png"/>
</p>

## <a name="6"></a> VPC/2 Subnets/Internet Gateway/AutoScaling/EBS Recovery  (1 instance max/min) 
[6_1_vpc_2_public_subnet_1_internet_gateway_autoscaling_ebs_recovery.json:](../master/6_1_vpc_2_public_subnet_1_internet_gateway_autoscaling_ebs_recovery.json)
   Same as [VPC/2 Subnets/Internet Gateway/AutoScaling (1 instance max/min)](#5) but the `imageId` is parameterized as an `optional` parameter. so we can:
   - Create snapshots of the EBS volume, and use snapshot if a virtual server needs to recover in another availability zone. EBS snapshots are stored on S3 to be available in multiple availability zones.
   ```
   aws ec2 create-image --instance-id=i-0ad5005528a7ed71f --name jenkins-instance
   ```
wait until recieving `available` for the image status.
```
aws ec2 describe-images --image-id $newImageId --query "Images[].State"
```
- Update the stack with the new imageId. 
```
aws cloudformation update-stack --stack-name $stackName --template-url $tempateUrl --parameters ParameterKey=JenkinsAdminPassword,UsePreviousValue=true ParameterKey=AMISnapshot,ParameterValue=$newImageId
```

### To Test it: 
- Open the jenkins server and create a job (just to make sure that the EBS Recovery is working)
- Write down the current instance avaialability zone. 
- Terminiate the current running instance
```
aws ec2 terminate-instances --instance-ids $instanceId
```
- Wait until the instance terminiated and newely instance created. 
- Get the new publicIP
- Log in to the jenkins server http://$publicIP:8080/ and verify if the job is there.

### To clean up:

Run below:
```
aws cloudformation delete-stack --stack-name $stackName
aws cloudformation describe-stacks --stack-name $stackName #wait until return error or stack deleted.
aws ec2 deregister-image --image-id $newImageId
aws ec2 delete-snapshot --snapshot-id $snapshotId
```
<p align="center">
    <img src="https://github.com/mohammedalics/aws_practices/blob/master/6_1_vpc_2_public_subnet_1_internet_gateway_autoscaling_ebs_recovery.png" alt="6_1_vpc_2_public_subnet_1_internet_gateway_autoscaling_ebs_recovery.png"/>
</p>

## <a name="7"></a> VPC/2 Subnets/Internet Gateway/AutoScaling/EBS Recovery/ElasticIP  (1 instance max/min) 
[7_1_vpc_2_public_subnet_1_internet_gateway_autoscaling_ebs_recovery_elastic_ip.json:](../master/7_1_vpc_2_public_subnet_1_internet_gateway_autoscaling_ebs_recovery_elastic_ip.json)
   Same as [VPC/2 Subnets/Internet Gateway/AutoScaling/EBS Recovery  (1 instance max/min)](#6) but overcoming the problem of assigning another publicIP/PrivateIP to the new instance in another availability zone since we can't keep the same publicIP between different zones. 
   By default, you also can’t use an Elastic IP as a public IP address for a virtual server launched by auto-scaling.
- Allocating an Elastic IP
- Adding the association of an Elastic IP to the script in the user data
- Creating an IAM role and policy to allow the EC2 instance to associate an Elastic IP

### To Test it: 
- Open the jenkins server with the elasticIP assigned. 
- Terminiate the current running instance
- Wait until the instance terminiated and newely instance created. 
- Log in to the jenkins server http://$elasticIP:8080/ and verify if jenkins is running.

<p align="center">
    <img src="https://github.com/mohammedalics/aws_practices/blob/master/7_1_vpc_2_public_subnet_1_internet_gateway_autoscaling_ebs_recovery_elastic_ip.png" alt="7_1_vpc_2_public_subnet_1_internet_gateway_autoscaling_ebs_recovery_elastic_ip.png"/>
</p>


## <a name="8"></a> VPC/2 Subnets/Internet Gateway/AutoScaling/EBS Recovery/LoadBalaner (2 instance max/min) 
[8_1_vpc_2_public_subnet_1_internet_gateway_autoscaling_ebs_recovery_loadbalancer.json:](../master/8_1_vpc_2_public_subnet_1_internet_gateway_autoscaling_ebs_recovery_loadbalancer.json)
   Same as [VPC/2 Subnets/Internet Gateway/AutoScaling/EBS Recovery  (1 instance max/min)](#6) but overcoming the problem of assigning another publicIP/PrivateIP to the new instance in another availability zone since we can't keep the same publicIP between different zones. 
For that reason, a loadbalancer has been created to distribute the traffic to the EC2 instances. Once a new instance created, It registers itself to the loadbalaner. 


<p align="center">
    <img src="https://github.com/mohammedalics/aws_practices/blob/master/8_1_vpc_2_public_subnet_1_internet_gateway_autoscaling_ebs_recovery_loadbalancer.png" alt="8_1_vpc_2_public_subnet_1_internet_gateway_autoscaling_ebs_recovery_loadbalancer.png"/>
</p>

___
- [draw.io source folder](https://drive.google.com/drive/folders/1u0WFI9xcUswzycxNzZ1JL4MwiRtE45CF?usp=sharing). 
