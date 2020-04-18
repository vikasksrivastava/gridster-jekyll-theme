---
layout: post
title: AWS Certified Solution Architect Associate Notes
description: My Notes on AWS CSA
comments: true
sidebar_toc:	true
---

# Introduction

---
### AWS Well Architected Framework

The well oriented architecture consist of the following **`5`** Pillars ;  by following a process of **evaluating your architecture against each pillar**.

Each of these pillars are **<span style="color:blue"> questions you ask yourself</span>**

- **`Operation Excellence`** : How will you implement monitoring and logging and redesign your architecture based on the information from the monitoring system.
- **`Reliability`** : Recover from Failure and mitigate disruptions.
- **`Security`** : Protect and Monitor systems.
- **`Performance Efficiency`** : Use computing resource efficiently
- **`Cost Optimization`** : Avoid or Limit expense.

---

### AWS Shared Security Model

AWS is responsible for the security of the global infrastructure and foundational services. The responsibility for security is shared between customer and AWS.

**AWS Responsibilities**
- `Facilities`
- `Physical security of Hardware`
- `Storage Decomissioning`
- `Netwrok Infrastrcutre`
- `Virtualisation Infrastructure`

**Customer Responsibilities**
- `Amazon Machine Images`
- `Operating Systems`
- `Applications`
- `Secuirty Groups`
- `Firewalls`
- `Data in Transit`
- `Data at rest`
- `Credentials`
- `Intrusion Detection and Intrusion Prevention`


---

### AWS Physical Organisation


At a very high level AWS can be broken down into two main blocks

- **`AWS Regions`** : Are grouping of independently separated datacenters (`Availabilty Zones`) in a specific geographic regions ,

- **`AWS Edge Location`** : It is a Datacenter which does not contain any AWS Services ; Instead it is used to deliver contents to parts of the world. `CloudFront`


**<span style="color:blue"> Important Points </span>**

![](/assets/markdown-img-paste-20190725071537652.png)

> - Not All AWS Services are availaible globally , one of the example of a Global Service is the `IAM`
> - AWS Services are region specific and when they are first rolled out , they are rolled out in a region first
>- An `AWS Region` has multiple `Availabilty Zones`
> - Now within each `Availabilty Zone` there can be multiple `Datacenters`
> - `Availabilty Zones` are **phyically sperated** to each other but **have high speed connection between them** to provide **Fault Tolerance**

> So as an example `S3` is replicated accross all `Availabilty Zones` and all `Datacenters` for **reliability and high availaibility**.



**AWS has 19 Regions and 57 AZs within those 19 regions.**
![](/assets/markdown-img-paste-20190725061715368.png)

-------

### AWS Region

![](/assets/markdown-img-paste-20190725072213950.png)
>**AWS re:Invent 2014: AWS Innovation at Scale with James Hamilton**
>https://www.youtube.com/watch?v=JIQETrFC_SQ


Let look into one of the regions: <span style="color:blue">**`us-east`**</span>
- Each region has at least **2** `availaibility zones`.
- Instead of a `DC` being a region, AWS has a collection of `AZ` as a **region**. There are **two transit centers** which connect to customers.
- The AZs are quite apart (few miles). The speed between AZs are **extremely high**. **25Tbps**.

> **Transit points allow conectivity with other `regions` and customer peering points.**

---

![](/assets/markdown-img-paste-20190725073401480.png)
**The Problem :** <span style="color:red">The picture above illustrates the limitation/problems with DCs that are separated too far (LA and NY)</span>

**Solution :**
- In an AZ you can commit synchronously to your Data with a 1ms delay rather than 74ms delay (above) . This is empowering and great for the applications.
- Failover between AZs is **SUPERFAST** , compare this to a Intra Region failover which would be much slower.

---

### Deeper view of 'Availaibility Zone' and 'Data Center'

**Going further**, lets look into **<span style="color:blue">Availability Zones</span>**

![](/assets/markdown-img-paste-20190725090416853.png)

**Double clicking** into **<span style="color:blue">Data Center</span>**

![](/assets/markdown-img-paste-20190725090810986.png)
![](/assets/markdown-img-paste-20190725090954823.png)

**Another view of details shared above**

|   |   |
|---|---|
| ![](/assets/markdown-img-paste-20190725094118435.png)  |  ![](/assets/markdown-img-paste-20190725094202242.png)  |


---

## AWS Indentity and Access Management

Now lets look at the `IAM` Components where you manager Users, roles and groups, IAM Consistes of :

- `IAM User`
- `IAM Groups`
- `IAM Policy`
- `IAM Role `
- `IAM API Keys`
- `IAM Password Policy`

> **IAM is Global and does not require a region selection.**

Now once you have enable IAM , you can use the link show in the picture below to login


![](/assets/markdown-img-paste-2019111907144369.png)

### IAM Policies

Here's where the IAM Policies exist in AWS

![](/assets/markdown-img-paste-20191119071607205.png)

Clicking on that policy further gives you information on the **JSON properties**

![](/assets/markdown-img-paste-20191119071843610.png)

An IAM Policy, looks like this. Note that `Deny` will have a precedence over `Allow`.


<img src="/assets/markdown-img-paste-20180317163328797.png" alt="Drawing" style="width: 500px;"/>

**IAM Policies can be created into Visual Editor or JSON Editor**

![](/assets/markdown-img-paste-20191119072236119.png)

<iframe width="700" height="500"
src="https://www.youtube.com/embed/9a0IQCXguCY"
frameborder="0"
allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
allowfullscreen></iframe>

<br>

