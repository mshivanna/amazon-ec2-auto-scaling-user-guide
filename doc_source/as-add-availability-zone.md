# Expanding Your Scaled and Load\-Balanced Application to an Additional Availability Zone<a name="as-add-availability-zone"></a>

You can take advantage of the safety and reliability of geographic redundancy by spanning your Auto Scaling group across multiple Availability Zones within a region and then attaching a load balancer to distribute incoming traffic across those Availability Zones\. Incoming traffic is distributed equally across all Availability Zones enabled for your load balancer\.

**Note**  
An Auto Scaling group can contain EC2 instances from multiple Availability Zones within the same region\. However, an Auto Scaling group can't contain EC2 instances from multiple regions\.

When one Availability Zone becomes unhealthy or unavailable, Amazon EC2 Auto Scaling launches new instances in an unaffected Availability Zone\. When the unhealthy Availability Zone returns to a healthy state, Amazon EC2 Auto Scaling automatically redistributes the application instances evenly across all of the Availability Zones for your Auto Scaling group\. Amazon EC2 Auto Scaling does this by attempting to launch new instances in the Availability Zone with the fewest instances\. If the attempt fails, however, Amazon EC2 Auto Scaling attempts to launch in other Availability Zones until it succeeds\.

You can expand the availability of your scaled and load\-balanced application by adding an Availability Zone to your Auto Scaling group and then enabling that Availability Zone for your load balancer\. After you've enabled the new Availability Zone, the load balancer begins to route traffic equally among all the enabled Availability Zones\. 

**Topics**
+ [Add an Availability Zone Using the Console](#as-add-az-console)
+ [Add an Availability Zone Using the AWS CLI](#as-add-az-aws-cli)

## Add an Availability Zone Using the Console<a name="as-add-az-console"></a>

Use the following procedure to expand your Auto Scaling group to an additional subnet \(EC2\-VPC\) or Availability Zone \(EC2\-Classic\)\.

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **Auto Scaling**, choose **Auto Scaling Groups**\.

1. Select your group\.

1. On the **Details** tab, choose **Edit**\.

1. Do one of the following:
   + \[EC2\-VPC\] In **Subnet\(s\)**, select the subnet corresponding to the Availability Zone\.
   + \[EC2\-Classic\] In **Availability Zones\(s\)**, select the Availability Zone\.

1. Choose **Save**\.

1. On the navigation pane, choose **Load Balancers**\.

1. Select your load balancer\.

1. Do one of the following:
   + \[Classic Load Balancer in EC2\-Classic\] On the **Instances** tab, choose **Edit Availability Zones**\. On the **Add and Remove Availability Zones** page, select the Availability Zone to add\.
   + \[Classic Load Balancer in a VPC\] On the **Instances** tab, choose **Edit Availability Zones**\. On the **Add and Remove Subnets** page, for **Available subnets**, choose the add icon \(\+\) for the subnet to add\. The subnet is moved under **Selected subnets**\.
   + \[Application Load Balancer\] On the **Description** tab, for **Availability Zones**, choose **Edit**\. Choose the add icon \(\+\) for one of the subnets for the Availability Zone to add\. The subnet is moved under **Selected subnets**\.

1. Choose **Save**\.

## Add an Availability Zone Using the AWS CLI<a name="as-add-az-aws-cli"></a>

The commands that you'll use depend on whether your load balancer is a Classic Load Balancer in a VPC, a Classic Load Balancer in EC2\-Classic, or an Application Load Balancer\.

**For an Auto Scaling group with a Classic Load Balancer in a VPC**

1. Add a subnet to the Auto Scaling group using the following [update\-auto\-scaling\-group](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command:

   ```
   aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg --vpc-zone-identifier subnet-41767929 subnet-cb663da2 --min-size 2
   ```

1. Verify that the instances in the new subnet are ready to accept traffic from the load balancer using the following [describe\-auto\-scaling\-groups](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command:

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

1. Enable the new subnet for your Classic Load Balancer using the following [attach\-load\-balancer\-to\-subnets](http://docs.aws.amazon.com/cli/latest/reference/elb/attach-load-balancer-to-subnets.html) command:

   ```
   aws elb attach-load-balancer-to-subnets --load-balancer-name my-lb --subnets subnet-41767929
   ```

**For an Auto Scaling group with a Classic Load Balancer in EC2\-Classic**

1. Add an Availability Zone to the Auto Scaling group using the following [update\-auto\-scaling\-group](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command:

   ```
   aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg --availability-zones us-west-2a us-west-2b us-west-2c --min-size 3
   ```

1. Verify that the instances in the new Availability Zone are ready to accept traffic from the load balancer using the following [describe\-auto\-scaling\-groups](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command:

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

1. Enable the new Availability Zone for your Classic Load Balancer using the following [enable\-availability\-zones\-for\-load\-balancer](http://docs.aws.amazon.com/cli/latest/reference/elb/enable-availability-zones-for-load-balancer.html) command:

   ```
   aws elb enable-availability-zones-for-load-balancer --load-balancer-name my-lb --availability-zones us-west-2c
   ```

**For an Auto Scaling group with an Application Load Balancer**

1. Add a subnet to the Auto Scaling group using the following [update\-auto\-scaling\-group](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/update-auto-scaling-group.html) command:

   ```
   aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg --vpc-zone-identifier subnet-41767929 subnet-cb663da2 --min-size 2
   ```

1. Verify that the instances in the new subnet are ready to accept traffic from the load balancer using the following [describe\-auto\-scaling\-groups](http://docs.aws.amazon.com/cli/latest/reference/autoscaling/describe-auto-scaling-groups.html) command:

   ```
   aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name my-asg
   ```

1. Enable the new subnet for your Application Load Balancer using the following [set\-subnets](http://docs.aws.amazon.com/cli/latest/reference/elbv2/set-subnets.html) command:

   ```
   aws elbv2 set-subnets --load-balancer-arn my-lb-arn --subnets subnet-41767929 subnet-cb663da2
   ```