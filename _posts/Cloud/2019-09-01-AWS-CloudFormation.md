---
layout: post
title: Learning CloudFormation
description: First steps with CloudFormation
comments: true
featimg: Cloudformation.png
tags: [Cloudformation, AWS, Cloud]
category: [Cloud]
sidebar_toc:	true
---


# About BGP  are all different but they have one thing in common are all different but they have one thing in common


#### 1:33 Speed / kcfnuser / D

<html>
<p>
  <a class="btn btn-primary" data-toggle="collapse" href="#collapseExample" role="button" aria-expanded="false" aria-controls="collapseExample">
    Link with href
  </a>
  <button class="btn btn-primary" type="button" data-toggle="collapse" data-target="#collapseExample" aria-expanded="false" aria-controls="collapseExample">
    Button with data-target
  </button>
</p>
<div class="collapse" id="collapseExample">
  <div class="card card-body">
    Anim pariatur cliche reprehenderit, enim eiusmod high life accusamus terry richardson ad squid. Nihil anim keffiyeh helvetica, craft beer labore wes anderson cred nesciunt sapiente ea proident.
  </div>
</div>
</html>



<button data-toggle="collapse" data-target="#demo">Collapsible</button>

<div id="demo" class="collapse">
Lorem ipsum dolor text....
</div>

> **Good link for CFN  IDE Tips**
> https://hodgkins.io/up-your-cloudformation-game-with-vscode

A wordpress stack typically consists of the following:


![](/assets/markdown-img-paste-20190730204655360.png)

### JSON
- JSON stands for "JavaScript Object Notation"
- Used heavility within the AWS Product set
- JSON is constructd of name/value pair
- Data is seperated by commas
- Value can be strin,number,boolean,object,array or null
- `[]` Square brackets holds arrays
- Cury Braces {} Holds JSON

`Null` in its purest form is not used in CloudFormation.

> In the example below notice that the `ProductID` variable is used twice , which is fine since it is under two different JSON objects (Curly Brackets)
**Same variable cannot be used at the same JSON depth**

 ```json

{
"OrderID": 12345,
"ShopperName" : "Reyansh",
"Email": "email@gmail.com",
"OrderData" : [
    {
        "ProductID": "12",
        "Quantity": "1"
    },
    {
        "ProductID":"34",
        "Quantity": 2

    }
]
}

 ```

Below is an example of a YAML CloudFormation Template

> Notice that under `Listeners` we have 2 `-` ; which means  start of two different lists

```yaml
myELBname:
  Type: AWS::ElasticLoadBalancing::LoadBalancer
  Properties:
    Scheme: internal
    SecurityGroups:
      - sg-id
    Subnets:
      - subnet-id
    Instances:
      - instance-id
    Listeners:
      - LoadBalancerPort:
        InstancePort:
        Protocol: HTTP|HTTPS|TCP|SSL
        InstanceProtocol: HTTP|HTTPS|TCP|SSL
      - LoadBalancerPort: '443'
        InstancePort: '443'
        Protocol: HTTPS
        InstanceProtocol: HTTPS
        SSLCertificateId:
```

**Lets start our first cloudformation template with the following :**

```yaml
Resources:
  DB:
    Type: "AWS::RDS::DBInstance"
    Properties:
      AllocatedStorage: 2
      StorageType: gp2
      DBInstanceClass: db.t2.micro
      DBName: wordpress
      Engine: MySQL
      MasterUsername: root
      MasterUserPassword: redhat123
  EC2:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-343243423434
      InstanceType: t2.micro
  S3:
    Type: "AWS::S3::Bucket"
    Properties:
      BucketName: wp-vikassri12312312313
```

Thats a good start up template but lets look at the challenges in `re-use` of the template .

> - Everytime this template is used , `DBInstanceClass` and `InstanceType` has hardcoded values , which as per the customer requirement should be Small ,Medium , Large ; which cin this case is static.
> - The instance type is specific to a region and the CloudFormation template is not portable across regions.
> - Also the S3 bucket is static , so the name would conflict.

![](/assets/markdown-img-paste-20190731194304617.png)

The way we fix the above issues is with the following script :