> `ARN` stands for : **Amazon Resource Name**

### IAM groups

Groups are like AD Groups are also there  in a company where you segregate users into respective division , like HR, Finace etc .

So now the `policies` can be attached to the `groups` directy.

### IAM Roles

**IAM Roles are an Important Concept to understand**

> An AWS IAM Role is like the 'sudo' in Linux. A user can assume the role of root and do the required tasks without knowing the actual root password.

<iframe width="700" height="500"
src="https://www.youtube.com/embed/Z4jogVG30-w"
frameborder="0"
allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
allowfullscreen></iframe>

A real example is a AWS Instance trying to access S3

- Look at the example below and notice how that all the Users and Roles are created at the IAM Level.

  - `Non AWS Account Holder` : Now what we integrate AWS with the AD in a company. The users from AD who are **authenticated via SAML** now need to be mapped to `Role` to be able to do things in the AWS

  - `EC2 instance needing access to S3`

> An EC2 instance can only have one role attached to it .

> Credentials configure using the `aws-configure` command are store as cleartext int he `.aws` dir under the file `credentials`
> which is `.aws/credentials`



#### Role Assumption

Now lets say , the users below in the `DEV` group need access to the  resources in the `PROD`, then can **assume** a `Role` and be able to access the resource in `PROD`.

**How does the above assumption happen?**

An `IAM Policy` is attached to an `IAM Role` which is then assigned to the `User` or `Resource` which needs acces.

<img src="/assets/markdown-img-paste-20180317165340821.png" alt="Drawing" style="width: 300px;"/>

Notice in the screenshot below , we have 3 different types of Roles:

- **`AWS Service Roles`** : Like EC2 accessing S3
- **`Role for Cross Account access`** : Allowing access for `DEV` IAM group to `PROD`
- **`Role for Identity Provider Access`** : Federation and SAML , Facebook, Google.



<img src="/assets/markdown-img-paste-20180317165948959.png" alt="Drawing" style="width: 600px;"/>

---

### IAM API Keys

When a user is created , he can have programmatic access to his environment using the Secret Access Key

![](/assets/markdown-img-paste-20191122052631862.png)

**BUT** hardcoding this API  access key into git is also not a good idea, so it better to leverage roles (which we discussed above 'IAM Roles')

**Now** `IAM Roles` also in the backend use "Security Access Keys" **BUT** they are short lived. This temporary set of access keys is provided by **Security Token Service**.

#### Security Token Service

As mentioned above this a temporary credetials which allows access to the AWS Resource.
So in the example of `EC2` accessing `S3` and `STS` is generated for the EC2 instance.

The benefit is that the **credetials are not required to be embedded** in the application.
STS times out / expires after certain time period.
When an STS call is made , a credential object is returned conatining :

- **`Session Token`**
- **`An Access Key ID`**
- **`A Secret Access Key`**
- **`Expiration Timestamp`**

Here are the type of STS API Calls which can be made :

- **`AssumeRole`** : When you are using a Role you can use an Assume Role to choose a Role you want to assume. You permissions are replaced to the permissions provided by the new role.
- **`AssumeRoleWithWebIdentity`**  Facebook/Google/Amazaon , essentials you allow users to login using Web Identity and then allow them to make API calls using the same.
- **`AssumeRoleWithSAML`** This is for integration with SAML providers like AD. Helps achiveving SSO (Single Sign On)
- **`GetFederationToken`** *"Give me a set of credetials and I only want to have these permissions."*
- **`GetSessionToken`** :  A person who would normally use his permanent credentials but is logging in from an untrusted device , hence can request safer temporary credentials then permanent)

The major difference between the **Assume** roles and **GetRoles**

![](/assets/markdown-img-paste-20191122061036280.png)

> `https://mermaidjs.github.io/mermaid-live-editor/`

---

### IAM Identity Federation

There are three primary ways of Identity Federation :

- **`Custom Indentity Provider`** : Your inhouse Identity Solution
- **`LDAP / Active Directory`** : Standards based identity solution (SAML)
- **`Web Indentity`**

#### Custom Indentity Provider

![](/assets/markdown-img-paste-20191122063904308.png)
In a custom indentity provider :
1. You client connect to the `Identity Broker`
2. Teh Identity broker confirm the credentials from the Identity Store
3. Once step 2 is successsfull , Indentity Broker now reaches out to `STS` for a token.
4. The token is then provided to the Client
5. Now the client can access AWS Console/Services

#### LDAP / Active Directory (SAML) for SSO

> `SAML : Security Assertion Markup Language`

> **2018 AWS Federated Authentication with Active Directory Federation Services (AD FS)** https://aws.amazon.com/blogs/security/aws-federated-authentication-with-active-directory-federation-services-ad-fs/
> **2013 Enabling Federation to AWS Using Windows Active Directory, ADFS, and SAML 2.0** https://aws.amazon.com/blogs/security/enabling-federation-to-aws-using-windows-active-directory-adfs-and-saml-2-0/

![](/assets/markdown-img-paste-20191122064402130.png)
In a SAML/LDAP Login :
1. User Logs into a AD Webpage
2. Authentication is done and a SAML Asser to the user browser is done
3. User is then redirected to AWS Sign in which validates the SAML and responds back with login to the AWS Console.
4. The permissions in the console are based on the mapping of groups in AWS and AD.

<iframe width="700" height="500"
src="https://www.youtube.com/embed/VP9wBGm982g"
frameborder="0"
allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
allowfullscreen></iframe>


#### Web Indentity Provider (OpenID like Google,FB)


