### Cloud Formation Template for AWS Compute Components

Using the below syntax we will create AWS Compute Resources containing:
1. EC2
2. Launch Congfiguration and Auto Scaling Group
3. Application Load Balancer and Target Group

Now, we will discuss about some terms which you will be seeing inside the code:
1. Parameters section can be used to customize your templates. Parameters enable you to input custom values to your template each time you create or update a stack. Parameters can be used as reference in Resources section using the intrinsic function Ref.
2. Resources section is used to declare AWS resources you want to include in the stack, such as an Amazon S3 bucket or an Amazon VPC.
3. Outputs is an optional section where you can declare output values that you can import into other stacks (to create cross-stack references). For example, you can output the S3 bucket name for a stack to make the bucket easier to find.

### Documentation Referred:

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-vpc.html

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet.html

Now, here is the complete template in JSON for creating the EC2 instance.

### ec2.json

```sh
{
  "Description": "Create an EC2 instance by AWS CloudFormation",
 "Parameters" : {
  "EC2" : {
    "Type" : "String",
    "Default" : "myEC2",
    "Description" : "Enter the name of EC2"
  },
  "InstanceTypeParameter" : {
    "Type" : "String",
    "Default" : "t2.micro",
    "AllowedValues" : ["t2.micro", "m1.small", "m1.large"],
    "Description" : "Enter t2.micro, m1.small, or m1.large. Default is t2.micro."
  },
  "ImageId" : {
  "Description" : "Please enter Image Id you want to use",
  "Type" : "String",
  "Default" : "ami-0be2609ba883822ec"
  },
  "AvailabilityZone" : {
  "Description" : "Please enter the AZ you want to put your Instance in",
  "Type" : "String",
  "Default" : "us-east-1a"
  },
  "SubnetID" : {
  "Description" : "Please enter the Subnet ID",
  "Type" : "String"
  },
  "NetworkStackName" : {
  "Description" : "Please enter the VPC Stack Name",
  "Type" : "String",
  "Default" : "VPC"
  }
  
  
},

  "Resources": {
    "SecurityGroupDemoSvrTraffic": {
      "Type": "AWS::EC2::SecurityGroup",
      "Properties": {
        "GroupName": "sgCFN",
        "SecurityGroupIngress": [
          {
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIp": "0.0.0.0/0",
            "Description": "For traffic from Internet"
          }
        ],
        "GroupDescription": "Security Group for demo server",
		"VpcId" : {
          "Fn::ImportValue" : {
            "Fn::Sub" : "${NetworkStackName}-VpcId"

          }
        }
      }
    },
    "EC2InstanceDemoSvr": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "AvailabilityZone": {"Ref":"AvailabilityZone"},
        "BlockDeviceMappings": [
          {
            "DeviceName": "/dev/sdm",
            "Ebs": {
              "DeleteOnTermination": "true",
              "VolumeSize": "8",
              "VolumeType": "gp2"
            }
          }
        ],
        "ImageId": { "Ref" : "ImageId" },
        "InstanceType": { "Ref" : "InstanceTypeParameter" },
        "KeyName": "keypublic",
        "NetworkInterfaces": [
          {
            "Description": "Primary network interface",
            "DeviceIndex": "0",
            "SubnetId": {"Ref":"SubnetID"},
			"AssociatePublicIpAddress" : "True",
            "GroupSet": [
              {
                "Ref": "SecurityGroupDemoSvrTraffic"
              }
            ]
          }
        ]
      }
    },
	"RootRole": {
            "Type": "AWS::IAM::Role",
            "Properties": {
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Effect": "Allow",
                            "Principal": {
                                "Service": [
                                    "ec2.amazonaws.com"
                                ]
                            },
                            "Action": [
                                "sts:AssumeRole"
                            ]
                        }
                    ]
                },
                "Path": "/",
                "Policies": [
                    {
                        "PolicyName": "root",
                        "PolicyDocument": {
                            "Version": "2012-10-17",
                            "Statement": [
                                {
                                    "Effect": "Allow",
                                    "Action": "*",
                                    "Resource": "*"
                                }
                            ]
                        }
                    }
                ]
            }
        },
        "RootInstanceProfile": {
            "Type": "AWS::IAM::InstanceProfile",
            "Properties": {
                "Path": "/",
                "Roles": [
                    {
                        "Ref": "RootRole"
                    }
                ]
            }
        }
  },
  
  "Outputs" : {
  "SGID" : {
  "Description" : "The ID of the SG is:",
  "Value" : { "Ref" : "SecurityGroupDemoSvrTraffic" },
    "Export" : {
    "Name" : {
    "Fn::Sub" : "${AWS::StackName}-SgId"
        }
		}
      },
  "EC2ID" : {
  "Description" : "The ID of the EC2 is:",
  "Value" : { "Ref" : "EC2InstanceDemoSvr" },
    "Export" : {
    "Name" : {
    "Fn::Sub" : "${AWS::StackName}-Ec2Id"
        }
		}
      }
  }
}
```

