
# 1. Account
Term to understand: "Sign-in for IAM users in AWS Account"
## Root & externals relationship
+ AWS account is the **environment**
+ Root is a **special user**, initially created when AWS Account is created

![[Pasted image 20260123122538.png]]
## IAM
+ IAM is what allows **additional identities** to be **created** within an AWS account
+ IAM identities start with **no permissions** on an AWS Account, but can be granted permissions by the Account Root User.
![[Pasted image 20260123132302.png]]
Feat(s) about IAM:
+ **No cost**
+ **Globally**
+ Only control **local** Identities in your AWS Account 

# 2. Access
## Access keys
+ Access keys are how the **AWS Command Line Tools** (CLI Tools) interact with AWS accounts.
+ Access keys are created for IAM user
+ Access keys includes 2 parts:
	+ **Access key ID** ~ like username -> public
	+ **Secret access key** ~ like password -> one-time access
	![[Pasted image 20260123142618.png]]

