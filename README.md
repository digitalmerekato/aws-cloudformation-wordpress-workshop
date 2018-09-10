# Deploying a Scalable and Highly Available Wordpress Application on AWS using AWS CloudFormation

This is a simple 30-minute hands-on tutorial on how to build and deploy WordPress on to Amazon EC2 instances in an Auto Scaling group with a multi-AZ Amazon RDS database instance for storage using AWS CloudFormation

## Part 1: Concepts

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
  "Resources": {
    "ExampleS3Bucket": {
      "Type": "AWS::S3::Bucket",
      "Properties": {
        "BucketName": "jrdalino-cf-workshop-example"
      }
    }
  }
}
```

YAML
```yaml
Resources:
  ExampleS3Bucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: jrdalino-cf-workshop-example
```

### Outputting and Concatenating Data
You can use Fn:GetAtt to output data
```json
{
  ...
  
  "Public": {
  "Description": "Public IP address of the web server",
  "Value": {
    "Fn::GetAtt": ["WebServerHost", "PublicIp"]
  }
}
```

You can use Fn:Join to concatenate data
```json
{
  ...
  
  "Outputs" : {
    "WebsiteURL" : {
      "Value" : { "Fn::Join" : ["", ["http://", { "Fn::GetAtt" : [ "WebServer", "PublicDnsName" ]}, "/wordpress" ]]},
      "Description" : "WordPress Website"
    }
  }
}
```

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

### VPC's can be created and customized
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

### Let's examine the different sections of a CloudFormation Template
```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  
  "Description" : "...",
  
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

## Part 2: (Optional) Let's build and deploy our VPC CF Template

## Part 3: Let's build our Wordpress CF Template

### Let's add the Description Section
```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  
  "Description" : "AWS CloudFormation Sample Template WordPress_Multi_AZ: WordPress is web software you can use to create a beautiful website or blog. This template installs a highly-available, scalable WordPress deployment using a multi-az Amazon RDS database instance for storage. It demonstrates using the AWS CloudFormation bootstrap scripts to deploy WordPress. **WARNING** This template creates an Amazon EC2 instance, an Application Load Balancer and an Amazon RDS database instance. You will be billed for the AWS resources used if you create a stack from this template.",
  
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

### Let's add the Parameters Section
```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  
  "Description" : "...",
  
  "Parameters" : {
    "VpcId" : {
      "Type" : "AWS::EC2::VPC::Id",
      "Description" : "VpcId of your existing Virtual Private Cloud (VPC)",
      "ConstraintDescription" : "must be the VPC Id of an existing Virtual Private Cloud."
    },

    "Subnets" : {
      "Type" : "List<AWS::EC2::Subnet::Id>",
      "Description" : "The list of SubnetIds in your Virtual Private Cloud (VPC)",
      "ConstraintDescription" : "must be a list of at least two existing subnets associated with at least two different availability zones. They should be residing in the selected Virtual Private Cloud."
    },

    "KeyName": {
      "Description" : "Name of an existing EC2 KeyPair to enable SSH access to the instances",
      "Type": "AWS::EC2::KeyPair::KeyName",
      "ConstraintDescription" : "must be the name of an existing EC2 KeyPair."
    },

    "InstanceType" : {
      "Description" : "WebServer EC2 instance type",
      "Type" : "String",
      "Default" : "t2.small",
      "AllowedValues" : [ "t1.micro", "t2.nano", "t2.micro", "t2.small", "t2.medium", "t2.large", "m1.small", "m1.medium", "m1.large", "m1.xlarge", "m2.xlarge", "m2.2xlarge", "m2.4xlarge", "m3.medium", "m3.large", "m3.xlarge", "m3.2xlarge", "m4.large", "m4.xlarge", "m4.2xlarge", "m4.4xlarge", "m4.10xlarge", "c1.medium", "c1.xlarge", "c3.large", "c3.xlarge", "c3.2xlarge", "c3.4xlarge", "c3.8xlarge", "c4.large", "c4.xlarge", "c4.2xlarge", "c4.4xlarge", "c4.8xlarge", "g2.2xlarge", "g2.8xlarge", "r3.large", "r3.xlarge", "r3.2xlarge", "r3.4xlarge", "r3.8xlarge", "i2.xlarge", "i2.2xlarge", "i2.4xlarge", "i2.8xlarge", "d2.xlarge", "d2.2xlarge", "d2.4xlarge", "d2.8xlarge", "hi1.4xlarge", "hs1.8xlarge", "cr1.8xlarge", "cc2.8xlarge", "cg1.4xlarge"]
,
      "ConstraintDescription" : "must be a valid EC2 instance type."
    },

    "SSHLocation": {
      "Description": "The IP address range that can be used to SSH to the EC2 instances",
      "Type": "String",
      "MinLength": "9",
      "MaxLength": "18",
      "Default": "0.0.0.0/0",
      "AllowedPattern": "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})",
      "ConstraintDescription": "must be a valid IP CIDR range of the form x.x.x.x/x."
    },

    "DBClass" : {
      "Description" : "Database instance class",
      "Type" : "String",
      "Default" : "db.t2.small",
      "AllowedValues" : [ "db.t1.micro", "db.m1.small", "db.m1.medium", "db.m1.large", "db.m1.xlarge", "db.m2.xlarge", "db.m2.2xlarge", "db.m2.4xlarge", "db.m3.medium", "db.m3.large", "db.m3.xlarge", "db.m3.2xlarge", "db.m4.large", "db.m4.xlarge", "db.m4.2xlarge", "db.m4.4xlarge", "db.m4.10xlarge", "db.r3.large", "db.r3.xlarge", "db.r3.2xlarge", "db.r3.4xlarge", "db.r3.8xlarge", "db.m2.xlarge", "db.m2.2xlarge", "db.m2.4xlarge", "db.cr1.8xlarge", "db.t2.micro", "db.t2.small", "db.t2.medium", "db.t2.large"]
,
      "ConstraintDescription" : "must select a valid database instance type."
    },

    "DBName" : {
      "Default": "wordpressdb",
      "Description" : "The WordPress database name",
      "Type": "String",
      "MinLength": "1",
      "MaxLength": "64",
      "AllowedPattern" : "[a-zA-Z][a-zA-Z0-9]*",
      "ConstraintDescription" : "must begin with a letter and contain only alphanumeric characters."
    },

    "DBUser" : {
      "NoEcho": "true",
      "Description" : "The WordPress database admin account username",
      "Type": "String",
      "MinLength": "1",
      "MaxLength": "16",
      "AllowedPattern" : "[a-zA-Z][a-zA-Z0-9]*",
      "ConstraintDescription" : "must begin with a letter and contain only alphanumeric characters."
    },

    "DBPassword" : {
      "NoEcho": "true",
      "Description" : "The WordPress database admin account password",
      "Type": "String",
      "MinLength": "8",
      "MaxLength": "41",
      "AllowedPattern" : "[a-zA-Z0-9]*",
      "ConstraintDescription" : "must contain only alphanumeric characters."
    },

    "MultiAZDatabase": {
      "Default": "false",
      "Description" : "Create a Multi-AZ MySQL Amazon RDS database instance",
      "Type": "String",
      "AllowedValues" : [ "true", "false" ],
      "ConstraintDescription" : "must be either true or false."
    },

    "WebServerCapacity": {
      "Default": "1",
      "Description" : "The initial number of WebServer instances",
      "Type": "Number",
      "MinValue": "1",
      "MaxValue": "5",
      "ConstraintDescription" : "must be between 1 and 5 EC2 instances."
    },

    "DBAllocatedStorage" : {
      "Default": "5",
      "Description" : "The size of the database (Gb)",
      "Type": "Number",
      "MinValue": "5",
      "MaxValue": "1024",
      "ConstraintDescription" : "must be between 5 and 1024Gb."
    }
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

### Let's add the Mappings Section
```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  
  "Description" : "...",
  
  "Parameters" : {
    ...
  },  

  "Mappings" : {
    "AWSInstanceType2Arch" : {
      "t1.micro"    : { "Arch" : "PV64"   },
      "t2.nano"     : { "Arch" : "HVM64"  },
      "t2.micro"    : { "Arch" : "HVM64"  },
      "t2.small"    : { "Arch" : "HVM64"  },
      "t2.medium"   : { "Arch" : "HVM64"  },
      "t2.large"    : { "Arch" : "HVM64"  },
      "m1.small"    : { "Arch" : "PV64"   },
      "m1.medium"   : { "Arch" : "PV64"   },
      "m1.large"    : { "Arch" : "PV64"   },
      "m1.xlarge"   : { "Arch" : "PV64"   },
      "m2.xlarge"   : { "Arch" : "PV64"   },
      "m2.2xlarge"  : { "Arch" : "PV64"   },
      "m2.4xlarge"  : { "Arch" : "PV64"   },
      "m3.medium"   : { "Arch" : "HVM64"  },
      "m3.large"    : { "Arch" : "HVM64"  },
      "m3.xlarge"   : { "Arch" : "HVM64"  },
      "m3.2xlarge"  : { "Arch" : "HVM64"  },
      "m4.large"    : { "Arch" : "HVM64"  },
      "m4.xlarge"   : { "Arch" : "HVM64"  },
      "m4.2xlarge"  : { "Arch" : "HVM64"  },
      "m4.4xlarge"  : { "Arch" : "HVM64"  },
      "m4.10xlarge" : { "Arch" : "HVM64"  },
      "c1.medium"   : { "Arch" : "PV64"   },
      "c1.xlarge"   : { "Arch" : "PV64"   },
      "c3.large"    : { "Arch" : "HVM64"  },
      "c3.xlarge"   : { "Arch" : "HVM64"  },
      "c3.2xlarge"  : { "Arch" : "HVM64"  },
      "c3.4xlarge"  : { "Arch" : "HVM64"  },
      "c3.8xlarge"  : { "Arch" : "HVM64"  },
      "c4.large"    : { "Arch" : "HVM64"  },
      "c4.xlarge"   : { "Arch" : "HVM64"  },
      "c4.2xlarge"  : { "Arch" : "HVM64"  },
      "c4.4xlarge"  : { "Arch" : "HVM64"  },
      "c4.8xlarge"  : { "Arch" : "HVM64"  },
      "g2.2xlarge"  : { "Arch" : "HVMG2"  },
      "g2.8xlarge"  : { "Arch" : "HVMG2"  },
      "r3.large"    : { "Arch" : "HVM64"  },
      "r3.xlarge"   : { "Arch" : "HVM64"  },
      "r3.2xlarge"  : { "Arch" : "HVM64"  },
      "r3.4xlarge"  : { "Arch" : "HVM64"  },
      "r3.8xlarge"  : { "Arch" : "HVM64"  },
      "i2.xlarge"   : { "Arch" : "HVM64"  },
      "i2.2xlarge"  : { "Arch" : "HVM64"  },
      "i2.4xlarge"  : { "Arch" : "HVM64"  },
      "i2.8xlarge"  : { "Arch" : "HVM64"  },
      "d2.xlarge"   : { "Arch" : "HVM64"  },
      "d2.2xlarge"  : { "Arch" : "HVM64"  },
      "d2.4xlarge"  : { "Arch" : "HVM64"  },
      "d2.8xlarge"  : { "Arch" : "HVM64"  },
      "hi1.4xlarge" : { "Arch" : "HVM64"  },
      "hs1.8xlarge" : { "Arch" : "HVM64"  },
      "cr1.8xlarge" : { "Arch" : "HVM64"  },
      "cc2.8xlarge" : { "Arch" : "HVM64"  }
    },

    "AWSInstanceType2NATArch" : {
      "t1.micro"    : { "Arch" : "NATPV64"   },
      "t2.nano"     : { "Arch" : "NATHVM64"  },
      "t2.micro"    : { "Arch" : "NATHVM64"  },
      "t2.small"    : { "Arch" : "NATHVM64"  },
      "t2.medium"   : { "Arch" : "NATHVM64"  },
      "t2.large"    : { "Arch" : "NATHVM64"  },
      "m1.small"    : { "Arch" : "NATPV64"   },
      "m1.medium"   : { "Arch" : "NATPV64"   },
      "m1.large"    : { "Arch" : "NATPV64"   },
      "m1.xlarge"   : { "Arch" : "NATPV64"   },
      "m2.xlarge"   : { "Arch" : "NATPV64"   },
      "m2.2xlarge"  : { "Arch" : "NATPV64"   },
      "m2.4xlarge"  : { "Arch" : "NATPV64"   },
      "m3.medium"   : { "Arch" : "NATHVM64"  },
      "m3.large"    : { "Arch" : "NATHVM64"  },
      "m3.xlarge"   : { "Arch" : "NATHVM64"  },
      "m3.2xlarge"  : { "Arch" : "NATHVM64"  },
      "m4.large"    : { "Arch" : "NATHVM64"  },
      "m4.xlarge"   : { "Arch" : "NATHVM64"  },
      "m4.2xlarge"  : { "Arch" : "NATHVM64"  },
      "m4.4xlarge"  : { "Arch" : "NATHVM64"  },
      "m4.10xlarge" : { "Arch" : "NATHVM64"  },
      "c1.medium"   : { "Arch" : "NATPV64"   },
      "c1.xlarge"   : { "Arch" : "NATPV64"   },
      "c3.large"    : { "Arch" : "NATHVM64"  },
      "c3.xlarge"   : { "Arch" : "NATHVM64"  },
      "c3.2xlarge"  : { "Arch" : "NATHVM64"  },
      "c3.4xlarge"  : { "Arch" : "NATHVM64"  },
      "c3.8xlarge"  : { "Arch" : "NATHVM64"  },
      "c4.large"    : { "Arch" : "NATHVM64"  },
      "c4.xlarge"   : { "Arch" : "NATHVM64"  },
      "c4.2xlarge"  : { "Arch" : "NATHVM64"  },
      "c4.4xlarge"  : { "Arch" : "NATHVM64"  },
      "c4.8xlarge"  : { "Arch" : "NATHVM64"  },
      "g2.2xlarge"  : { "Arch" : "NATHVMG2"  },
      "g2.8xlarge"  : { "Arch" : "NATHVMG2"  },
      "r3.large"    : { "Arch" : "NATHVM64"  },
      "r3.xlarge"   : { "Arch" : "NATHVM64"  },
      "r3.2xlarge"  : { "Arch" : "NATHVM64"  },
      "r3.4xlarge"  : { "Arch" : "NATHVM64"  },
      "r3.8xlarge"  : { "Arch" : "NATHVM64"  },
      "i2.xlarge"   : { "Arch" : "NATHVM64"  },
      "i2.2xlarge"  : { "Arch" : "NATHVM64"  },
      "i2.4xlarge"  : { "Arch" : "NATHVM64"  },
      "i2.8xlarge"  : { "Arch" : "NATHVM64"  },
      "d2.xlarge"   : { "Arch" : "NATHVM64"  },
      "d2.2xlarge"  : { "Arch" : "NATHVM64"  },
      "d2.4xlarge"  : { "Arch" : "NATHVM64"  },
      "d2.8xlarge"  : { "Arch" : "NATHVM64"  },
      "hi1.4xlarge" : { "Arch" : "NATHVM64"  },
      "hs1.8xlarge" : { "Arch" : "NATHVM64"  },
      "cr1.8xlarge" : { "Arch" : "NATHVM64"  },
      "cc2.8xlarge" : { "Arch" : "NATHVM64"  }
    }
,
    "AWSRegionArch2AMI" : {
      "us-east-1"        : {"PV64" : "ami-2a69aa47", "HVM64" : "ami-97785bed", "HVMG2" : "ami-0a6e3770"},
      "us-west-2"        : {"PV64" : "ami-7f77b31f", "HVM64" : "ami-f2d3638a", "HVMG2" : "ami-ee15a196"},
      "us-west-1"        : {"PV64" : "ami-a2490dc2", "HVM64" : "ami-824c4ee2", "HVMG2" : "ami-0da4a46d"},
      "eu-west-1"        : {"PV64" : "ami-4cdd453f", "HVM64" : "ami-d834aba1", "HVMG2" : "ami-af8013d6"},
      "eu-west-2"        : {"PV64" : "NOT_SUPPORTED", "HVM64" : "ami-403e2524", "HVMG2" : "NOT_SUPPORTED"},
      "eu-west-3"        : {"PV64" : "NOT_SUPPORTED", "HVM64" : "ami-8ee056f3", "HVMG2" : "NOT_SUPPORTED"},
      "eu-central-1"     : {"PV64" : "ami-6527cf0a", "HVM64" : "ami-5652ce39", "HVMG2" : "ami-1d58ca72"},
      "ap-northeast-1"   : {"PV64" : "ami-3e42b65f", "HVM64" : "ami-ceafcba8", "HVMG2" : "ami-edfd658b"},
      "ap-northeast-2"   : {"PV64" : "NOT_SUPPORTED", "HVM64" : "ami-863090e8", "HVMG2" : "NOT_SUPPORTED"},
      "ap-northeast-3"   : {"PV64" : "NOT_SUPPORTED", "HVM64" : "ami-83444afe", "HVMG2" : "NOT_SUPPORTED"},
      "ap-southeast-1"   : {"PV64" : "ami-df9e4cbc", "HVM64" : "ami-68097514", "HVMG2" : "ami-c06013bc"},
      "ap-southeast-2"   : {"PV64" : "ami-63351d00", "HVM64" : "ami-942dd1f6", "HVMG2" : "ami-85ef12e7"},
      "ap-south-1"       : {"PV64" : "NOT_SUPPORTED", "HVM64" : "ami-531a4c3c", "HVMG2" : "ami-411e492e"},
      "us-east-2"        : {"PV64" : "NOT_SUPPORTED", "HVM64" : "ami-f63b1193", "HVMG2" : "NOT_SUPPORTED"},
      "ca-central-1"     : {"PV64" : "NOT_SUPPORTED", "HVM64" : "ami-a954d1cd", "HVMG2" : "NOT_SUPPORTED"},
      "sa-east-1"        : {"PV64" : "ami-1ad34676", "HVM64" : "ami-84175ae8", "HVMG2" : "NOT_SUPPORTED"},
      "cn-north-1"       : {"PV64" : "ami-77559f1a", "HVM64" : "ami-cb19c4a6", "HVMG2" : "NOT_SUPPORTED"},
      "cn-northwest-1"   : {"PV64" : "ami-80707be2", "HVM64" : "ami-3e60745c", "HVMG2" : "NOT_SUPPORTED"}
    }
  },
  
  "Resources" : {
    ...
  },
  
  "Outputs" : {
    ...
  }  
}
```

### Let's add the Resources Section
```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  
  "Description" : "...",
  
  "Parameters" : {
    ...
  },  

  "Mappings" : {
    ...
  },
  
  "Resources" : {

    "ApplicationLoadBalancer" : {
      "Type" : "AWS::ElasticLoadBalancingV2::LoadBalancer",
      "Properties" : {
        "Subnets" : { "Ref" : "Subnets"}
      }
    },

    "ALBListener" : {
      "Type" : "AWS::ElasticLoadBalancingV2::Listener",
      "Properties" : {
        "DefaultActions" : [{
          "Type" : "forward",
          "TargetGroupArn" : { "Ref" : "ALBTargetGroup" }
        }],
        "LoadBalancerArn" : { "Ref" : "ApplicationLoadBalancer" },
        "Port" : "80",
        "Protocol" : "HTTP"
      }
    },

    "ALBTargetGroup" : {
      "Type" : "AWS::ElasticLoadBalancingV2::TargetGroup",
      "Properties" : {
        "HealthCheckPath" : "/wordpress/wp-admin/install.php",  
        "HealthCheckIntervalSeconds" : 10,
        "HealthCheckTimeoutSeconds" : 5,
        "HealthyThresholdCount" : 2,
        "Port" : 80,
        "Protocol" : "HTTP",
        "UnhealthyThresholdCount" : 5,
        "VpcId" : {"Ref" : "VpcId"},       
        "TargetGroupAttributes" :
          [ { "Key" : "stickiness.enabled", "Value" : "true" },
            { "Key" : "stickiness.type", "Value" : "lb_cookie" },
            { "Key" : "stickiness.lb_cookie.duration_seconds", "Value" : "30" }
        ]
      }
    },
   
    "WebServerSecurityGroup" : {
      "Type" : "AWS::EC2::SecurityGroup",
      "Properties" : {
        "GroupDescription" : "Enable HTTP access via port 80 locked down to the load balancer + SSH access",
        "SecurityGroupIngress" : [
          {"IpProtocol" : "tcp", "FromPort" : "80", "ToPort" : "80", "SourceSecurityGroupId" : {"Fn::Select" : [0, {"Fn::GetAtt" : ["ApplicationLoadBalancer", "SecurityGroups"]}]}},
          {"IpProtocol" : "tcp", "FromPort" : "22", "ToPort" : "22", "CidrIp" : { "Ref" : "SSHLocation"}}
        ],
        "VpcId" : { "Ref" : "VpcId" }
      }
    },

    "WebServerGroup" : {
      "Type" : "AWS::AutoScaling::AutoScalingGroup",
      "Properties" : {
        "VPCZoneIdentifier" : { "Ref" : "Subnets" },
        "LaunchConfigurationName" : { "Ref" : "LaunchConfig" },
        "MinSize" : "1",
        "MaxSize" : "5",
        "DesiredCapacity" : { "Ref" : "WebServerCapacity" },
        "TargetGroupARNs" : [ { "Ref" : "ALBTargetGroup" } ]
      },
      "CreationPolicy" : {
        "ResourceSignal" : {
          "Timeout" : "PT15M"
        }
      },
      "UpdatePolicy": {
        "AutoScalingRollingUpdate": {
          "MinInstancesInService": "1",
          "MaxBatchSize": "1",
          "PauseTime" : "PT15M",
          "WaitOnResourceSignals": "true"
        }
      }
    },

    "LaunchConfig": {
      "Type" : "AWS::AutoScaling::LaunchConfiguration",
      "Metadata" : {
        "AWS::CloudFormation::Init" : {
          "configSets" : {
            "wordpress_install" : ["install_cfn", "install_wordpress" ]
          },
          "install_cfn" : {
            "files": {
              "/etc/cfn/cfn-hup.conf": {
                "content": { "Fn::Join": [ "", [
                  "[main]\n",
                  "stack=", { "Ref": "AWS::StackId" }, "\n",
                  "region=", { "Ref": "AWS::Region" }, "\n"
                ]]},
                "mode"  : "000400",
                "owner" : "root",
                "group" : "root"
              },
              "/etc/cfn/hooks.d/cfn-auto-reloader.conf": {
                "content": { "Fn::Join": [ "", [
                  "[cfn-auto-reloader-hook]\n",
                  "triggers=post.update\n",
                  "path=Resources.LaunchConfig.Metadata.AWS::CloudFormation::Init\n",
                  "action=/opt/aws/bin/cfn-init -v ",
                          "         --stack ", { "Ref" : "AWS::StackName" },
                          "         --resource LaunchConfig ",
                          "         --configsets wordpress_install ",
                          "         --region ", { "Ref" : "AWS::Region" }, "\n"
                ]]},          
                "mode"  : "000400",
                "owner" : "root",
                "group" : "root"
              }
            },
            "services" : {
              "sysvinit" : {
                "cfn-hup" : { "enabled" : "true", "ensureRunning" : "true",
                              "files" : ["/etc/cfn/cfn-hup.conf", "/etc/cfn/hooks.d/cfn-auto-reloader.conf"]}
              }
            }
          },

          "install_wordpress" : {
            "packages" : {
              "yum" : {
                "php"       : [],
                "php-mysql" : [],
                "mysql"     : [],
                "httpd"     : []
              }
            },
            "sources" : {
              "/var/www/html" : "http://wordpress.org/latest.tar.gz"
            },
            "files" : {
              "/tmp/create-wp-config" : {
                "content" : { "Fn::Join" : [ "", [
                  "#!/bin/bash\n",
                  "cp /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php\n",
                  "sed -i \"s/'database_name_here'/'",{ "Ref" : "DBName" }, "'/g\" wp-config.php\n",
                  "sed -i \"s/'username_here'/'",{ "Ref" : "DBUser" }, "'/g\" wp-config.php\n",
                  "sed -i \"s/'password_here'/'",{ "Ref" : "DBPassword" }, "'/g\" wp-config.php\n",
                  "sed -i \"s/'localhost'/'",{ "Fn::GetAtt" : [ "DBInstance", "Endpoint.Address" ] }, "'/g\" wp-config.php\n"
                ]]},
                "mode" : "000500",
                "owner" : "root",
                "group" : "root"
              }
            },
            "commands" : {
              "01_configure_wordpress" : {
                "command" : "/tmp/create-wp-config",
                "cwd" : "/var/www/html/wordpress"
              }
            },
            "services" : {
              "sysvinit" : {
                "httpd" : { "enabled" : "true", "ensureRunning" : "true" }
              }
            }
          }
        }
      },
      "Properties": {
        "ImageId" : { "Fn::FindInMap" : [ "AWSRegionArch2AMI", { "Ref" : "AWS::Region" },
                          { "Fn::FindInMap" : [ "AWSInstanceType2Arch", { "Ref" : "InstanceType" }, "Arch" ] } ] },
        "InstanceType"   : { "Ref" : "InstanceType" },
        "SecurityGroups" : [ {"Ref" : "WebServerSecurityGroup"} ],
        "KeyName"        : { "Ref" : "KeyName" },
        "UserData" : { "Fn::Base64" : { "Fn::Join" : ["", [
                       "#!/bin/bash -xe\n",
                       "yum update -y aws-cfn-bootstrap\n",

                       "/opt/aws/bin/cfn-init -v ",
                       "         --stack ", { "Ref" : "AWS::StackName" },
                       "         --resource LaunchConfig ",
                       "         --configsets wordpress_install ",
                       "         --region ", { "Ref" : "AWS::Region" }, "\n",

                       "/opt/aws/bin/cfn-signal -e $? ",
                       "         --stack ", { "Ref" : "AWS::StackName" },
                       "         --resource WebServerGroup ",
                       "         --region ", { "Ref" : "AWS::Region" }, "\n"
        ]]}}
      }
    },

    "DBEC2SecurityGroup": {
      "Type": "AWS::EC2::SecurityGroup",
      "Properties" : {
        "GroupDescription": "Open database for access",
        "SecurityGroupIngress" : [{
        "IpProtocol" : "tcp",
        "FromPort" : "3306",
        "ToPort" : "3306",
        "SourceSecurityGroupId" : { "Ref" : "WebServerSecurityGroup" }
        }],
        "VpcId" : { "Ref" : "VpcId" }
      }
    },

    "DBInstance" : {
      "Type": "AWS::RDS::DBInstance",
      "Properties": {
        "DBName"            : { "Ref" : "DBName" },
        "Engine"            : "MySQL",
        "MultiAZ"           : { "Ref": "MultiAZDatabase" },
        "MasterUsername"    : { "Ref" : "DBUser" },
        "MasterUserPassword": { "Ref" : "DBPassword" },
        "DBInstanceClass"   : { "Ref" : "DBClass" },
        "AllocatedStorage"  : { "Ref" : "DBAllocatedStorage" },
        "VPCSecurityGroups" : [{ "Fn::GetAtt": [ "DBEC2SecurityGroup", "GroupId" ]}]
      }
    }
  },  
 
  "Outputs" : {
    "WebsiteURL" : {
      ...
    },
  }  
}
```

### Finally, Let's add the Outputs Section
```json
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  
  "Description" : "...",
  
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
    "WebsiteURL" : {
      "Value" : { "Fn::Join" : ["", ["http://", { "Fn::GetAtt" : [ "ApplicationLoadBalancer", "DNSName" ]}, "/wordpress" ]]},
      "Description" : "WordPress Website"
    }
  }
}
```

## Part 4: Deploy the Wordpress CF Temlate

### Log on the the AWS Console 
- AWS Console > Services > CloudFormation
- Click on Create Stack
- Select Upload a template to Amazon S3 and click Next
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

### Click on Next > Next > Create > Go to CloudFormation Stacks, Select Stack > Outputs Tab, Configure Wordpress

## Congratulations, you've just completed this tutorial! Next Steps:

### VPC Peering
You can create multiple VPCs inside one template
You can enable VPC Peering using CloudFormation

### Route 53 is Supported
You can create new hosted zones or update existing hosted zones using CloudFormation Templates
This includes adding or changing items such as A records, Aliases, CNAMEs, etc

### Chef & Puppet Integration
CloudFormation supports Chef & Puppet Integration meaning that you can deploy and configure right down to the application layer

Bootstrap scripts are also supported enabling you to install packages, files, and services on your EC2 Instances by simply describing them in your CloudFormation template

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