Now, here is the complete template in JSON for creating the Launch Congfiguration along with Auto Scaling Group and Application Load Balancer with its Target Group.

### compute.json
```sh
{
"AWSTemplateFormatVersion" : "2010-09-09",
"Description": "Create an Load Balancer and ASG by AWS CloudFormation",
"Parameters" : {
  "LCName1" : {
    "Type" : "String",
    "Default" : "LaunchConfiguration",
    "Description" : "Enter the name of Launch Configuration"
  },
  "InstanceTypeParameter" : {
    "Type" : "String",
    "Default" : "t2.micro",
    "AllowedValues" : ["t2.micro", "m1.small", "m1.large"],
    "Description" : "Enter t2.micro, m1.small, or m1.large. Default is t2.micro."
  
  },
  "SubnetID" : {
  "Description" : "Please enter the Subnet ID",
  "Type" : "List<AWS::EC2::Subnet::Id>"
  },
  "LoadBalancerN" : {
    "Type" : "String",
    "Default" : "myALB",
    "Description" : "Enter the name of Load Balancer"
  },
  
  "Scheme1" : {
    "Description" : "Enter the type of scheme(internal/internet-facing)?",
    "Type" : "String",
    "Default" : "internet-facing",
	"AllowedValues" : ["internal", "internet-facing"]  
  },
  "TYPE" : {
    "Description" : "Enter the type of load balancer you want?",
    "Type" : "String",
    "Default" : "application",
	"AllowedValues" : ["application", "network","gateway"]   
  },
  "ComputeStackName" : {
  "Description" : "Please enter the EC2 Stack Name",
  "Type" : "String",
  "Default" : "EC2"
  },
  "NetworkStackName" : {
  "Description" : "Please enter the VPC Stack Name",
  "Type" : "String",
  "Default" : "VPC"
  },
  "TGName" : {
    "Type" : "String",
    "Default" : "myTG",
    "Description" : "Enter the name of Target Group"
  }

},
"Resources": {
"IAMRole1": {
            "Type": "AWS::IAM::Role",
            "Properties": {
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Effect": "Allow",
                            "Principal": {
                                "Service": [
                                    "ec2.amazonaws.com"
                                ]
                            },
                            "Action": [
                                "sts:AssumeRole"
                            ]
                        }
                    ]
                },
                "Path": "/",
                "Policies": [
                    {
                        "PolicyName": "AmazonS3FullAccess",
                        "PolicyDocument": {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "*"
        }
    ]
}
                    },
					{
                        "PolicyName": "AmazonRDSFullAccess",
                        "PolicyDocument": {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "rds:*",
                "application-autoscaling:DeleteScalingPolicy",
                "application-autoscaling:DeregisterScalableTarget",
                "application-autoscaling:DescribeScalableTargets",
                "application-autoscaling:DescribeScalingActivities",
                "application-autoscaling:DescribeScalingPolicies",
                "application-autoscaling:PutScalingPolicy",
                "application-autoscaling:RegisterScalableTarget",
                "cloudwatch:DescribeAlarms",
                "cloudwatch:GetMetricStatistics",
                "cloudwatch:PutMetricAlarm",
                "cloudwatch:DeleteAlarms",
                "ec2:DescribeAccountAttributes",
                "ec2:DescribeAvailabilityZones",
                "ec2:DescribeCoipPools",
                "ec2:DescribeInternetGateways",
                "ec2:DescribeLocalGatewayRouteTables",
                "ec2:DescribeLocalGatewayRouteTableVpcAssociations",
                "ec2:DescribeLocalGateways",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSubnets",
                "ec2:DescribeVpcAttribute",
                "ec2:DescribeVpcs",
                "ec2:GetCoipPoolUsage",
                "sns:ListSubscriptions",
                "sns:ListTopics",
                "sns:Publish",
                "logs:DescribeLogStreams",
                "logs:GetLogEvents",
                "outposts:GetOutpostInstanceTypes"
            ],
            "Effect": "Allow",
            "Resource": "*"
        },
        {
            "Action": "pi:*",
            "Effect": "Allow",
            "Resource": "arn:aws:pi:*:*:metrics/rds/*"
        },
        {
            "Action": "iam:CreateServiceLinkedRole",
            "Effect": "Allow",
            "Resource": "*",
            "Condition": {
                "StringLike": {
                    "iam:AWSServiceName": [
                        "rds.amazonaws.com",
                        "rds.application-autoscaling.amazonaws.com"
                    ]
                }
            }
        }
    ]
}
                    }
                ]
            }
        },
        "RootInstanceProfile": {
            "Type": "AWS::IAM::InstanceProfile",
            "Properties": {
                "Path": "/",
                "Roles": [
                    {
                        "Ref": "IAMRole1"
                    }
                ]
            }
        },
    "LaunchConfiguration": {
  "Type" : "AWS::AutoScaling::LaunchConfiguration",
  "Properties" : {
      "AssociatePublicIpAddress" : "True",
	  "LaunchConfigurationName" : {"Ref":"LCName1"},
      "ImageId" : "ami-0be2609ba883822ec",
	  "IamInstanceProfile":{ "Ref":"RootInstanceProfile" },
      "InstanceType" : { "Ref" : "InstanceTypeParameter" },
      "KeyName" : "keypublic",
      "SecurityGroups" : [{
          "Fn::ImportValue" : {
            "Fn::Sub" : "${ComputeStackName}-SgId"

          }
        }]
    }
},

	"myASG": {
      "Type":"AWS::AutoScaling::AutoScalingGroup",
      "Properties": {
        "MinSize":"1",
        "MaxSize":"1",
        "DesiredCapacity":"1",
		"TargetGroupARNs":[{"Ref":"TargetGroup"}],
        "LaunchConfigurationName":{"Ref":"LaunchConfiguration"},
        "VPCZoneIdentifier":{
          "Ref":"SubnetID"
        }
      }
    },
	"SecurityGroupDemoSvrTraffic": {
      "Type": "AWS::EC2::SecurityGroup",
      "Properties": {
        "GroupName": "sgASG",
        "SecurityGroupIngress": [
          {
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIp": "0.0.0.0/0",
            "Description": "For traffic from Internet"
          },
		  {
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIp": "0.0.0.0/0",
            "Description": "For HTTP requests"
          }
        ],
        "GroupDescription": "Security Group for demo server",
		"VpcId" : {
          "Fn::ImportValue" : {
            "Fn::Sub" : "${NetworkStackName}-VpcId"

          }
        }
      }
    },
	"LoadBalancer" : {
  "Type" : "AWS::ElasticLoadBalancingV2::LoadBalancer",
  "Properties" : {
      "IpAddressType" : "ipv4",
      "Name" : {"Ref":"LoadBalancerN"},
      "Scheme" : {"Ref":"Scheme1"},
	  "SecurityGroups" : [
              {
                "Ref": "SecurityGroupDemoSvrTraffic"
              }
            ],
      "Subnets" : {"Ref":"SubnetID"},
      "Type" : {"Ref":"TYPE"}
    }

},
"TargetGroup" : {
  "Type" : "AWS::ElasticLoadBalancingV2::TargetGroup",
  "Properties" : {
      "HealthCheckEnabled" : "True",
      "HealthCheckPath" : "/",
      "HealthCheckPort" : "80",
      "HealthCheckProtocol" : "HTTP",
      "Name" : {"Ref":"TGName"},
      "Port" : "80",
      "Protocol" : "HTTP",
      "Tags" : [
	  {
	  "Key": "Name",
	  "Value":"TargetG"}],


      "VpcId" :{
          "Fn::ImportValue" : {
            "Fn::Sub" : "${NetworkStackName}-VpcId"

          }
        }
    }
}
  }


}
```
