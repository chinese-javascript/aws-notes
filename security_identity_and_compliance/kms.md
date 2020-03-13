# Key Management Service (KMS)
AWS KMS provides management of encryption keys for use with other AWS services (e.g. EBS, S3, RDS, etc.).

## Concepts
### Customer Master Keys (CMKs)
The primary resource of AWS KMS is the *Customer Master Key* which is used to generate, encrypt and decrypt the data keys (aka. envelope key) used for data encryption in other integrated AWS services. CMKs cannot be exported outside of the KMS service. They are generated based on the key metadata attributes:

* Alias
* Creation date
* Description
* Key state
* Key material (either customer-provided or AWS-provided)

When setting up a CMK, alongside key metadata, access must be defined as well for:

* **Key Administrative Permissions**: IAM entities that can administer this key through the KMS API.
* **Key Usage Permissions**:  IAM entities that can use this key to encrypt/decrypt data when using AWS services integrated with KMS.

To create a data key, call the `GenerateDataKey` operation. AWS KMS uses the CMK that you specify to generate a data key. The operation returns a plaintext copy of the data key and a copy of the data key encrypted under the CMK. `GenerateDataKeyWithoutPlaintext` will only return the encrypted data key.

![](https://docs.aws.amazon.com/kms/latest/developerguide/images/generate-data-key.png)

## Data Keys
Data keys are used for encrypting data in other AWS services integrated with KMS (e.g. S3 or EBS). Data keys are not stored within KMS, instead are managed outside in their respective AWS services. Only the plaintext version of the data key can be used for encrypting, and once the operation is done it should be removed. The encrypted version can be safely stored for decryption later.

![](https://docs.aws.amazon.com/kms/latest/developerguide/images/encrypt-with-data-key.png)

The `Decrypt` operation will use the CMK to decrypt the encrypted data key back into the plaintext version, which can then be used to decrypt encrypted data.

![](https://docs.aws.amazon.com/kms/latest/developerguide/images/decrypt.png)

## API Reference
* `aws kms encrypt`
* `aws kms decrypt`
* `aws kms re-encrypt`
* `aws kms enable-key-rotation`