![](/assets/markdown-img-paste-20191122064956631.png)

In a Web Indentity Login :


---

> https://www.youtube.com/watch?v=SIBCi76wspQ&t=1947s
> https://www.youtube.com/watch?v=bXrsUEI1V38

---

# AWS Organisations

AWS Organisations is a way to **consolidate multiple individual AWS accounts** **into one Organisation.**

So for example , **you can group multiple discrete accounts** like the following into one.

<img src="/assets/markdown-img-paste-20191127054106107.png" alt="Drawing" style="width: 200px;"/>

<br><br>

**Challenges** (without AWS Ogranisations)

  - IAM policy replication across AWS accounts requires custom automation.
  - Creating new accounts involved many steps including the requirements to validate the payment method. **The account needed to be created manually in AWS UI**
  - It would bery difficult to do security governance at individual account levels.

**Benefits** (with AWS Organisations)

  - Grouping accouts gives you consolidated billing. (**Getting billing consolidated gets you better discount.**)
  - **Single place to manage your accounts without**the need to validate billing for each account.
  - **Automate account creation** : Instead of the need to go to AWS UI everytime , AWS Organisation provides API calls for account creation.
  - **Single place to manage security policies** from one place (IAM could only have managed security policies in one account)


### How are Accounts and OUs Organised under `AWS Organisations`

<img src="/assets/markdown-img-paste-2019112705591646.png" alt="Drawing" style="width: 600px;"/>
<br>

<br>

- One OU cannot be under two OUs
- One AWS Account only belongs to One OU , it cannot belong to multple OU
- The tree can only go 5 levels deeps.
- Security Policy can be applied to OU or the Account.

> **By default the level of depth allowed on AWS Organisation is low** a service case needs to be opened to fix it.

**Best Practices:**

- CloudTrail (AccessLogs) in master account and esnure the SCP does not let the same to be deleted at individiual account level.
- Do not create resources (`EC2` etc) in Master Account, Use it for Billign , Manageent , Policies

## Service Control Policy

Service control policy is the policy feature used in AWS Organisation. It can be applied at the Organisation or OU level.

SCP is like a boundary to multiple accounts.
**A Good Example** : Lets say you have multiple accounts , and CloudTrail (logging) is enabled. Now the root user can go ahead and delete the logs. In this case if the policy is set at the AWS Org level **DENIES* access to be taken place.
**Even the root user cant do it**

**A service policy is applied to level its applied to and trickles down to everything below it.**


**How does the Permissions Apply when two SCPs intersect**

<img src="/assets/markdown-img-paste-20191127071224350.png" alt="Drawing" style="width: 400px;"/>

### Strategies for using SCPs

You can configure the SCPs in your organization to work as either of the following:

A **`deny list`** – actions are allowed by default, and you specify what services and actions are prohibited.

An **`allow list`** – actions are prohibited by default, and you specify what services and actions are allowed

**Decision Process**

![](/assets/markdown-img-paste-2019112707292648.png)


**`Service Control Policy`  in conjuction with  `IAM Policy`**

<img src="/assets/markdown-img-paste-20191127074422917.png" alt="Drawing" style="width: 200px;"/>

<br>

> Service Link Roles "Roles Used by services" to operate on other services are not affected by **Service Control Polices**


**SCP Configuration Example**

https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_example-scps.html#example_scp_1

![](/assets/markdown-img-paste-20191127072249878.png)

Reference

> https://www.youtube.com/watch?v=-HsfTwdRxRI
> https://www.youtube.com/watch?v=_50P0o14Ul0

**Lab Tasks/Video**

- Create Test Accounts
- Show no need for payment validation
- An Organisational Account is the Collection of Accounts

- An OU Will Inherit All POlicies from ALL OU Above
- Show SCP Attach / Detach options.

- Create A service control policy for  VPC Restriction .
    - Create
    - Attach

---

# Make A Template of a good section with Challenges , benefits , CloudFormation , Reference

# AWS Single Sign On

# AWS Tower

# AWS Active Directory

---




> https://daviseford.com/blog/2018/12/21/aws-advanced-networking-specialty-exam-tips.html

Now as we further move ahead , the `VPC` :


>  - **The VPC is a Logical Construct with little significance to actual networking.**
>  - **A Large IP Space is define while creating a VPC , but eventually smaller subnets are to be created specific to the AZs.**
>  - **A subnet spans only the AZ**
> - **When an Instance is created with a Subnet Specified -- The instance gets created in the same AZ as the subnet (Obviously :))**
> - **Dual Interfaces can reside in a single subnet only** (As per the lab)


> **Security Groups are State Full , once you allow a traffic inbount , there is no need to create an outbout rule for the same.**
> **NACLs are not statefull , both INBOUND and OUTBOUND rules need to be created**




<img src="/assets/markdown-img-paste-20180317161347849.png" alt="Drawing" style="width: 300px;"/>

### AWS Terminology

- `High Availability (HA)` This means creating and managing the ability to automatically “failover” to a secondary system if the primary system goes down for any reason as well as **eliminating all single points of failure** from your infrastructure.

- `Fault Tolerance` this means infrastructure that is designed in such a way that **when one component fails**(be it hardware or software), a **backup component takes over operations immediately** so that there is no loss of service.

- `Scalability` is the capability to scale out **easily**.

- `Elasticity` the ability to **scale-up** as well as **scale-down**


## AWS Networking

> VPC `Virtual Private Cloud` is how the networking is architected in the cloud.

