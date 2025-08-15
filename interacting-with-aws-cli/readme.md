# Demo Project:  
**Interacting with AWS CLI**

## Technologies used:  
- AWS  
- Linux  

## Project Description:  
- [Install and configure AWS CLI tool to connect to our AWS account](#install-and-configure-aws-cli-tool-to-connect-to-our-aws-account)  
- [Create EC2 Instance using the AWS CLI with all necessary configurations like Security Group](#create-ec2-instance-using-the-aws-cli-with-all-necessary-configurations-like-security-group) - [Create SSH key pair](#create-ssh-key-pair)  
- [Create IAM resources like User, Group, Policy using the AWS CLI](#create-iam-resources-like-user-group-policy-using-the-aws-cli)  
- [List and browse AWS resources using the AWS CLI](#list-and-browse-aws-resources-using-the-aws-cli)  

---

## Install and configure AWS CLI tool to connect to our AWS account

1. Open your terminal and type:  
```bash
brew install awscli
```

2. Check the AWS CLI version: **aws --version**
3. **Create Admin User (if not already created)**  
   - Go to **IAM** in the AWS Management Console.  
   - Add a new user named `admin`.  
   - Provide **AWS Management Console** access.  
   - Assign the policy **AdministratorAccess**.
4. **Create Access Keys**  
   - Go to **Users**, select `admin`, and click on the **Security Credentials** tab.  
   - Scroll to **Access keys** and click **Create access key**.  
   - Select **Command Line Interface**.  
   - Download the `.csv` file containing the keys and click **Done**.
5. In the terminal, type: **aws configure**
**Use the admin user details:**
- **Access Key ID** and **Secret Access Key**: Copy from the downloaded `.csv` file.  
- **Default region**: `eu-central-1` (Frankfurt)  
- **Default output format**: `json`
6. Configuration is stored in the home directory under `~/.aws`: **ls ~/.aws/**

## Create EC2 Instance using the AWS CLI with all necessary configurations like Security Group

1. List all available security groups:  
```bash
aws ec2 describe-security-groups
```

2. Get the IDs of existing VPCs:
```bash
aws ec2 describe-vpcs
```

3. Create a new security group
	- You need a VPC ID from the previous step:
```bash
aws ec2 create-security-group --group-name my-sg --description "my sg" --vpc-id <vpc-id-from-above>
```

4. Get more information about the security group:
```bash
aws ec2 describe-security-groups --group-ids <security-group-id>
```

5. Allow SSH (port 22) to the new instance
- Replace `<group-id-from-above>` and `<our_own_ip_address>` with your values:
```bash
aws ec2 authorize-security-group-ingress \
  --group-id <group-id-from-above> \
  --protocol tcp \
  --port 22 \
  --cidr <our_own_ip_address>/32
```

6. Check the security group again 
```bash
aws ec2 describe-security-groups --group-ids <security-group-id>
```

7. Verify in AWS Console
- Go to EC2 Instances and refresh.
- Navigate to Security Groups and check the rules displayed.

## Create SSH key pair

```bash
   aws ec2 create-key-pair \
     --key-name MyKpCli \
     --query 'KeyMaterial' \
     --output text > MyKpCli.pem
```

1. List all subnets:
```bash
aws ec2 describe-subnets
```

2. Select a subnet
> Choose one in *zone 1* and copy its *Subnet ID*.
3. Get the Image ID (AMI)
- Go to the AWS Console → EC2 → Launch Instance.
- Copy the *Image ID* of a *Linux AMI*.
4. Run a new EC2 instance with all the details from above:
```bash
aws ec2 run-instances \
  --image-id <image-id-from-above> \
  --count 1 \
  --instance-type t2.micro \
  --key-name MyKpCli \
  --security-group-ids <security-group-id-created-above> \
  --subnet-id <subnet-id-from-above>
```

5. Check the instances: `aws ec2 describe-instances`
6. Copy the public IP address needed to SSH into the instance.
7. Change PEM file permissions:
```bash
chmod 400 MyKpCli.pem
```

8. Verify PEM file permissions: `ls -l MyKpCli.pem`
9. SSH into the instance: 
```bash
ssh -i MyKpCli.pem ec2-user@<public-ip-address>
```

## Create IAM resources like User, Group, Policy using the AWS CLI

1.  **Add a new group**  
```bash
aws iam create-group --group-name MyGroupCli
```

2. **Add a new user**
```bash
aws iam create-user --user-name MyUserCli
```

3. **Add the user to the group**
```bash
aws iam add-user-to-group --user-name MyUserCli --group-name MyGroupCli
```

4. Check group information: `aws iam get-group --group-name MyGroupCli`
5. Give the user permission for EC2
- Go to the *AWS Console → IAM → Policies*.
- Search for *EC2 Full Access*, click it, and copy the *Policy ARN*:
```bash
arn:aws:iam::aws:policy/AmazonEC2FullAccess
```

- Alternatively, if you know the policy name, you can get the ARN using AWS CLI:
```bash
aws iam list-policies --query "Policies[?PolicyName=='AmazonEC2FullAccess'].Arn" --output text
```

6. Attach the policy to the group
```bash
aws iam attach-group-policy --group-name MyGroupCli --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
```

7. Validate that the policy is attached:
```bash
aws iam list-attached-group-policies --group-name MyGroupCli
```

### Create Credentials for the New User

1. Create login profile with password and access keys:
```bash
aws iam create-login-profile --user-name MyUserCli --password MyPassword! --password-reset-required
```

2. Get the user details to copy account ID:
```bash
aws iam get-user --user-name MyUserCli
```

3. Validate login
- Use the *Account ID* from the *ARN* above to log in to the AWS Console with MyUserCli.

### Create policy and assign to group to change password.

# Create a JSON File for Password Policy

1. **Create a new JSON file in the terminal**  
```bash
vim changePwdPolicy.json
```

2. Copy the following json and save it:
```json
{
  "Version": "2025-08-15",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:ChangePassword"
      ],
      "Resource": [
        "arn:aws:iam::330673547330:user/${aws:username}"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "iam:GetAccountPasswordPolicy"
      ],
      "Resource": "*"
    }
  ]
}
```

3. Create the IAM policy:
```bash
aws iam create-policy --policy-name changePwd --policy-document file://changePwdPolicy.json
```

4. Attach the policy to the user group:
```bash
aws iam attach-group-policy --group-name MyGroupCli --policy-arn <policy-arn-from-above>
```

5. Verify in AWS Console
- Check that the policy is attached.
- Confirm that users can successfully change their passwords.

## List and browse AWS resources using the AWS CLI

1. Create Access Keys for a New User
```bash
aws iam create-access-key --user-name MyUserCli
```

2.  Set Environment Variables for a New User
- To avoid overwriting the default admin user, set environment variables for `MyUserCli`:
```bash
export AWS_ACCESS_KEY_ID=<access-key-of-MyUserCli>
export AWS_SECRET_ACCESS_KEY=<secret-access-key-of-MyUserCli>
export AWS_DEFAULT_REGION=eu-west-2
```

- Execute Allowed Commands, for example, list EC2 instances:
```bash
aws ec2 describe-instances
```

- Try creating a new IAM user (should fail):
```bash
aws iam create-user --user-name test
```

- You should receive an Access Denied error.

- Explore Command Shortcuts / Aliases

To see the list of delete commands in EC2:
```bash
aws ec2 bb
```

To see similar shortcuts in IAM:
```bash
aws iam aa
```
