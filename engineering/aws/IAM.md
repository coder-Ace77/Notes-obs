
---

IAM - identity and access management, global service. 

Root user is created by default and should not be shared. 
Users are people in organization and can be grouped. Groups only contain users not other groups. Users don't have to belong one group and can belong to multiple groups. Groups can overlap and can be veried to a great degree. 

Users and groups can be assigned a JSON document called policies or IAM policies. they describe in plain english what a user is allowed and not allowed to do. 

sample format

```json
{
	"Version":"2012-10-17",
	"Id":"optional for identifications",
	"Statement":[
		{
			"Effect":"Allow",
			"Principal":{
				"AWS":["arn:aws:iam:1234123234:root"]
			}
			"Action":[
				"S3:GetObject",
				"S3:PutObject"
			],
			"Resource":[
				"arn:aws:s3:::mybucket/*"
			]
		}
	]
}
```

Most important here is a statement. Statement is an array of objects. Each object defines single permission

1. Effect - "Allow"/"deny" defines weather statement allows or denies access. 
2. Principal - account/user/role to which this policy is applied to 
3. Action - A list of actions this policy allows or denies 
4. Resource - A list of resources to which this action applies to

Finally Action and Resource can have `"*"` meaning allowed to do everything.

IAM policy assigned at the group will be assigned to every person of the group. 
MFA - Multifactored access - Need atleast one more security device to access the account. 

AWS can be accessed by cli terminal as well which is protected via access keys. 
AWS SDK are ways in which our applications can access the aws and are also managed by access keys. 

Access Key ID = username
Secret access key = password

AWS provides an cli which can be used to access the aws account. 

IAM roles can also be given to services for example ec2 because it may want to perform some action on other resource for instance S3 bucket. 

An **IAM Role** is an **identity in AWS that has a set of permissions**, but _unlike a user_, it **does not belong to a specific person**.

Instead, **roles are assumed** by:

- AWS services (e.g., EC2, Lambda)
- Users (temporary credentials)
- Applications
- External identities (e.g., GitHub OIDC, another AWS account)

When someone or something assumes a role, AWS gives **temporary security credentials**. Creating iam role can be done directly from iam console. 

IAM credentials report is an account level report that lists all the account's user and status of all credentials.
IAM access advisor shows theb service permissions granted to user and when these were last granted. 

**A role is an identity with permissions, but without a password or access key.**

Users login.  
Roles do **not** login.  
Roles are **assumed** by:

- AWS services (EC2, Lambda, ECS, Glue…)
- Other AWS accounts
- Federated users (SSO, Google/Microsoft sign-in)

## Shared responsibility model

AWS assumes the shared responsibility model so as a customer we are responsible for the secuty **in** the cloud and amazon is responsivble for the security **of** the cloud. 

![[Pasted image 20251118145202.png]]

Shared responsibility model for IAM:

Here aws will ensure - 

1. Infra (global network security), configuration and vulnerability validation. 

We have to ensure - 

1. User , gp ,roles , policies , monirtoring 
2. Enabling MFA on all accounts 
3. Analysing the access patterns reviewing permissions.


IAM security tools 

1. IAM credential reports (account level) - A report that lists all the account's users and the status of various credentials

2. IAM Accesss advisor (user level) - Shows the service permission granted to user and when these services were last granted.  