- Now lets take a deeper dive into VPC ad look it under the hood.

  `VPC` spans `Avalaibility Zones` and `Multiple Datacenters`.

  <img src="/assets/markdown-img-paste-20180317171726859.png" alt="Drawing" style="width: 300px;"/>

  Now take a look at the picture belpw and see what the a dissection of VPC looks like internally.

   - Notice how the VPC spans the two different AZs in the picture.
   - Notice the **private subnets** e.g `172.16.0.0` spanning two different AZs. This enables you to design and put your servers in **different failure domains enabling High Avalaibility**
   - You can define custom `CIDR` in each subnet.
   - You can create routes between the Subnets.
   - You can also have subnet level firewalls and access rules.

  <img src="/assets/markdown-img-paste-20180317172222633.png" alt="Drawing" style="width: 300px;"/>


### Typical VPC screen

- Notice on the left the different networking constructs you have to work and play with :)

  <img src="/assets/markdown-img-paste-20180317174000405.png" alt="Drawing" style="width: 200px;"/>

### VPC Routing Basics

- Now lets take a look at a deeper level on the VPC Routing Basics the `Internet  gateway`, `Subnets` and the `Routing table`.

  - `Internet Gateway` : The is a default internet gateway connected to the default VPC. It is horizontally scales in the backend and there is no need to manage the bandwidth and capacity of the same. It has **built in Fault tolernace at the Amazon Networking level**.

    > We dont have to manage `Internet Gateways` bandwidth, AWS Does that for us.

    Again , you can only have **one Internet Gatewat attached a VPC**.

    Notice the attached `Internet Gateway` to the VPC in the picture below.

    <img src="/assets/markdown-img-paste-20180317174555203.png" alt="Drawing" style="width: 300px;"/>

    Now take a look at the `Route Table` in the picture below and how `0.0.0.0` is mapped to the `Internet Gateway` for all **outbound internet traffic**.

    <img src="/assets/markdown-img-paste-20180317174853412.png" alt="Drawing" style="width: 300px;"/>

  - `Subnets`: After creating VPC, you can create o**ne or more subnets** in each `Availabilty Zone`. Each **subet much reside in one Availaibilty Zone** and **cannot span** different `Availability Zone`.

    > So **VPC spans Availabilty Zones** but **Subnet are contained in one Availabilty Zone**.

    IMPORTANT NOTE : `Subnets` are associated to `Route Tables` and the `Route Table` may or may not be attached to an `Internet Gateway`

  The following association is show in the picture below .

  `Subnet ---> Route Table ---> Internet Gateway`

  <img src="/assets/markdown-img-paste-20180318151727852.png" alt="Drawing" style="width: 300px;"/>

  `Explicit Association` is when you move the route tables from default to the one which you created.

  **High Level AWS Networking Internal Architecture**

  <img src="/assets/markdown-img-paste-20180318212412969.png" alt="Drawing" style="width: 300px;"/>


### VPC Security

- **`Network ACLs`** `allow` or `deny` traffic at the subnet level and are stateless, which means that the return traffic had to be `ALLOWED` for traffic for both direction.

  This follows at the Cisco ACL `access-list` order charecteristics.


- **`Security Groups`** Just like Openstack Security groups but they are **statefull** . Security group **only supports** `allow` rules.

### VPC Workflow

1. Create the `VPC` and give it a `CIDR` Block range. **Notice** that you can also define `IPv6` CIDR ranges. When `Tenancy`  is set to dedicate you are **not sharing** your servers with the other users.
<img src="/assets/markdown-img-paste-20180318160426965.png" alt="Drawing" style="width: 300px;"/>
2. Create the `Internet Gateway`
<img src="/assets/markdown-img-paste-20180318160902338.png" alt="Drawing" style="width: 300px;"/>
3. Attach the `Internet Gateway` to the `VPC`
<img src="/assets/markdown-img-paste-20180318161028276.png" alt="Drawing" style="width: 300px;"/>
4. Next we will create **two different** `Route Tables`, one which is **connected** to the `Internet Gateway` and the other **which is not** connected.
<img src="/assets/markdown-img-paste-20180318161246639.png" alt="Drawing" style="width: 300px;"/>
Now **connect** the `Route Table` created above to the `Internet Gateway`. If you leave the `Route Table` not connected to the `Internet Gateway` it becomes a private route table.
<img src="/assets/markdown-img-paste-20180318161517281.png" alt="Drawing" style="width: 300px;"/>
5. Now Create a `Subnet`. Notice that you have to specifically define the `Availabilty Zone` (a subnet can only span an AZ) and the `CIDR` block range which is within the range of the VPC.
<img src="/assets/markdown-img-paste-20180318162016693.png" alt="Drawing" style="width: 300px;"/>
6. Now you can create the Network ACL which is a straight forward process.
<img src="/assets/markdown-img-paste-20180318162441801.png" alt="Drawing" style="width: 300px;"/>

### VPC Limits

- 5 VPCs per region (you can have this increased)
- 5 Internet Gateways per region , this is equal the VPC limit as **one VPC can only have one internet gateway**.
- 50 VPN Connections per region.



-------

## Server-Based Compute Services (EC2 Elastic Cloud Compute)

### EC2 Limits

- Take a look at the default EC2 Limits :

  <img src="/assets/markdown-img-paste-20180318200924374.png" alt="Drawing" style="width: 300px;"/>

### EC2 Purchasing Options



**There are 3 different types of EC2 Instances :**

- **`Ondemand`** Most expensive , run and destroy whenever you like. You care billed as per per `second` or per `hour`. **These differ based on Images** too , not all images support per second basis. A`mazon Linux` or `Ubuntu` offer per second pricing.

  > Note that if the image is stopped  you are not charged for the same.

