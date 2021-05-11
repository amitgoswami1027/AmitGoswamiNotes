# AWS - BLUE GREEN DEPLOYMENT
* Blue/green deployments provide near zero-downtime release and rollback capabilities.
* Blue/green deployment works by shifting traffic between two identical environments that are running different versions of the application
* Blue environment represents the current application version serving production traffic.
* In parallel, the green environment is staged running a different version of your application.
* After the green environment is ready and tested, production traffic is redirected from blue to green.
* If any problems are identified, you can roll back by reverting traffic back to the blue environment.

## AWS SERVICES
### Route 53
* Route 53 is a highly available and scalable authoritative DNS service that route user requests
* Route 53 with its DNS service allows administrators to direct traffic by simply updating DNS records in the hosted zone
* TTL can be adjusted for resource records to be shorter which allow record changes to propagate faster to clients

### Elastic Load Balancing
* Elastic Load Balancing distributes incoming application traffic across EC2 instances
* Elastic Load Balancing scales in response to incoming requests, performs health checking against Amazon EC2 resources, and naturally integrates with other AWS tools, such as Auto Scaling.
* ELB also helps perform health checks of EC2 instances to route traffic only to the healthy instances

### Auto Scaling
* Auto Scaling allows different versions of launch configuration, which define templates used to launch EC2 instances, to be attached to an Auto Scaling group to enable blue/green deployment.
* Auto Scaling’s termination policies and Standby state enable blue/green deployment
* Termination policies in Auto Scaling groups to determine which EC2 instances to remove during a scaling action.
* Auto Scaling also allows instances to be placed in Standby state, instead of termination, which helps with quick rollback when required
* Auto Scaling with Elastic Load Balancing can be used to balance and scale the traffic

### Elastic Beanstalk
* Elastic Load Balancing distributes incoming application traffic across EC2 instances
* Elastic Load Balancing scales in response to incoming requests, performs health checking against Amazon EC2 resources, and naturally integrates with other AWS tools, such as Auto Scaling.
* ELB also helps perform health checks of EC2 instances to route traffic only to the healthy instances
* Elastic Beanstalk makes it easy to run multiple versions of the application and provides capabilities to swap the environment URLs, facilitating blue/green deployment.
* Elastic Beanstalk supports Auto Scaling and Elastic Load Balancing, both of which enable blue/green deployment

### Auto Scaling
* Auto Scaling allows different versions of launch configuration, which define templates used to launch EC2 instances, to be attached to an Auto Scaling group to enable blue/green deployment.
* Auto Scaling’s termination policies and Standby state enable blue/green deployment
* Termination policies in Auto Scaling groups to determine which EC2 instances to remove during a scaling action.
* Auto Scaling also allows instances to be placed in Standby state, instead of termination, which helps with quick rollback when required
* Auto Scaling with Elastic Load Balancing can be used to balance and scale the traffic

### OpsWorks
* OpsWorks has the concept of stacks, which are logical groupings of AWS resources with a common purpose & should be logically managed together
* Stacks are made of one or more layers with each layer represents a set of EC2 instances that serve a particular purpose, such as serving applications or hosting a database server.
* OpsWorks simplifies cloning entire stacks when preparing for blue/green environments.

### CloudFormation
* CloudFormation helps describe the AWS resources through JSON formatted templates and provides automation capabilities for provisioning blue/green environments and 
facilitating updates to switch traffic, whether through Route 53 DNS, Elastic Load Balancing, etc
* CloudFormation provides infrastructure as code strategy, where infrastructure is provisioned and managed using code and software development techniques, such as version 
control and continuous integration, in a manner similar to how application code is treated

### CloudWatch
* CloudWatch monitoring can provide early detection of application health in blue/green deployments

# Deployment Techniques - DNS Routing using Route 53
* Route 53 DNS service can help switch traffic from the blue environment to the green and vice versa, if rollback is necessary
* Route 53 can help either switch the traffic completely or through a weighted distribution
* Weighted distribution
  * helps distribute percentage of traffic to go to the green environment and gradually update the weights until the green environment carries the full production traffic
  * provides the ability to perform canary analysis where a small percentage of production traffic is introduced to a new environment
  * helps manage cost by using auto scaling for instances to scale based on the actual demand
