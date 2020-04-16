---
layout: post
title: AWS IAM Integration with Active Directory for SSO/SAML
description: AWS IAM Integration with Active Directory for SSO/SAML
comments: true
---

**This content is based on**

<iframe width="560" height="315" src="https://www.youtube.com/embed/VP9wBGm982g?controls=0&showinfo=0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

> **2018 AWS Federated Authentication with Active Directory Federation Services (AD FS)** https://aws.amazon.com/blogs/security/aws-federated-authentication-with-active-directory-federation-services-ad-fs/


![](/assets/markdown-img-paste-20191124215412168.png)

1. Corporate user accesses the corporate Active Directory Federation Services portal sign-in page and provides Active Directory authentication credentials.
2. AD FS authenticates the user against Active Directory.
3. Active Directory returns the user’s information, including AD group membership information.
4. AD FS dynamically builds ARNs by using Active Directory group memberships for the IAM roles and user attributes for the AWS account IDs, and sends a signed assertion to the users browser with a redirect to post the assertion to AWS STS.
5. Temporary credentials are returned using STS `AssumeRoleWithSAML`.
6. The user is authenticated and provided access to the AWS management console.



<iframe src="https://player.vimeo.com/video/396773347" width="800" height="600" frameborder="0" allow="autoplay; fullscreen" allowfullscreen></iframe>

## Install the Required Services

- Install these servcies , `Active Directory Domain Services` , `Active Directory Federation Services` and `DNS` , `IIS`

## Install Active Directory

- Setup : AD for the name `devopssimplified.com`
- Restart the Computer

## Configure Basic Active Directory

-  **Create Group** : `AWS-3443243287102-ADFS-AD-DevOps-Prod` - This naming conventions is best practices based , on the AWS documentation.

- Create User : bob@devopssimplified.com and **add to the group above**
  - **Make sure email address is set**

---
## ADFS Configuration Preparation
### While we are here we configure the user required for ADFS Services

- Create user : `adfssvc`
- This will be used by the ADFS Service which we will use later.and run
- Also set the `setspn - a host/localhost adfssvc` to mitigate an error received during the ADFS configuration. **COMPLAINS**


### Now lets Configure a Self Signed Cert

- Go to `IIS Configuration`
- Create **self** signed certificate `webcert`
- Export it to the desktop

---

## Now that the ADFS Confiration base work is done lets move on to Configuring  ADFS Service

`webcert` | `WebHosting` in the menu .

---

---
# Configuring Trust Relation ship and SAML
## Now go to ADFS managment Console (To setup trust from AD side)

## Step 1

![](/assets/markdown-img-paste-20191122234850127.png)
Download the File from https://signin.aws.amazon.com/static/saml-metadata.xml

## Step 2

![](/assets/markdown-img-paste-20191122235024686.png)

## Step 3

https://aws.amazon.com/SAML/Attributes/RoleSessionName
![](/assets/markdown-img-paste-20191122235050515.png)

## Step 4

![](/assets/markdown-img-paste-20191122235242117.png)
```sh
c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname", Issuer == "AD AUTHORITY"]
 => add(store = "Active Directory", types = ("http://temp/variable"), query = ";tokenGroups;{0}", param = c.Value);
```
## Step 5

```sh
c:[Type == "http://temp/variable", Value =~ "(?i)^AWS-([\d]{12})"]
 => issue(Type = "https://aws.amazon.com/SAML/Attributes/Role", Value = RegExReplace(c.Value, "AWS-([\d]{12})-", "arn:aws:iam::743230357567:saml-provider/idp1,arn:aws:iam::743230357567:role/"));
```

![](/assets/markdown-img-paste-20191122235347442.png)

---
## Trust relationship from AWS Side
Go to AD and download https://localhost/federationmetadata/2007-06/federationmetadata.xml

## Import this file in AWS, Name it `idp1`

- We name it `idp1` hust for follwign the guide we referenced earlier.

**IAM --> Indentity Providers --> Creta provide --> SAML**
## Now in AWS IAM Create a SAML Role which the user will choose to Assume

`ADFS-AWS-Admin`

---

# Verification / SAML Tracer

## Now login to ADFS Single Sign on on URL
https://localhost/adfs/ls/idpinitiatedsignon.aspx


Good Working SAML Response