```yaml
Parameters:
  EnvironmentSize:
    Type: String
    Default: SMALL
    AllowedValues:
      - SMALL
      - MEDIUM
      - LARGE
    Description: Select Environment Size (S,M,L)
Mappings:
  RegionMap:
    us-east-1:
      "AMALINUX" : "ami-c481fad3"
    us-east-2:
      "AMALINUX" : "ami-71ca9114"
    us-west-1:
      "AMALINUX" : "ami-de347abe"
    us-west-2:
      "AMALINUX" : "ami-b04e92d0"
  InstanceSize:
    SMALL:
      "EC2" : "t2.micro"
      "DB" : "db.t2.micro"
    MEDIUM:
      "EC2" : "t2.small"
      "DB" : "db.t2.small"
    LARGE:
      "EC2" : "t2.medium"
      "DB" : "db.t2.medium"
Resources:
  DB:
    Type: "AWS::RDS::DBInstance"
    Properties:
      AllocatedStorage: 5
      StorageType: gp2
      DBInstanceClass: !FindInMap [InstanceSize, !Ref EnvironmentSize, DB]
      DBName: wordpress
      Engine: MySQL
      MasterUsername: wordpress
      MasterUserPassword: w0rdpr355
  EC2:
    Type: "AWS::EC2::Instance"
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMALINUX]
      InstanceType: !FindInMap [InstanceSize, !Ref EnvironmentSize, EC2]
  S3:
    Type: "AWS::S3::Bucket"
```

![](/assets/markdown-img-paste-20190731200352803.png)

- **In the above Code Template we used `Mapping` tables to reuse variables and make the template more dynamic**
- **The `!FindInMap` references the Table and looks for the value there in the table**

----

In the code example below , take a look at the following :

- `Fn::Base64"` : Allows to put in `cloud-init` data
- `Sub` : Function for string substitution
- `$Variables` : Used below in the `sed` line to make the `cloud-init` dynamic.

> `Keypair below "AdvancedCFN" is used to reference an already created SSH key.`

```yaml
  EC2:
    Type: "AWS::EC2::Instance"
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMALINUX] # Dynamic mapping + Pseudo Parameter
      InstanceType: !FindInMap [InstanceSize, !Ref EnvironmentSize, EC2]
      KeyName: AdvancedCFN
      UserData:
        "Fn::Base64":
          !Sub |
            #!/bin/bash
            yum install httpd php mysql php-mysql -y
            yum update -y
            chkconfig httpd on
            service httpd start
            cd /var/www/html
            wget https://wordpress.org/latest.tar.gz
            tar -zxvf latest.tar.gz --strip 1
            rm latest.tar.gz
            cp wp-config-sample.php wp-config.php
            sed -i 's/database_name_here/${DatabaseName}/g' wp-config.php
            sed -i 's/localhost/${DB.Endpoint.Address}/g' wp-config.php
            sed -i 's/username_here/${DatabaseUser}/g' wp-config.php
            sed -i 's/password_here/${DatabasePassword}/g' wp-config.php
```

***What the above means in simple terms***

![](/assets/markdown-img-paste-20190731204841465.png)

> **Note that in above example we have the `DB.Endpoint.Address` is required to be created `before` the `EC2` instance can be created. Hence CloudFormation creates the DB first and then the EC2**

> **The above is done automatically by the AWS systems**


As a best practice , we should declare the dependency manually using the `DependsOn` attribute. Notice in the example below under EC2 , we say `DependsOn: DB` which basically means that EC2 would have to wait for the DB to done first before the EC2 is created.

```yaml
Resources:
  DB:
    Type: "AWS::RDS::DBInstance"
    Properties:
      AllocatedStorage: 5
      StorageType: gp2
      DBName: wordpress
  EC2:
    Type: "AWS::EC2::Instance"
    Properties:
      DependsOn: DB
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMALINUX]
      InstanceType: !FindInMap [InstanceSize, !Ref EnvironmentSize, EC2]
  ```

### CloudFormationInit

**Benefits**
- `cfn-init` is OS independent
- `cfn-init` is a desired state engine. Like Ansible you define that you need `http` server ; you dont have to install , start the service etc . !!


Example CloudFormation Init Template

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Metadata:
      AWS::CloudFormation::Init:
        config:
          packages:
            :
          groups:
            :
          users:
            :
          sources:
            :
          files:
            :
          commands:
            :
          services:
```

- The `Metadata` tag is at the same level as the Instance Type tag
- The CloudFormation::Init starts by default with `config` if not `configsets` are defined
- `packages` : Define the packages to be installed
- `groups` : user groups be present
- `users` : users to be present
- `sources` : Ensures to allow certain files stored on the instance
- `file` : Create , Modify , delete and control file content.
- `commands` : Allows you to run commands .
- `services` : Enables service declarations. service restart stop etc.


```yaml
  EC2:
    Type: "AWS::EC2::Instance"
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMALINUX] # Dynamic mapping + Pseudo Parameter
      InstanceType: !FindInMap [InstanceSize, !Ref EnvironmentSize, EC2]
      KeyName: AdvancedCFN
      UserData:
        "Fn::Base64":
          !Sub |
            #!/bin/bash
            yum update -y aws-cfn-bootstrap # good practice - always do this.
            /opt/aws/bin/cfn-init -v --stack ${AWS::StackName} --resource EC2 --configsets wordpress --region ${AWS::Region}
            yum -y update
```


CFN Hup is a sister utility for CFN Init. It listens for a change on the meta data for the instance. When the metadat changes , CFN Hup can update the instances based on the updated Metadata.