- **`Reserved`** Allows you purchase instance for `1` or `3` years and you get **discount** becuause of the commitment. But here you pay the whole amount regardless of how much you use it. You can pay upfront partial partial and no upfornt

- **`Spot`** In this case prices of instances fluctuate based on what AWS considers is **unused capacity in their datacenters**. In this case you the billing happens like the `Ondemand` instance.

  And instance is provisioned for you **when your bid price is less than or equal to the instance price**. If the **AWS's price increases your instance is terminated**.

  If the instance is **terminated by AWS** you **dont pay for that slot** of duration (1 minute or 1 hour) . If **you delete it**, **you pay** for it.


### AMI's

AMI are the images/files that goes to the boot volume of the instance.

### Linux AMI Virtualisation Type :

- **`HVM`**  **Hardware Virtual Machine** , Conside this like Intel VTx/d technology by which guests can take advantage of virtualisation feature supported by the CPU. Proved enhanced networking and GPU processing to the VM.

- **`PV`**  **Paravirtual** AMIs Guests could run on Hardware which does not have HVM support. cannot take advantage of GPU / Advanced networking.

HVM is the recommended AMI type.

> `ENA Enabled` : Means enhanced networking capabilities based provided by hardware extensions (HVM)

### EC2 Instance Type :

Has the following virtual hardware components, you have to choose the right instance type **based on your application requirement** :

- `vCPU`
- `RAM`
- `Storage Options `
- `Network Performance`

  Notice in the picture below the above characteristics.
  <img src="/assets/markdown-img-paste-20180318203911238.png" alt="Drawing" style="width: 600px;"/>

  **Different Instance types**

  <img src="assets/markdown-img-paste-20191201073012960.png" alt="Drawing" style="width: 600px;"/>

  An instance type is given a `Letter` and a `Number` so example `T` and `2` which becomes `T2`

  Further T2 comes into a wide range for flavors - `t2.small` to `t2.xlarge`

  All instance types are documented here : https://aws.amazon.com/ec2/instance-types/

  **T2 Burstable** : When you are not using CPU , you are building credits and you use these credits when you use the CPU. So its a credits based system.

  **T2 Unlimited** Its an option when launching instance where you can request AWS to not bring your instance down to the lower baseline CPU performance and let it continue to use standard CPU utilisation. This has additional charge.




### EC2 Instance Metadata

- `http://169.254.169.254/lates/user-data` Shows the **user data provided for bootstrapping**.

- `http://169.254.169.254/lates/user-data` Displays **AMI, Instance Type** etc.

### EC2 Instance Storage types



![](assets/markdown-img-paste-20191201082628711.png)

**`Instance Store`** volumes are ephemeral data. The data on the volume only exists till the life of the instance. If the instance is stopped or shutdown the data will be lost.
If the instance is rebooted the data is retained.

> Instance store is good for a `swap` type usage

**`EBS Elastic Block Store`**

 - EBS Volumes are persistent and can stay even after instance deletion, unlike Instance Store.
 - Can **only be attached to one instance** a time.
 - EBC can be backed up into snapshots. These snapshot can later be restored to EBS volume.
 - EBS Volume measures performance in IOPS , AWS one IOPS is 256KB chunk (input or output)
 - Even with provision IOPs you may not get the performance you want, you may want to choose an instance type which is EBS optimized instance type.
 - New EBS volume no longer need to be pre-warmed. Maximum performance is received as soon as the EBS volume is spawned.
 - Now with the above point note that if you created a volume from a snapshot you still need to warm-up  (read all the storage blocks)/ read otherwise the access could be slow.

> In an EBS Volume there are two separate hardware devices per EBS volume , so if there is a failure in one disk , while it rebuilds the data is still accessible from the other device. **EBS Volumes can be backed up in S3**


### Different EBS types

- `General Purpose` (1GiB - 16TiB), best used for Dev and Test environment. This allows 3 IOPS/GiB (so size of the disk has direct relation with the IOPS) but is burstable upto 3000 IOPS (Remember burst is a credit based system , which means you ear credits for not using it and then you can use those credits). Max performance of 32000 IOPS per volume.
- `Provisioned IOPS SSD` (4GiB - 16TiB) , Used for mission critical applications. This performs at the provisioned IOPS level and can go upto 64,000 IOPS.


### EBS snapshot

- You can take a snapshot to recreate the volume. Snapshots are stored in S3. You are only charged for the difference between the snapshots.
- When a `restore` is done from this snapshot , the data is made availaible as they are read , which makes it slow. So you can make an script to access the entire disk so that all data is accessible.

## Lab

 - Instance Creation Flow Chart
 - EBS Snapshot --> Show in S3 if its exists
 - Restore Snapshot
 - Creatign Custom AMIs using the Snapshot


  <img src="/assets/JB392svLUx.gif" alt="Zoom In" style="width:2000px;"/>


  <img src="http://g.recordit.co/nMsgI72s6p.gif" alt="Zoom In"/>


### Placement groups

> Instances must have **10G connection** type to be able to use Placement Groups.

 - When there is a requirement to keep the instances as close possible to other instances a placement group is used. Like `Affinity Groups` in Openstack.

    Instances not created in a Placement group cannot be moved into a placement group.
    **A placement group cannot span AZs**.

### EFS Elastic File System

- **Consider this like NFS** , this can be shared between multiple EC2 instances.


-------

## Server**less** Compute Services (AWS Lambda)