```xml
<samlp:Response xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol"
                ID="_6a302b8d-303c-4498-86dc-27112ccb0c3b"
                Version="2.0"
                IssueInstant="2019-11-25T07:02:13.056Z"
                Destination="https://signin.aws.amazon.com/saml"
                Consent="urn:oasis:names:tc:SAML:2.0:consent:unspecified"
                >
    <Issuer xmlns="urn:oasis:names:tc:SAML:2.0:assertion">http://ad-server.devopssimplified.com/adfs/services/trust</Issuer>
    <samlp:Status>
        <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success" />
    </samlp:Status>
    <Assertion xmlns="urn:oasis:names:tc:SAML:2.0:assertion"
               ID="_5399b751-bddf-4f2a-806f-245589d4fc7b"
               IssueInstant="2019-11-25T07:02:13.055Z"
               Version="2.0"
               >
        <Issuer>http://ad-server.devopssimplified.com/adfs/services/trust</Issuer>
        <ds:Signature xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
            <ds:SignedInfo>
                <ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
                <ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256" />
                <ds:Reference URI="#_5399b751-bddf-4f2a-806f-245589d4fc7b">
                    <ds:Transforms>
                        <ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature" />
                        <ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#" />
                    </ds:Transforms>
                    <ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256" />
                    <ds:DigestValue>dmBvdXODcozeo4BL3CsOh8E3vm0Ps0blwzzMDBUCbr4=</ds:DigestValue>
                </ds:Reference>
            </ds:SignedInfo>
            <ds:SignatureValue>GlmrHkPuNlvWrjbas00vHl9B5BiUs22WSr5sFRRqzmAeFT9cZ8SbA2yQZg+BvMCO6aS2lAt8Qty0NJR6CIeY3QX0JbTwlal4YnRHaGhN+EEJWjFTXShNaBAwHssLiKbSLPnvtnU9O1Tzgu0UKhuPHySFDVoy8Qtcc4YqfVkXPyCeKS8v8dNMVjgns+Dxmdt5cW+OLxYOa8TiZOvEOqa6C8VW0EDsokX74+rCaRMOremGMLftYQ4bouyGM6JsjBMdoXlZ5TzuMtIdAgyk+nBEkXdwNrAYY1Gbu/BWLBjF9l4CPVrcrrtCV2X/+N4OomvSHIKMtZ0UP5LgkgD/hW5BuQ==</ds:SignatureValue>
            <KeyInfo xmlns="http://www.w3.org/2000/09/xmldsig#">
                <ds:X509Data>
                    <ds:X509Certificate>MIIC+DCCAeCgAwIBAgIQbXeQtdTwvb5GKTpJe94/bTANBgkqhkiG9w0BAQsFADA4MTYwNAYDVQQDEy1BREZTIFNpZ25pbmcgLSBhZC1zZXJ2ZXIuZGV2b3Bzc2ltcGxpZmllZC5jb20wHhcNMTkxMTI1MDUzNDMzWhcNMjAxMTI0MDUzNDMzWjA4MTYwNAYDVQQDEy1BREZTIFNpZ25pbmcgLSBhZC1zZXJ2ZXIuZGV2b3Bzc2ltcGxpZmllZC5jb20wggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDTkYyWsE06YfrglQjazfs4tz8MjmEvMOPdpj/rJSKM/UstLDYRcMybjm8DsuiE36/ZRumj9DQUD60hacFfcmoYVthpHh9LZN4ifPpTe6XR7z8bMkplBt2oXTFjy+lLx3jN7kP6YlrxBrAIBM1HTWL0FEhfKtR3PKrGA8/EDbxkq9WyIxbx5c0cugr4N5fuTwmJWCj4oZxxRVEzP9fsQUCUy/eHL3sxd7dyS2fjN7qnL/ZUCC0hX7DiFbIZQmal9UrJhjodDEXELZbQ6iRYHMAg409nhUeK+i4A6KjmjwnWoU+/7r0SxjawaNTq85QaIx3NLuC1pJsh7fqfwMBMGKH/AgMBAAEwDQYJKoZIhvcNAQELBQADggEBAMo8BH8kdDXksfvfbKlKKT1Z1FPSVqvQJ7rBRAh6GIsrnD93INs8GywekLqxTQ1K0AHFWMeobCEyc1SoK3e2Q4x0/K6ISm4f3nPdHNcmFMI64D1GOmF7V4ILrhOIZZc8bOkhxAA/hmBp4+Zjvvc6Oaptn3wfsw+dnFRwle+UUqIE80yCptPIFvjN5eUmIhxzpekrfxjAS+tz9bge682EIDe/MZyRY+LB1RdSV3+WmBA95lEvc/oMhZGNfqf96xe2xFVjsJ8y4UMdgez1MjqYPUdQOvrfNfmzypQGbupAnEG6qMN/D0C516HilmWf5519BxOYqPTySoZJUoFgTLGQbDQ=</ds:X509Certificate>
                </ds:X509Data>
            </KeyInfo>
        </ds:Signature>
        <Subject>
            <NameID Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent">DEVOPSSIMPLIFIE\bob</NameID>
            <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
                <SubjectConfirmationData NotOnOrAfter="2019-11-25T07:07:13.056Z"
                                         Recipient="https://signin.aws.amazon.com/saml"
                                         />
            </SubjectConfirmation>
        </Subject>
        <Conditions NotBefore="2019-11-25T07:02:13.053Z"
                    NotOnOrAfter="2019-11-25T08:02:13.053Z"
                    >
            <AudienceRestriction>
                <Audience>urn:amazon:webservices</Audience>
            </AudienceRestriction>
        </Conditions>
        <AttributeStatement>
            <Attribute Name="https://aws.amazon.com/SAML/Attributes/RoleSessionName">
                <AttributeValue>bob@devopssimplified.com</AttributeValue>
            </Attribute>
            <Attribute Name="https://aws.amazon.com/SAML/Attributes/Role">
                <AttributeValue>arn:aws:iam::743230357567:saml-provider/idp1,arn:aws:iam::743230357567:role/ADFS-Admin</AttributeValue>
            </Attribute>
        </AttributeStatement>
        <AuthnStatement AuthnInstant="2019-11-25T07:00:34.467Z"
                        SessionIndex="_5399b751-bddf-4f2a-806f-245589d4fc7b"
                        >
            <AuthnContext>
                <AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport</AuthnContextClassRef>
            </AuthnContext>
        </AuthnStatement>
    </Assertion>
</samlp:Response>

```

**"Error: RoleSessionName is required in AuthnResponse (Service: AWSSecurityTokenService; Status Code: 400; Error Code: InvalidIdentityToken)"**

> **Solution** Email Address Under User was not configured !

```
Duo
https://help.duo.com/s/article/4045?language=en_US

distinguishedName
mail
sAMAccountName
userPrincipalName
```

<iframe width="700" height="500"
src="https://www.youtube.com/embed/VP9wBGm982g"
frameborder="0"
allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture"
allowfullscreen></iframe>
