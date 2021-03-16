### Cloud Formation Template for AWS Network Infrastructure

Using the below syntax we will create an entire VPC network Infrastructure containing:
1. VPC
2. 3 public subnets in different availability zones
3. 3 private subnets in different availability zones
4. A public route table with 3 public subnets associated with it.
5. A private route table with 3 private subnets associated with it.
6. An Internet Gateway, attach it to public route table
7. NAT Gateway, attach it to a private route table.

Now, we will discuss about some terms which you will be seeing inside the code:
1. Parameters section can be used to customize your templates. Parameters enable you to input custom values to your template each time you create or update a stack. Parameters can be used as reference in Resources section using the intrinsic function Ref.
2. Resources section is used to declare AWS resources you want to include in the stack, such as an Amazon S3 bucket or an Amazon VPC.
3. Outputs is an optional section where you can declare output values that you can import into other stacks (to create cross-stack references). For example, you can output the S3 bucket name for a stack to make the bucket easier to find.

### Documentation Referred:

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-vpc.html

https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet.html

### vpc.json

```sh
{
"AWSTemplateFormatVersion" : "2010-09-09",
"Description" : "VPC using CFN", 

"Parameters" : {
  "vpcName" : {
    "Type" : "String",
    "Default" : "myVPC",
    "Description" : "Enter the name of VPC"
  }
},



"Resources" : {
   "vpcCFN" : {
   "Type" : "AWS::EC2::VPC",
   "Properties" : {
      "CidrBlock" : "10.0.0.0/16",
      "Tags" : [ {"Key" : "Name", "Value" : "VPC_CFN"} ]
   }
},
   "subnet1" : {
   "Type" : "AWS::EC2::Subnet",
   "Properties" : {
      "VpcId" : { "Ref" : "vpcCFN" },
      "CidrBlock" : "10.0.1.0/24",
      "AvailabilityZone" : "us-east-1a",
      "Tags" : [ { "Key" : "Name", "Value" : "Public1" } ]
   }
},
   "subnet2" : {
   "Type" : "AWS::EC2::Subnet",
   "Properties" : {
      "VpcId" : { "Ref" : "vpcCFN" },
      "CidrBlock" : "10.0.2.0/24",
      "AvailabilityZone" : "us-east-1b",
      "Tags" : [ { "Key" : "Name", "Value" : "Public2" } ]
   }
},
   "subnet3" : {
   "Type" : "AWS::EC2::Subnet",
   "Properties" : {
      "VpcId" : { "Ref" : "vpcCFN" },
      "CidrBlock" : "10.0.3.0/24",
      "AvailabilityZone" : "us-east-1c",
      "Tags" : [ { "Key" : "Name", "Value" : "Public3" } ]
   }
},
   "subnet4" : {
   "Type" : "AWS::EC2::Subnet",
   "Properties" : {
      "VpcId" : { "Ref" : "vpcCFN" },
      "CidrBlock" : "10.0.4.0/24",
      "AvailabilityZone" : "us-east-1a",
      "Tags" : [ { "Key" : "Name", "Value" : "Private1" } ]
   }
},
   "subnet5" : {
   "Type" : "AWS::EC2::Subnet",
   "Properties" : {
      "VpcId" : { "Ref" : "vpcCFN" },
      "CidrBlock" : "10.0.5.0/24",
      "AvailabilityZone" : "us-east-1b",
      "Tags" : [ { "Key" : "Name", "Value" : "Private2" } ]
   }
},
   "subnet6" : {
   "Type" : "AWS::EC2::Subnet",
   "Properties" : {
      "VpcId" : { "Ref" : "vpcCFN" },
      "CidrBlock" : "10.0.6.0/24",
      "AvailabilityZone" : "us-east-1c",
      "Tags" : [ { "Key" : "Name", "Value" : "Private3" } ]
   }
},
   "publicRT" : {
   "Type" : "AWS::EC2::RouteTable",
   "Properties" : {
      "VpcId" : { "Ref" : "vpcCFN" },
      "Tags" : [ { "Key" : "Name", "Value" : "publicRT" } ]
   }
},
   "privateRT" : {
   "Type" : "AWS::EC2::RouteTable",
   "Properties" : {
      "VpcId" : { "Ref" : "vpcCFN" },
      "Tags" : [ { "Key" : "Name", "Value" : "privateRT" } ]
   }
},   "mySubnetRouteTableAssociation1" : {
   "Type" : "AWS::EC2::SubnetRouteTableAssociation",
   "Properties" : {
      "SubnetId" : { "Ref" : "subnet1" },
      "RouteTableId" : { "Ref" : "publicRT" }
   }
},   "mySubnetRouteTableAssociation2" : {
   "Type" : "AWS::EC2::SubnetRouteTableAssociation",
   "Properties" : {
      "SubnetId" : { "Ref" : "subnet2" },
      "RouteTableId" : { "Ref" : "publicRT" }
   }
},   "mySubnetRouteTableAssociation3" : {
   "Type" : "AWS::EC2::SubnetRouteTableAssociation",
   "Properties" : {
      "SubnetId" : { "Ref" : "subnet3" },
      "RouteTableId" : { "Ref" : "publicRT" }
   }
},   "mySubnetRouteTableAssociation4" : {
   "Type" : "AWS::EC2::SubnetRouteTableAssociation",
   "Properties" : {
      "SubnetId" : { "Ref" : "subnet4" },
      "RouteTableId" : { "Ref" : "privateRT" }
   }
},   "mySubnetRouteTableAssociation5" : {
   "Type" : "AWS::EC2::SubnetRouteTableAssociation",
   "Properties" : {
      "SubnetId" : { "Ref" : "subnet5" },
      "RouteTableId" : { "Ref" : "privateRT" }
   }
},   "mySubnetRouteTableAssociation6" : {
   "Type" : "AWS::EC2::SubnetRouteTableAssociation",
   "Properties" : {
      "SubnetId" : { "Ref" : "subnet6" },
      "RouteTableId" : { "Ref" : "privateRT" }
   }
},
      "myInternetGateway" : {
      "Type" : "AWS::EC2::InternetGateway",
      "Properties" : {
        "Tags" : [ {"Key" : "Name", "Value" : "IGW_CFN"}]
      }
   },
   "AttachGateway" : {
   "Type" : "AWS::EC2::VPCGatewayAttachment",
   "Properties" : {
      "VpcId" : { "Ref" : "vpcCFN" },
      "InternetGatewayId" : { "Ref" : "myInternetGateway" }
    }
},
   "myRoute" : {
   "Type" : "AWS::EC2::Route",
   "Properties" : {
      "RouteTableId" : { "Ref" : "publicRT" },
      "DestinationCidrBlock" : "0.0.0.0/0",
      "GatewayId" : { "Ref" : "myInternetGateway" }
   }
},
"NAT" : {
   "Type" : "AWS::EC2::NatGateway",
   "Properties" : {
      "AllocationId" : { "Fn::GetAtt" : ["EIP", "AllocationId"]},
      "SubnetId" : { "Ref" : "subnet3"},
      "Tags" : [ {"Key" : "Name", "Value" : "NAT_CFN" } ]
     }
},
"EIP" : {
   "Type" : "AWS::EC2::EIP",
   "Properties" : {
      "Domain" : "vpcCFN"
   }
},
"Route" : {
   "Type" : "AWS::EC2::Route",
   "Properties" : {
      "RouteTableId" : { "Ref" : "privateRT" },
      "DestinationCidrBlock" : "0.0.0.0/0",
      "NatGatewayId" : { "Ref" : "NAT" }
   }
}

},

"Outputs" : {
  "VPCID" : {
  "Description" : "The ID of the VPC is:",
  "Value" : { "Ref" : "vpcCFN" },
    "Export" : {
    "Name" : {
    "Fn::Sub" : "${AWS::StackName}-VpcId"
        }
		}
      },
  
  "mySubnet" : {
    "Description" : "The ID of the Subnet1",
    "Value" : { "Ref" : "subnet1" },
    "Export" : {
      "Name" : "SubnetId"
    }
  }
}


}
```