- You do not need EC2 servers (Serverless) to run your application , you can directly run your programs here !

  A picture says a thousand words :

  <img src="/assets/markdown-img-paste-20180318214149491.png" alt="Drawing" style="width: 500px;"/>



  ### When should you run Lambda
  > You are only billed for the duration your code is running.

  Generally you will run Lambda when you want a function to run in response to an event.

  Example Lambda Use :
`Upload Object to S3 --> S3 Event --> Triggers a Lambda Function`

## Quick Detour

### CloudWatch (Monitoring)

- Used to monitor AWS Services such as EC2
- Provides centralised monitoring and performance mentrics.
- CloudWatch Alarms can be used a triggers.

### CloudTrail (API Logging Service)

- Logs all API calls.

### SNS Notification Service

- Automates the process of sending notification email or text . SNS is integrated in many AWS services so it is easy to use it.

-------

## Advanced Networking

###

If you notice in the picture below , we have now added a load balancer and an autoscaling group which now spans two vPCs .

<img src="/assets/markdown-img-paste-20180322075845612.png" alt="Drawing" style="width: 400px;"/>

**`Elastic Loadbalancer`** : EC2 Service which loadbalances traffic to multiple EC2 instances across multiple `Availabilty Zones`.

- Also `Elastic Loadbalancer` should also be paired with `Auto Scaling Group` for High Availabilty.

- There could also be an Internal Elastic Loadbalancer for loadbalancing traffic internally within private subnets.
- Elastic Loadbalancer can also stop sending traffic to a non responding instance based on `Health Checks`.
- It can an also do the SSL Encryption and Decryption at its level , instead of these certificates installed at the instance level.

**`Auto Scaling Group`** :  Automates the process of increasing or decreasing the amount of provised on-demand instances availaible for your application.

 - Auto Scaling can increase or decrease the amount of instances based on `CloudWatch` metrics.

Auto Scaling has two main components :

- `Launch Configuration` : The template of the instance which will be launched.
- `Auto Scaling Groups` : All the rules and setting which defines the trigger for when the instance will be created or deleted.
      - Number of `MIN` and `MAX` instances allowed.
      - VPCs and AZs to launch instances into.


> For an architecture to be considered highly availaible and fault tolerant it MUST have an ELB service traffic to an Auto Scaling Group with a minimum of two instances in separate AZs.

## Workflow

It makes sense to create the Target Groups first and then the Load Balancers

> In the example below we are creating the following scenario and based on the content (pictures or videos) in the URL and route based on the same.

<img src="/assets/markdown-img-paste-20180322112423269.png" alt="Drawing" style="width: 400px;"/>


<img src="/assets/markdown-img-paste-20180322112318970.png" alt="Drawing" style="width: 400px;"/>

1. Create a Target Group
<img src="/assets/markdown-img-paste-20180322132805821.png" alt="Drawing" style="width: 400px;"/>
2. Add Target (instances) to the target group
<img src="/assets/markdown-img-paste-2018032213302429.png" alt="Drawing" style="width: 400px;"/>
<img src="/assets/markdown-img-paste-20180322133049555.png" alt="Drawing" style="width: 400px;"/>
3. Configure the Load balancer
<img src="/assets/markdown-img-paste-2018032213324375.png" alt="Drawing" style="width: 400px;"/>
4. Add the Target group created in step 1
<img src="/assets/markdown-img-paste-20180322133338478.png" alt="Drawing" style="width: 400px;"/>
5. Note that we have now the DNS name of the loadbalancer which can be accessed to access the service:
<img src="/assets/markdown-img-paste-20180322133500210.png" alt="Drawing" style="width: 400px;"/>
6. Additionaly when the Load balancer is created we can edit the rules for more advanced rules and routing:
<img src="/assets/markdown-img-paste-20180322134148144.png" alt="Drawing" style="width: 400px;"/>
Example is , that any time the path contains *pictures* we can forward it to the pictures target group , similary another group can be created for videos:
<img src="/assets/markdown-img-paste-20180322134322594.png" alt="Drawing" style="width: 400px;"/>

### Service Traffic to and from Private Web Servers

**`Bastion Host`**: A terminal server for getting into instance in the private subnet.

**`NAT Gateway`** : For providing internet access to the instances in the private subnet.
`NAT Gateway` is without an actual instance doing the NATing and `NAT Instances` uses an actual instance.

### ELB Troubleshooting

- Loadbalancing is  not occuring in multiple `Availaibility Zones`
Enable Cross-Zone Loadbalancing
<img src="/assets/markdown-img-paste-20180322161335110.png" alt="Drawing" style="width: 400px;"/>

- If the Instance not coming up as health in the AWS, Check the ping protocol and make sure it is right.
<img src="/assets/markdown-img-paste-20180322161603584.png" alt="Drawing" style="width: 400px;"/>

- Also check the security group setting of the ELB to ensure if it has the right ports allowed for the communication
<img src="/assets/markdown-img-paste-20180322162035550.png" alt="Drawing" style="width: 400px;"/>

- Access Logs show IP Address of ELB and not the source traffic. For this to work configure the S3 Access Logs  in the picture below.
<img src="/assets/markdown-img-paste-20180322162203901.png" alt="Drawing" style="width: 400px;"/>

- Unable to add instances from a specific subnet . In this case you need to make sure that the AZ of the instance in question should be listed in the `Edit Availabilty Zone` and then it will appear in `Edit Instances` .
<img src="/assets/markdown-img-paste-20180322162345922.png" alt="Drawing" style="width: 400px;"/>

