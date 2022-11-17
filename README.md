# S3_Static_Website_Hosting
Deploy static sites on Amazon
--------------------------------------------
<img width="1440" alt="Screen Shot 2022-11-17 at 12 49 31 PM" src="https://user-images.githubusercontent.com/104800728/202520500-2f9c26c6-b7de-4ae0-a084-7aadb5257f7a.png">
S3 Bucket Setup
===============

Create a bucket 
---------------

Bucket names must conform with DNS requirements:

* **Should not contain uppercase characters**
* Should not contain underscores
* Should be between 3 and 63 characters long
* Should not end with a dash
* Cannot contain two, adjacent periods
* Cannot contain dashes next to periods (e.g., "my-.bucket.com" and "my.-bucket" are invalid)


Configure the Bucket Static Website Hosting
-------------------------------------------

Once the bucket is created, select it and choose `Properties > Static Website Hosting`. 

Choose proper values for the `Index Document` and `Error Document` fields (i.e. **index.html** and **404.html**). 

The Index Document is searched for relative to the requested folder: [http://my.aws.site.com/some/subfolder/]() **becomes** [http://my.aws.site.com/some/subfolder/**index.html**]().
	
The Error Document path is always relative to the root of the site. All errors are redirected to [http://my.aws.site.com/**404.html**](). 


Configure a Public Readable Policy for the Bucket
-------------------------------------------------

Static sites hosted on S3 do not support private files (password protection, etc). You must make all files publicly accessible. From your buckets `Properties` page, choose `Permissions > Edit/Add bucket policy`. Copy and past the policy below (replace **YOUR-BUCKET-NAME** for the bucketName you created previously)

```json
{
	"Version": "2008-10-17",
	"Statement": [
		{
			"Sid": "PublicReadForGetBucketObjects",
			"Effect": "Allow",
			"Principal": {
				"AWS": "*"
			},
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
		}
	]
}
```

S3 User Setup
=============

Log into your [AWS Console](https://console.aws.amazon.com/iam/?#users) and go to the [Users](https://console.aws.amazon.com/iam/?#users) management console. Click the `Create New Users` button and enter a username. 

Credentials File
----------------

Have AWS create a new key pair for the user and copy the contents into a `aws-credentials.json` file in the root directory of your project. You should add this file to `.gitignore` (or similar) so that credentials are not checked into version control.

```json
{ 
	"accessKeyId": "PUBLIC_KEY", 
	"secretAccessKey": "SECRET_KEY", 
	"region": "us-west-2" 
}
```

**note**: As AWS SDK's documentation points out, you could also set those as `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables 

User Permissions
----------------

From the [AWS IAM Users Console](https://console.aws.amazon.com/iam/?#users) select the newly created user, then the `Permissions` Tab, then click the `Attach User Policy` button.  Paste in the following (substituting BUCKET-NAME as appropriate).

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "s3:DeleteObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Sid": "AllowNewUserAccessToMyBucket",
      "Resource": [
        "arn:aws:s3:::BUCKET-NAME",
		"arn:aws:s3:::BUCKET-NAME/*"
      ],
      "Effect": "Allow"
    }
  ]
}
```
Upload!
=======

Simply call `s3-upload` from the same directory as your config file, and the upload will happen.
