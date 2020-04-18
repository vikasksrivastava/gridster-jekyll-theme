
![](/assets/markdown-img-paste-20190728152651559.png)

<p align="center">
  <img width="200" height="100" src="assets/markdown-img-paste-20191029211927809.png">
</p>

** Speak Slow **

Quick Video to Demonstrate IAM roles

 IAM Roles and Its Importance from a Security perspective.

---


1. Show An Instance without an S3 Access : "aws s3 ls" - Unable to Access

Instead of hardcoding credentials : Attach a Role that permits that .

2. Attach a Role Directly and show how simple it is -- aws s3 ls
3. Remove the role and show back again .

----- If you do not do this ...

2. Show the challenges with "aws configure"
3. Where it stores the password ./aws/credentials

--- Where are roles created ...

Now Show where the roles are created .  EC2 --- Access to Route 53 Commands . "Route53FullAccess"


```
graph TD
C(CTS API Calls)
C -->|In Assume Roles <br> You are getting a policy tied to a IAM Role | D[AssumeRole<br>AssumeRoleWithWebIdentity <br> AssumeRoleWithSAML <br><br>Max Life Time : 12 hrs ]
C -->|In Get Operations <br> you are getting credentials of a specific IAM User| E[GetFederationToken <br> GetSessionToken  <br><br>Max Life Time : 36 hrs]
E --> |GetFederationToken|F(When you make a call to <br> a GetFederation token <br>you define a user with <br>all the permissions everyone in <br>your company needs and filter<br> it down based on request basis.)
E --> |GetSessionToken| G(A person who would <br>normally use his permanent credentials <br>but is logging in from an <br>untrusted device , <br>hence can request safer temporary<br> credentials then permanent)

```
