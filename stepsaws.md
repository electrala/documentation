### Steps of reproduction

- First step is to Install [aws cli](https://docs.aws.amazon.com/cli/latest/userguide/install-macos.html#awscli-install-osx-pip)
- After cli is installed, run command `aws` in your terminal and it should output:
  ![](images/successful_AWS_Terminal.png)
  - If you got a message that says the command doesn't exist, try and redo the steps to install aws cli
- Head over to the [aws website](https://aws.amazon.com/) and create a user
  ![](images/create_AWS_Account.png)
- In the search bar, type in S3 and navigate to it
  ![](images/search_Bar_S3.png)
- Create a bucket with a meaningful name **AND** put the bucket region to _US West (Oregon)_
  ![](images/bucket_AWS_Region.png)
  > NOTE: you will be referencing this region later with **us-west-2**
- Go ahead and click next on the rest of the sections and create the bucket
- We are done on this section for now!

---

- Click on the name of the bucket you just made
  ![](images/click_Bucket_Name.png)

---

- Navigate to your [AWS Management Console](https://aws.amazon.com/console/home?region=us-west-2)
- In the search bar, type in **IAM** and click on the link
- As good practice, we will be making a group and setting policies/rules that are attached to whatever user we put in that group
- On the left pane, click the `Groups` link
  ![](images/iam_User_Create_Groups.png)
- Click `Create new Group`
- Input meaningful group name and hit next
- On the _Attach Permissions_ step, type in S3 in the search bar and check the box that says `AmazonS3FullAccess`
  > Note: We'll have to review these policies in the future so the user doesn't have too much access
- Click next and create the group!
- Now that the group is created, it's time to make a user
- On the left pane, click the `Users` link
  ![](images/iam_User_Create_Pane.png)
- Click the blue `Add User` button to begin creating a user
- Input a meaningful user name and check the box for `Programmatic access` then click next
  ![](images/add_Username_Role.png)
- The group you created should show up, go ahead and check the box and hit next
  ![](images/set_Permissions_User.png)
- Use the default values for the next steps and create the User but DO NOT close the tab yet
- On **Step 5**, after creating the user, make sure you copy or download your Access Key and Secret Key
  ![](images/download_AWS_User_Keys.png)
  > Note: This step is SUPER important because the Secret Access key and Access Key ID will not appear again after you close the page.
- After download, you can click the close button
- You should now be able to see the user you created and the group that user is a part of
  ![](images/successful_User_Creation.png)

---

- On the pane to the left, navigate to `Roles` link and click the button _Create Role_
- On the Create Role page, click `Web Identity` so we can create a Cognito Role. Identity provider will be [Amazon Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html) and input a meaningful identity pool id and click next
  ![](images/create_Cognito_Role.png)
- Click `Create Policy` button on the top left
- Navigate to the JSON app and replace the json with the below JSON and click next

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:DeleteObject", "s3:GetObject", "s3:PutObject"],
      "Resource": ["arn:aws:s3:::YO_BUCKET_NAME/*"]
    }
  ]
}
```

- In the `Review Policy` page, choose a meaningful name and click create
- On your bucket policy, input json

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "YO_POOLID_ROLE_ARN",
          "arn:aws:iam::YO_12_DIGIT_NUMBER_IN_ARN:root"
        ]
      },
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::YO_BUCKET_NAME",
        "arn:aws:s3:::YO_BUCKET_NAME/*"
      ]
    }
  ]
}
```

- On the same bucket, enable cors with

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <CORSRule>
    <AllowedOrigin>*</AllowedOrigin>
    <AllowedMethod>POST</AllowedMethod>
    <AllowedMethod>GET</AllowedMethod>
    <AllowedMethod>PUT</AllowedMethod>
    <AllowedMethod>DELETE</AllowedMethod>
    <AllowedMethod>HEAD</AllowedMethod>
    <AllowedHeader>*</AllowedHeader>
  </CORSRule>
</CORSConfiguration>
```

## Some intermediate needed steps include Creating a cognito role and setting permissions on the bucket and cognito role....

### Bucket policies

### We may not need to create an iam role if we are creating an identity pool id

- run 'aws configure' in your terminal and
  input the secret key and access id
- This will _ask_ you for your credentials (keys) - steps: - This will create a file `.aws` in your home directory with these files inside
- ~/.aws/credentials - ~/.aws/config
  ///// Go through the steps for aws configure!
- **follow these [steps](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html) to set up your credentials if lost**
  - You need to put anything that has **export** in the `~/.bash_profile` don't freak out if you don't have one, you can create this file with the command `touch ~/.bash_profile`
- If on **Mac** put these environment variables in your ~/.bashrc

```bash
export AWS_ACCESS_KEY_ID=((your access key id goes here))
export AWS_SECRET_ACCESS_KEY=((your secret access key))
```

- If on **Windows** put these environment variables in your ~/.bashrc

```bash
set AWS_ACCESS_KEY_ID=((your access key id goes here))
set AWS_SECRET_ACCESS_KEY=((your secret access key))
```

- Then, type in `source ~/.bashrc` and this should set your environment variables
- Don't forget about `npm init -y` just in case you don't have your package.json :)
- Now that you have that set up, you'll need to install aws-sdk with `npm i aws-sdk && npm i uuid`
  - refer to [aws-sdk site](https://www.npmjs.com/package/aws-sdk) or [uuid site](https://www.npmjs.com/package/uuid) if you need help knowing what it does

The following code should output your keys. if your output has your keys, then you have it set up correctly so far.

```js
var AWS = require("aws-sdk");

AWS.config.getCredentials(function(err) {
  if (err) console.log(err.stack);
  // credentials not loaded
  else {
    console.log("Access key:", AWS.config.credentials.accessKeyId);
    console.log("Secret access key:", AWS.config.credentials.secretAccessKey);
  }
});
```

- refer to this [link](https://docs.aws.amazon.com/code-samples/latest/catalog/javascript-s3-s3_upload.js.html) if you want to see some code examples
