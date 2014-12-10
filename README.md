## Backend-less dropbox clone

This sample uses Auth0 and its integration with AWS APIs (S3, SES, DynamoDB, EC2, etc.) in combination with the powerful IAM policies.

**Demo: <http://auth0.github.io/auth0-s3-sample>**

![](https://cloudup.com/cSwYBXbHdfc+)

## How this works?

1. User logs in with Auth0 (any identity provider)
2. Auth0 returns a JSON Web Token to the browser
3. The browser calls Auth0 `/delegation` endpoint to validate the token and exchange it for AWS temporal credentials
  
  ```js  
  var aws_arns = { role: 'arn:aws:iam::account_number:role/role_name', principal: 'arn:aws:iam::account_number:saml-provider/provider_name' };
  auth0.getDelegationToken(aws_api_auth0_clientid, jwt, aws_arns, function(err, result) {
    var aws_credentials = result.Credentials; // AWS temp credentials
    // call AWS API e.g. bucket.getObject(...)
  });
  ```
  
4. With the AWS credentials you can now call the AWS APIs and have policies using ${saml:sub} to create policies to authorize users to access a bucket, rows or columns in DynamoDB, sending an email, etc.

## FAQ

### How is this secure if it's all client side?

The key is: when the user log in, we obtain an AWS session token. With this token and the IAM policy using `{saml:sub}` we can constraint the resources the client can access.

### What else can I do with AWS IAM policies?

Almost every API in AWS support IAM Policies. For instance, you can create a policy in DynamoDB to do fine grained access control as [shown here](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/FGAC_DDB.html)

* [AWS Products that support IAM policies](http://docs.aws.amazon.com/IAM/latest/UserGuide/Using_SpecificProducts.html)

### What IAM policies this sample uses?

A role needs to be created in the IAM console containing these two statements. These statements allow everything on a specific bucket path and allow listing the bucket objects with a prefix condition based on user id. This effectively creates a secure compartiment in S3 for each user.

```
{
  "Version": "2012-10-17",
  "Statement": [{
      "Sid": "AllowEverythingOnSpecificUserPath",
      "Effect": "Allow",
      "Action": [
        "*"
      ],
      "Resource": [ 
          "arn:aws:s3:::YOUR_BUCKET/dropboxclone/${saml:sub}",
          "arn:aws:s3:::YOUR_BUCKET/dropboxclone/${saml:sub}/*"]
   },
   {
      "Sid": "AllowListBucketIfSpecificPrefixIsIncludedInRequest",
      "Action": ["s3:ListBucket"],
      "Effect": "Allow",
      "Resource": ["arn:aws:s3:::YOUR_BUCKET"],
      "Condition":{ 
        "StringEquals": { "s3:prefix":["dropboxclone/${saml:sub}"] }
       }
    }
  ]
}
```

The `${saml:sub}` variable in the policy is replaced in runtime by AWS with the user id. For example, if a user logged in with any of the identity providers supported by Auth0, and its `user_id` is `ad|3456783129`, then this policy will allow any action to the S3 resource with this path `YOUR_BUCKET/dropboxclone/ad|3456783129`. It will also allow executing `s3:ListBucket` over the bucket with the prefix `dropboxclone/ad|3456783129`.

## Running it locally

Install `serve` 
    
    npm install -g serve

Run it on port 1338

    serve -p 1338

And point your browser to <http://localhost:1338>

## Running CLI Tools

export AZURE_STORAGE_ACCOUNT=foo
export AZURE_STORAGE_ACCESS_KEY=baz
export AUTH0_API_AUTH_TOKEN=
