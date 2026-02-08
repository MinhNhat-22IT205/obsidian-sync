### 1. Identity (IAM User, Group, Role) Policies
+ Piorities:
***Deny alway wins***
![[Pasted image 20260130211801.png]]
![[Pasted image 20260130212109.png]]
In case there are multiple policy definition, AWS sum them all to evaluate

+ Types (based on how they get managed):
	+ Inline policy: assign policy to each individual account
		-> Use when exceptional allow or deny
	+ Manged policy: A reusable way
		-> Should be first consider

### 2. IAM Users and ARNs
#### When should you pick IAM User?
-> when you can identity a **single individual** or **'a named thing'** which will use that identity - a person, an application or a service account.

![[Pasted image 20260131082246.png]]
U&P for User
Access Keys for application
#### Amazon Resource Name (ARN)
It is *how a resource in AWS is identified*.
Ex:
![[Pasted image 20260131090340.png]]
* partition: 'aws' or 'aws-cn' (for china aws)

*Facts:*
![[Pasted image 20260131093202.png]]
So beware in situations like:
+ Internet-scale application
+ Large organization 
	-> Fix using: IAM Roles & Identify Federation

### 3. IAM Groups

Simply just a cotainer of IAM Users
For applying permission to it only
Nothing special

### 4. IAM Roles

##### WHen to use?
* For AWS service - Lambda