### Auto Scaling Troubleshooting

- An Auto Scaled instance continues to Start and Stop or `Create`/`Terminate` instances in short intervals.
This could happen if the `trigger` is too close for example CPU threshold `< 30 , > 40 `

- Also keep a note of `Max` instaces as this may not let you spin more instances.

-------

## Storage Services

Notice the Transit service which are responsible for sending data in and out of S3 and Glacier.
![](/assets/markdown-img-paste-2018032219013824.png)

Different components of S3

![](/assets/markdown-img-paste-20180322190930356.png)

### S3 Permissions

`Bucket Policies` These are the permissions which are direclty assigned to the S3 Bucket . These are called Bucket Policies.

`S3 Access Control List` : For Public Access or AWS authenticated user.

![](/assets/markdown-img-paste-20180322193909333.png)


### S3 Storage Class

Object Durability : The percent the object is goign to be there. Not goign to be deleted or corrupted.
Object Availaibilty : Availaible for Access.


Different types of S3 Storage :

- `Standard`:  Durability : 99.999999999%  Availabilty : 99.99%
- `Standard Infrequent Access`:  Durability : 99.999999999%  Availabilty : 99.90%
- `Reduced Redundancy Storage `:  Durability : 99.99%  Availabilty : 99.99%
- `Glacier`:  Durability : 99.999999999%. Following are the recovery options :
  - `Expedited` : 1- 5 minutes.
  - `Standard`: 3-5 hours.
  - `Bulk`: 5-12 hours.

> To move object to Glacier you have to use `Life Cycle Policies`

#### S3 Versioning

By default versioning is disabled on all objects.  Once enable it cannt be disabled completely , but stopped for future objects.
Versioning can only be setup at the bucket level.

#### S3 Lifecycle managment

In the example below , we are moving the object to Infrequent Access to Glacier.

![](/assets/markdown-img-paste-20180322200344251.png)

#### S3 Event Notification

Allows you to setup automated communication between S3 and other AWS services when a selected event occurs.
Event can be sent to `SNS`, `Lambda`, `SQS Queue`
![](/assets/markdown-img-paste-20180322201116455.png)

#### S3 Static Web Hosting

Mainly lets say if your servers are unreachanble , they can atleast get a page notifying your company message rather than a 404.


#### CORS / Cross Origin Resource Sharing

Sharing accross wed domains
![](/assets/markdown-img-paste-2018032220164126.png)

To be able to share information between two different buckets for two different domains CORS have to be enabled.

#### Multipart Upload

Uploading files in pieces , *highly suggested* when objects are bigger than `100MB`

#### AWS Import Export

You send a physical device to AWS which AWS Uploads for you .

#### Snowball

AWS Send you a transfer appliance which you can copy your data to to send .
This is for PB Scale data.

#### Storage Gateway

Connects On-Premise Datacenter to S3 Storage

- `Gateway Cached Volumes` (Read as **locally** **cached** volumes): Create the volume and mount them as iSCSI devices. The data will be stored in S3 and frequently accessed data will be cached locally.
- `Gateway Stored Volumes` (Read as **locally** **stored** volumes) : All data is stored locally and snapshots of the same as incremental backup are stored in S3.


#### DNS Management

`Hosted Zones` : A Hosted Zone basically keep record of your DNS domain and know what to do with a DNS Request .

`Public Hosted Zone` : On the Internet

`Private Zone` : Withing Virtual Private Cloud .

`Record Sets` : Record Sets are instructions that actually match domain names to IP Address. Record set comprised of the following:
  - Record type
  - Standard/alias
  - Routing Policy
  - Evaluated target health

Common Record types include :

- `A` : Point to IPv4 address
- `AAAA` : Points to IPv6 IP address
- `CNAME` : Used to point to a hostname or other hostname.
`MX` : Point to a mail record.

#### CloudFront

CloudFront is a global CDN which delivers content from an `origin` location to an `edge` locations .

> In order to server a new version of an object, either create a new object with a new name or create an `invalidation` on the CloudFront distribution based on onject names.
> Invalidations have cost so if you have to invalidate large CloudFront distributions then perhaps you should create a new distribution and move DNS names.

#### VPN Connection

`Virtual Private Gateway` : It is the connector at the VPC Side , the `Customer Gateway` is the connector at the customer side.

![](/assets/markdown-img-paste-20180331234616674.png)

#### AWS Direct Connnect

AWS Direct Connect is a service that provides a dedicated network connection between your network and one of the AWS Direct connect providers.

Dedicated Private connection is like a VPN connection to your DC , it does not provide Internet .

![](/assets/markdown-img-paste-20180401000721669.png)

`Private Virtual Interface` : This allows you to interface with an AWS VPC with automatic route discovery using BGP , this requires a private of a public ASN number.

`Public Virtual Interfaces` : This allows you to have a direct connection to public AWS endpoints like Amazon S3 or DynamoDB


#### VPC Peering


![](/assets/markdown-img-paste-20180401001713511.png)

> Earlier VPC Peerign was only allowed withing the region , but with new feature VPC Peering can be done within different regions as well.


![](/assets/markdown-img-paste-20180401003523130.png)

Notice the appearance of vpc-peer entry to point to in the route table

![](/assets/markdown-img-paste-2018040100361218.png)


#### AWS CLI Configuration

`aws configure`

`aws s3 mb s3://testbucket`

#### AWS Database

![](/assets/markdown-img-paste-20180403080739308.png)

RDS is a fully managed database service , you do not have access to the instance.

RDS sizes like M1 , Large etc correcpoind to the instance sizes.

