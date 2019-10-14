<!-- <style>
img[src*="border"] {
border: solid 2px black;
}
</style> -->

# AWS S3 image storage in Vanilla Javascript

We will be going over the steps of how to create a bucket in S3 using plain Javascript and connecting it to your Node App.

> NOTE: Words in ALL_CAPS are words that are meant to be replaced

### Here are the steps!

1. [Install AWS CLI](#installAWS)
2. [Create AWS Account](#createAccount)
3. [Create an S3 Bucket](#createBucket)
4. [Create an IAM Group](#makeGroup)
5. [Create an IAM User](#makeUser)
6. [Make a Cognito Pool id](#makeCognitoRole)
7. [Get Unautherized Role arn](#getARN)
8. [Input Bucket Policy](#inputBucketPolicy)
9. [Setting up my local environment](#environmentSetup)
10. [Create your project](#createProject)

## Install the AWS CLI {#installAWS}

- First step is to Install [aws cli](https://docs.aws.amazon.com/cli/latest/userguide/install-macos.html#awscli-install-osx-pip)
- After cli is installed, run command `aws` in your terminal and it should output:
  ![](images/successful_AWS_Terminal.png)
  - If you got a message that says the command doesn't exist, try and redo the steps to install aws cli
    > If you want to know more about the aws command, you can look up the [documentation](https://docs.aws.amazon.com/sdk-for-javascript/index.html) or in a windows type **help aws**, mac type **man aws** for a manual on the parameters and how to use it. [^1]

---

## Create an AWS Account {#createAccount}

- Head over to the [aws website](https://aws.amazon.com/) and create an account
  ![](images/create_AWS_Account.png)

---

## Create a Bucket {#createBucket}

- In the search bar, type in S3 and navigate to it
  ![](images/search_Bar_S3.png)
- Create a bucket with a meaningful name **AND** put the bucket region to _US West (Oregon)_
  ![](images/bucket_AWS_Region.png)
  > NOTE: This is the closest working region. You will be referencing this region later with **us-west-2**
- Go ahead and click next on the rest of the sections and create the bucket
- We are done on this section for now!

---

## Create a Group in IAM {#makeGroup}

- Navigate to your [AWS Management Console](https://aws.amazon.com/console/home?region=us-west-2)
- In the search bar, type in **IAM** and click on the link
  > If you want to know more about IAM, complete [documentation is here](https://docs.aws.amazon.com/iam/)
- As [best practice](https://docs.aws.amazon.com/AmazonS3/latest/dev/security-best-practices.html), we will be making a group and setting policies/rules that are attached to whatever user we put in that group
- On the left pane, click the `Groups` link
  ![](images/iam_User_Create_Groups.png)
- Click `Create new Group`
- Input meaningful group name and hit next
- On the _Attach Permissions_ step, type in S3 in the search bar and check the box that says `AmazonS3FullAccess`
  > Note: We'll have to review these policies in the future so the user doesn't have too much access
- Click next and create the group!

---

## Create a user {#makeUser}

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
  > Note: This step is SUPER important because the Secret Access key and Access Key ID will not appear again after you close this page.
- After you download, you can click the close button
- You should now be able to see the user you created and the group that the user is a part of
  ![](images/successful_User_Creation.png#border)

---

## Make a Cognito Role {#makeCognitoRole}

- Navigate to _Services_ drop down on the top of your nav bar and type **Cognito** in the search bar and click it [^2]
  ![](images/services_Search.png)
- On this page you have two options. Go ahead and click on the _Manage Identity Pools_ button
  ![](images/manage_Identity_Pool_Button.png)
- Next click on the _Create New Identity Pool_ button
  ![](images/create_New_Identity_Pool_Button.png)
- Input a meaningful identity pool name and check the `Enable access to unauthenticated identities` check box then click _Create Pool_ button
  ![](images/create_New_Identity_Pool.png)
- Make sure to read the differences between authenticated [^3] and unauthenticated [^4] users on the following page. Check also how to [grant least privileges](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#grant-least-privilege) to users.
- Click `Allow`
  ![](images/create_Unauthenticated_user.png)
- There is a bit of things to pay special attention to on this page. Make sure your platform is set to javascript and copy your identity pool id.
  ![](images/finished_Cognito_Identity_Pool.png)
  > Note: Don't freak out if you closed this window. You can get the `pool credentials` by clicking on **services** drop down on your navbar and searching **cognito** and pressing the `manage identity pool` button and clicking on the pool id you created.

---

## Get Unautherized Cognito Role ARN {#getARN}

- Navigate to your IAM from the Services drop down menu in your navbar
  ![](images/navigate_To_IAM.png)
- Navigate to Roles on the left pane and you should see the cognito role you created. Both an unautherized and autherized version. Click on the Unautherized role name.
  ![](images/unautherized_Role.png)
- You should be able to see the ARN now. Go ahead and copy and save that arn. You'll be using it in the next step.
  ![](images/copy_ARN.png)

---

## Add policies to your bucket {#inputBucketPolicy}

- Navigate to the bucket you created for this project
  ![](images/search_Bar_S3.png)
- Click the bucket name you created
  ![](images/click_Bucket_Name.png)
- Navigate to `Permissions` then `Bucket Policy` and input JSON

  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
          "AWS": [
            "YO_UNAUTHERIZED_POOL_ID_ROLE_ARN",
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

  ![](images/bucket_Policy.png)
  replace names in caps with your bucket name and the unautherized cognito role arn you created then click save

- On the same bucket, navigate to `CORS Configuration` and input below code to enable CORS and then click the save button on the right of your screen

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

  ![](images/enable_CORS.png)

---

## Setting up Unix and Windows environment {#environmentSetup}

- Start your Terminal/cmd
- run the command 'aws configure' in your terminal/cmd and
  input the **Access ID**, **Secret Key**, region name: **us-west-2**, and output format: **JSON**
- This will create a file `.aws` in your home directory with `credentials` and `config` file created inside
- **Follow [these steps](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html) to set up your credentials if my process has lost you**
- You need to put anything that has **export** in the `~/.bash_profile` for your Mac or your environment variables on Windows. If your Mac doesn't have a `~/.bash_profile`, create this file with the command `touch ~/.bash_profile`
  For Mac ~/.bash_profile

  ```bash
  export AWS_ACCESS_KEY_ID=((your access key id goes here))
  export AWS_SECRET_ACCESS_KEY=((your secret access key))
  ```

  For Windows Environment Variables

  ```shell
  set AWS_ACCESS_KEY_ID=((your access key id goes here))
  set AWS_SECRET_ACCESS_KEY=((your secret access key))
  ```

- Then, type in `source ~/.bashrc` and this should set your environment variables

---

## Create a Project in your editor {#createProject}

- Create a Project Directory and in the project terminal, input `npm init -y`
- Now that you have that set up, you'll need to install aws-sdk with `npm i aws-sdk && npm i uuid` in your project directory
- refer to [aws-sdk site](https://www.npmjs.com/package/aws-sdk) or [uuid site](https://www.npmjs.com/package/uuid) if you need help knowing what the packages do
- Create a Project Directory with an html file that has the following example code.

  ```html
  <!DOCTYPE html>
  <html>
    <head>
      <script src="https://sdk.amazonaws.com/js/aws-sdk-2.283.1.min.js"></script>
      <script src="./pushimage.js"></script>
    </head>
    <body>
      <h1>My Photo Albums App</h1>
      <div id="app">
        <input type="file" id="photoupload" />
        <input type="submit" onclick="addPhoto('BUCKET_NAME')" />
      </div>
    </body>
  </html>
  ```

- Also, create a js file that has the following example code:

  ```js
  const albumBucketName = "BUCKET_NAME";
  const bucketRegion = "BUCKET_REGION";
  const IdentityPoolId = "UNAUTHERIZED_COGNITO_ROLE_ID";

  AWS.config.update({
    region: bucketRegion,
    credentials: new AWS.CognitoIdentityCredentials({
      IdentityPoolId
    })
  });

  const s3 = new AWS.S3({
    apiVersion: "2006-03-01",
    params: { Bucket: albumBucketName }
  });

  function addPhoto(albumName) {
    const { files } = document.getElementById("photoupload");
    if (!files.length) {
      return alert("Please choose a file to upload first.");
    }
    const file = files[0];
    const fileName = file.name;
    const albumPhotosKey = `${encodeURIComponent(albumName)}//`;
    console.log(albumPhotosKey);

    const photoKey = albumPhotosKey + fileName;

    // Use S3 ManagedUpload class as it supports multipart uploads
    const upload = new AWS.S3.ManagedUpload({
      params: {
        Bucket: albumBucketName,
        Key: photoKey,
        Body: file,
        ACL: "public-read"
      }
    });

    const promise = upload.promise();

    promise
      .then(data => {
        alert("Successfully uploaded photo.");
        // viewAlbum(albumName);
      })
      .catch(err => {
        return alert("There was an error uploading your photo: ", err.message);
      });
  }
  ```

- Go ahead and run the code with `npm run filename`
- If it pushes up then you should be all set-up! Happy hacking!

- refer to this [link](https://docs.aws.amazon.com/code-samples/latest/catalog/javascript-s3-s3_upload.js.html) if you want to see some code examples

[^1]: Here is a [link](https://www.lemoda.net/windows/windows2unix/windows2unix.html) to Mac Terminal and Windows equivalent if you need to switch between both
[^2]: Check out this [link](https://aws.amazon.com/premiumsupport/knowledge-center/cognito-user-pools-identity-pools/) if you want to deep dive and know the difference between **User Pools** and **Identity Pools**
[^3]: Authenticated users
[^4]: Unauthenticated users
