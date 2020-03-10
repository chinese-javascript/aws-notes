# Identity Access Management
The IAM service provides centralised control of your AWS account. The key entities in IAM are **Users**, **Groups**, **Roles** and **Policies**.

IAM is universal - your configured entities will span across all regions.

## Best Practices
* Activate MFA
* **Least Prviliege** - Give users the minimum amount of access required
* **Use Groups** - apply IAM policies to groups instead of individual users.
* Apply an IAM password policy
* Use roles for applications running on EC2 instances
* Rotate credentials regularly

# Identities
## Users
* Represents an identity that interacts with AWS
* Can have the following types of access:
  * **Programmatic Access**: Access the AWS API, CLI, SDK, etc. using an *access key ID* & *secret access key*. **Best practice is to create separate key-pairs per developer with programmatic access.**
  * **Management Console Access**: Login to the AWS Console with an *account name* & *password*
* Multiple users can be placed in *Groups*
* The root account is created when first setting up an AWS account. It has complete admin access. **Best practice is to delete the AWS root account access keys, use a dedicated IAM user with admin access instead**

### AWS CLI
Users with programmatic access can use the AWS CLI to interact with AWS resources via the command line.

For general use, the `aws configure` command is the fastest way to set up the AWS CLI.
```shell
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```

By default, when calling an API via the CLI, the API uses a page size of 1000 when returning a response.
E.g. when running `aws s3api list-objects my_bucket` on a bucket containg 2500 objects, the CLI makes 3 API calls to S3 (to get 3 * 1000 items), but returns all 2500 results in one go.

Depending on the resource, the default 1000 page size may be too high and cause a 'Timed Out' error. To change the page size when calling the API, use the `--page-size` or `--max-items` option.
* `--page-size`: Set the number of items per page. Set to a lower number to prevent 'Timed Out' errors, however the CLI will end up making more calls to the API.
* `--max-items`: Set the max number of items to return to the CLI. Will not return all items.

### Working with Server Certificates
To enable HTTPS connections to your application in AWS, an SSL/TLS *server certificate* is required. **AWS Certificate Manager (ACM)** can provision, manage and deploy your server certificates (supported in select regions only). In regions not supported by ACM, use **IAM certificate store** instead.

## Roles
* Assigned to trusted entities and provides temporary access to AWS resources. Types of trusted entities:
  * AWS service e.g EC2, Lambda
  * Another AWS account belonging to you or 3rd party
  * Web identity (Cognito or any OpenID provider)
  * SAML 2.0 federation e.g your corporate directory
* Each role has a set of permissions/policies detailing what accesses they have
* Roles are preferred over users with access keys

### Instance Profiles
An *instance profile* is a container for an IAM role that, when applied to an EC2 instance, will provide the role's policies on applications running within the EC2 instance. 

## Temporary Security Credentials
You can use the AWS Security Token Service (AWS STS) to create and provide trusted users with temporary security credentials that can control access to your AWS resources.

### Requesting Temporary Security Credentials
To request temporary security credentials, you can call the AWS STS API via one of the following methods:
* **AWS SDK** - provides additional features such as cryptographically signing your requests, retrying requests if necessary, and handling error responses.
* **AWS CLI**
* **AWS Tools for Windows PowerShell**

The AWS STS API provides methods for 
You assume an IAM role by calling the AWS Security Token Service (STS) AssumeRole APIs (in other words, AssumeRole, AssumeRoleWithWebIdentity, and AssumeRoleWithSAML). These APIs return a set of temporary security credentials that applications can then use to sign requests to AWS service APIs.


# Access Management
## Policy Documents
* A document that defines permissions and access to resources
* All access is denied by default unless specified otherwise
* “Deny” rules take priority over “Allow” rules
* Can be attached to *Users*, *Groups* and *Roles*

### A Sample IAM Policy Document

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ManageRichardAccessKeys",
            "Effect": "Allow",
            "Action": [
                "iam:*AccessKey*",
                "iam:GetUser"
            ],
            "Resource": "arn:aws:iam::*:user/division_abc/subdivision_xyz/Richard"
        },
        {
            "Sid": "ListForConsole",
            "Effect": "Allow",
            "Action": "iam:ListUsers",
            "Resource": "*"
        }
    ]
}
```

### IAM Policy Simulator
The IAM Policy Simulator tool provided by AWS (located at https://policysim.aws.amazon.com) allows you to test the effects of IAM policies before committing to production. It will let you know whether a specific user/group/role has access to execute a specific action in AWS depending on the contents of the policy document.