* Route 53 can handle Public or Elastic IP address, Elastic Load Balancer, Elastic Beanstalk environment web tiers etc.

![image](https://user-images.githubusercontent.com/13011167/117758931-89886300-b240-11eb-99e6-dcdbaa6d53ea.png)

# Auto Scaling Group Swap Behind Elastic Load Balancer
![image](https://user-images.githubusercontent.com/13011167/117759023-ae7cd600-b240-11eb-8937-8cd3496b824c.png)

* Elastic Load Balancing with Auto Scaling to manage EC2 resources as per the demand can be used for Blue Green deployments
* Multiple Auto Scaling groups can be attached to the Elastic Load Balancer
* Green ASG can be attached to an existing ELB while Blue ASG is already attached to the ELB to serve traffic
* ELB would start routing requests to the Green Group as for HTTP/S listener it uses a least outstanding requests routing algorithm
* Green group capacity can be increased to process more traffic while the Blue group capacity can be reduced either by terminating the instances or by putting the 
instances in a standby mode
* Standby is a good option because if roll back to the blue environment needed, blue server instances can be put back in service and they’re ready to go
* If no issues with the Green group, the blue group can be decommissioned by adjusting the group size to zero

# Update Auto Scaling Group Launch Configurations
![image](https://user-images.githubusercontent.com/13011167/117759133-e126ce80-b240-11eb-8023-8c5839efbea1.png)

* Auto Scaling groups have their own launch configurations which define template for EC2 instances to be launched
* Auto Scaling group can have only one launch configuration at a time, and it can’t be modified. If needs modification, a new launch configuration can be created and 
attached to the existing Auto Scaling Group
* After a new launch configuration is in place, any new instances that are launched use the new launch configuration parameters, but existing instances are not affected.
* When Auto Scaling removes instances (referred to as scaling in) from the group, the default termination policy is to remove instances with the oldest launch configuration
* To deploy the new version of the application in the green environment, update the Auto Scaling group with the new launch configuration, and then scale the Auto Scaling group 
to twice its original size.
* Then, shrink the Auto Scaling group back to the original size
* To perform a rollback, update the Auto Scaling group with the old launch configuration. Then, do the preceding steps in reverse

# Elastic Beanstalk Application Environment Swap

![image](https://user-images.githubusercontent.com/13011167/117762660-382fa200-b247-11eb-9435-038adcf0268b.png)

* Elastic Beanstalk multiple environment and environment url swap feature helps enable Blue Green deployment
* Elastic Beanstalk can be used to host the blue environment exposed via URL to access the environment
* Elastic Beanstalk provides several deployment policies, ranging from policies that perform an in-place update on existing instances, to immutable deployment using a 
set of new instances.
* Elastic Beanstalk performs an in-place update when the application versions are updated, however application may become unavailable to users for a short period of time.
* To avoid the downtime, a new version can be deployed to a separate Green environment with its own URL, launched with the existing environment’s configuration
* Elastic Beanstalk’s Swap Environment URLs feature can be used to promote the green environment to serve production traffic
* Elastic Beanstalk performs a DNS switch, which typically takes a few minutes
* To perform a rollback, invoke Swap Environment URL again.

# Clone a Stack in AWS OpsWorks and Update DNS
* OpsWorks can be used to create
  * Blue environment stack with the current version of the application and serving production traffic
  * Green environment stack with the newer version of the application and is not receiving any traffic
* To promote to the green environment/stack into production, update DNS records to point to the green environment/stack’s load balancer

![image](https://user-images.githubusercontent.com/13011167/117762808-7200a880-b247-11eb-84ab-d85d0c1e6a65.png)




### Links Implementation
* https://jayendrapatil.com/aws-blue-green-deployment/
* https://youtu.be/aX54mhZbN58
