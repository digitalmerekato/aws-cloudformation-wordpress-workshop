# Deploying a Scalable and Highly Available Wordpress Application on AWS using AWS CloudFormation

This is a simple 30-minute hands-on tutorial on how to build and deploy WordPress on to Amazon EC2 instances in an Auto Scaling group with a multi-AZ Amazon RDS database instance for storage using AWS CloudFormation

## Part 0: Concepts

### What is CloudFormation?
It allows you to take what was once traditional hardware infrastructure and convert it into code.

CloudFormation gives developers and system administrators an easy way to create and manage a collection of related AWS resources, provisioning and updating them in an orderly and predictable fashion.

You don’t need to figure out the order for provisioning AWS services or the subtleties of making those dependencies work. CloudFormation takes care of this for you.

After the AWS resources are deployed, you can modify and update them in a controlled and predictable way, in effect applying version control to your AWS infrastructure the same way you do with your software.

### Templates, Stacks and Change Sets
A CloudFormation Templates is essentially an architectural diagram and a CloudFormation Stack is the end result of that diagram (i.e. what is actually provisioned).

You create, update, and delete a collection of resources by creating, updating, and deleting stacks using CloudFormation templates.

CloudFormation templates are in the JSON format or YAML. 

### Elements Of A Template
Mandatory Elements
- List of AWS Resources and their associated configuration values

Optional Elements
- The template’s file format and version number
- Templates Parameters: The input values that are supplied at stack creation time. Limit of 60.
- Output Values: The output values required once a stack has finished building (such as the public IP address, ELB address, etc). Limit of 60.
- List of data tables: Used to look up static configuration values such as AMI’s etc

### A Simple Template
JSON
```json
{
  "Resources" : {
    "HelloBucket" : {
      "Type": "AWS::S3:Bucket"
    }
  }
}
```

YAML
```yaml
Resources:
  HelloBucket:
    Type: AWS::S3:Bucket
```

### Outputting Data
You can use Fn:GetAtt to output data
```json
{
“Public”: {
  “Description”: “Public IP address of the web server”,
  “Value”: {
    “Fn::GetAtt”:  [
      “WenServerHost”,
      “PublicIp”
    ]
  }
}
```

### Chef & Puppet Integration
CloudFormation supports Chef & Puppet Integration meaning that you can deploy and configure right down to the application layer

Bootstrap scripts are also supported enabling you to install packages, files, and services on your EC2 Instances by simply describing them in your CloudFormation template

### Stack creation errors
By default, the “automatic rollback on error” feature is enabled. This will cause all AWS resources that AWS CloudFormation created successfully for a stack up to the point where an error occured to tb deleted.

Note: You will be charged for resources that are provisioned, even if there is an error.

### Stacks can wait for applications
AWS CloudFormation provides a WaitCondition resource that acts as a barrier, blocking the creation of other resources until a completion signal is received from an external source, such as your application or management system.

### You can specify deletion policies
AWS CloudFormation allows you to define deletion policies for resources in the template. You can specify that snapshots be created for Amazon EBS volumes or Amazon RDS Database instances before they are deleted.

You can also specify that a resource should be preserved and not deleted when the stack is deleted. This is useful for preserving Amazon S3 buckets when the stack is deleted.

### You can update your stack after it is created
You can use AWS CloudFormation to modify and update the resources in your existing stacks in a controlled and predictable way. By using templates to manage your stack changes, you have the ability to apply version control to your AWS infrastructure, just as you do with the software running on it.

### CloudFormation & Roles
CloudFormation can be used to create roles in IAM
CloudFormation can be used to grant EC2 instances access to roles

### VPC's can be created and Customized
CloudFormation supports creating:
- VPCs
- Subnets 
- Gateways
- Route Tables
- Networks ACLs
- Elastic IPs
- Amazon EC2 Instances
- EC2 Security Groups
- Auto Scaling Groups
- Elastic Load Balancers
- Amazon RDS Database Instances
- Amazon RDS Security Groups
- and many more!
Full list here: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-supported-resources.html

You can specify IP address ranges both in terms of CIDR ranges as well as individual IP addresses for specific instances. You can specify pre-existing EIPs.

### Let's examine the different sections
```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  
  "Description" : "lorem ipsum",
  
  "Parameters" : {
    ...
  },  

  "Mappings" : {
    ...
  },
  
  "Resources" : {
    ...
  },
  
  "Outputs" : {
    ...
  }  
}
```

## Part 1: Let's build our VPC CF Template

### Let's examine the different sections
```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  
  "Description" : "lorem ipsum",
  
  "Parameters" : {
    ...
  },  

  "Mappings" : {
    ...
  },
  
  "Resources" : {
    ...
  },
  
  "Outputs" : {
    ...
  }  
}
```

## Part 2: Deploy the VPC CF Template

## Part 3: Let's build our Wordpress CF Template

### Let's examine the different sections
```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  
  "Description" : "lorem ipsum",
  
  "Parameters" : {
    ...
  },  

  "Mappings" : {
    ...
  },
  
  "Resources" : {
    "ApplicationLoadBalancer" : {
      ...
    },
    "ALBListener" : {      
      ...
    },
    "ALBTargetGroup" : {      
      ...
    },
    "WebServerSecurityGroup" : {      
      ...
    },
    "WebServerGroup" : {      
      ...
    },
    "LaunchConfig" : {      
      ...
    },
    "DBEC2SecurityGroup" : {      
      ...
    },
    "DBInstance" : {      
      ...
    },     
 
  "Outputs" : {
    "WebsiteURL" : {
      ...
    },
  }  
}
```

## Part 4: Deploy the Wordpress CF Temlate
- Make sure you are using the correct region

### Specify Details
- Stack name: WordPress-sample-scalable

### Parameters
- DBAllocatedStorage: 5 (The size of the database (Gb))
- DBClass: db.t2.small (Database instance class)
- DBName: wordpressdb (The WordPress database name)
- DBPassword: <your password here> (The WordPress database admin account password)
- DBUser: <your user name here> (The WordPress database admin account username)
- InstanceType: t2.small (WebServer EC2 instance type)
- KeyName: <your key pair here> (Name of an existing EC2 KeyPair to enable SSH access to the instances)
- MultiAZDatabase: true (Create a Multi-AZ MySQL Amazon RDS database instance)
- SSHLocation: 0.0.0.0/0 (The IP address range that can be used to SSH to the EC2 instances)
- Subnets: <your subnets here>
- VpcId: <your vpc here>
- WebServerCapacity: 1

## Congratulations, you've just completed this tutorial! Next Steps:

### VPC Peering
You can create multiple VPCs inside one template
You can enable VPC Peering using CloudFormation

### Route 53 is Supported
You can create new hosted zones or update existing hosted zones using CloudFormation Templates
This includes adding or changing items such as A records, Aliases, CNAMEs, etc

### CloudFormation Limits
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cloudformation-limits.html

### Using CloudFormer to Create AWS CloudFormation Templates from Existing AWS Resources
CloudFormer is a template creation beta tool that creates an AWS CloudFormation template from existing AWS resources in your account. You select any supported AWS resources that are running in your account, and CloudFormer creates a template in an Amazon S3 bucket.

### AWS Best Practices
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html

### AWS Sample Templates
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-sample-templates.html

### VSCode Extension
CloudFormation support for Visual Studio Code
https://marketplace.visualstudio.com/items?itemName=aws-scripting-guy.cform