The following databases are supported :

- MySQL
- MariaDB
- PostgreSQL
- Oracle
- MS SQL Servers
- Aurora
  - Amazon Developed DB Service.


Benefits of RDS :
- Automatic minor updates
- Automatic backups.

#### AWS RDS

![](/assets/markdown-img-paste-20180401013344765.png)

`RDS Read Replica` : A DB copy which can be used for `READS` and can offload some load from the main DB .

`Multi AZ Failover` : RDS can be synchronously replicated to the `Standby` Instance which can be used for the failover.

### RDS Launch Demo

1. First This is to create a subnet group (you do not create the subnet but just associate the subnet already created)
![](/assets/markdown-img-paste-20180402210530609.png)
2. Select the instance type.
![](/assets/markdown-img-paste-20180402210745413.png)
3. Provide the SB Launch Details
![](/assets/markdown-img-paste-2018040221082210.png)
![](/assets/markdown-img-paste-20180402210941830.png)
4. View of a fully launched database
![](/assets/markdown-img-paste-20180402211024126.png)
Notice the `middle tab` with lot of information
![](/assets/markdown-img-paste-20180402211135353.png)
Creating `Read Replicas`
![](/assets/markdown-img-paste-20180402211953464.png)

### DynamoDB

DynamoDB is a serverless SQL Service. A fully managed NoSQL Service , similar to MongoDB.
All you need to do is define the throughput capacity. This scales automatically (grow and shrink)
AWS handles the backups as well.

Very straight forward configuration
![](/assets/markdown-img-paste-20180403080044390.png)

Used for storing : IoT , Gaming , Mobile

### ElasticCache

It a fullymanaged `in-memory` cache engine. It caches large taxing queries on the DB which increases performance.

memcached and redis are the currently availaible engines to power the ElasticCache.

### RedShift

Amazon Redshift is a petabyte scale datawareshousing solution, fully managed and scalable and is generally used for Big Data analytics.

## AWS Application Services

![](/assets/markdown-img-paste-20180403080953828.png)

### SNS
![](/assets/markdown-img-paste-20180403081047797.png)
SNS corodinates and manages the sending and delivery of messages to specific endpoints.
Since it is integrated into many services natively it is very easy to set this up .

SNS is broken down into 3 main parts:
- **`Topic`** : Topic is a group of subscriptions you send the messages to.
- **`Subscription`** : An endpoint to which the message is sent to.
  - `HTTP`, `HTTPS`, `Email`, `Email-JSON`, `SQS`, `Mobile Apps`, `Lambda`, `SMS`
- **`Publisher`** : Entity that triggers the sending of the event , for examle , Human, S3 Event, Cloudwatch Alarm.

#### Workflow Example
  - Create a Topic
  ![](/assets/markdown-img-paste-20180403082358366.png)
  - Now , within the topic create a subscription
  ![](/assets/markdown-img-paste-20180403082547509.png)
  ![](/assets/markdown-img-paste-20180403082616699.png)
  - Now click on Publish a topic
  ![](/assets/markdown-img-paste-20180403082801498.png)
  ![](/assets/markdown-img-paste-20180403082920738.png)
  And finally publish the message to the email subscription created above
  ![](/assets/markdown-img-paste-20180403083036726.png)

<br><br>


### SQS Essentials

![](/assets/markdown-img-paste-20180405062412916.png)

> SQS Is a fully managed service by AWS

Simple Queing Service allows the creation of decoupled services. Messages between servers are retrieved through polling.

There are two types of polling:

1. Long polling : Long polling (1-20 seconds) , Less number of API Calls.
2. Short polling : More frequent polling , more number of API calls.

SQS can contain upto 256KB of text (in any format)

Amazon SQS offers two different types of queues :

1. `Standard Queue` : Does not gaurantee FIFO but tries the same . It gurantees the delivery of the message atleast once.
2. `FIFO Queue` : For the applications where order is important , or where duplicates can not be tolerated.


### SWF , Simple Workflow Service

SWF is a fully managed workflow service provided by AWS.  A workflow execution can last upto 1 year.

Components of SWF :

 - `Workflow` : A sequence of steps required to perform a speficific task.
 - `Activities` : A `single step` in the workflow.
 - `Tasks`
   - `Activity Task` : Tells the worker to perform a specific task.
   - `Decision Task` : Tells the decider the sate of the workflow execution and allows him to take the decision on the next steps.
 - `Worker` : Responsible for taking a task and working on it .

### AWS API Gateway

API Gateway is a fully managed service which allows you to create and manage your own APIs.

Lets say you have an application running on Lambda Functions and EC2 .

![](/assets/markdown-img-paste-20180405065436505.png)

You can `version` control your APIs as well.
![](/assets/markdown-img-paste-20180405065740886.png)

CloudFront Edge locations are the entrypoint , which help with DDoS protection as well.

API Gateway also caches the responses as well .

### CloudWatch Essentials

CloudWatch is used to monitor AWS Services, such as EC2,ELB and S3.
You monitor your environment by configuring and viewing Cloudwatch metrics.

![](/assets/markdown-img-paste-20180405070816470.png)

CloudWatch Alarms allow for you to be notfiied when a certain defined threshold is met for CloudWatch Metrics.

Tuesday : 3 - 4 PM
Thursday : 3 -


-------

## Hybrid Environments



-------

# Advanced Networking (START HERE IN LINUXACADEMY)
-------

## Advanced VPC Networking
-------

## Network Troubleshooting
-------

## Storage Services
-------

## Hybrid Environments
