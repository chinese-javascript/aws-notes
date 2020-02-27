# Cognito
AWS Cognito is an Identity Broker which handles interaction between your applications and a Web ID provider (Google, Facebook, Amazon). Following successfull authentication, the user receives an authentication code from the Web ID provider, which gets traded for temporary AWS security credentials e.g. to access data in S3 or DynamoDB.

# STS
AWS Cognito interfaces with AWS Security Token Service (STS) to request temporary credentials for authenticated users. Once a user has authenticated with the Web ID provider, the application makes the `sts assume-role-with-web-identity` API call to STS.

## AssumeRoleWithWebIdentity API Response
* **Credentials**: The temporary security credentials, which include an access key ID, a secret access key, and a security token.
* **AssumedRoleUser**: The ARN and the assumed role ID, which are identifiers for the temporary security credentials that you can programatically refer to.

# Workflow
The process of authenticating a user with Cognito is as follows:
1. The user signs in with a Web ID provider (Google, Facebook, Amazon, etc.)
2. The Web ID provider returns a JWT token to the user.
3. The user application makes an STS API call: `sts assume-role-with-web-identity`
4. STS returns an API response with the temporary credentials.
5. The user application now has AWS access e.g. for S3, DynamoDB, etc.
