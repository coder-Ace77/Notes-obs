
---

### AWS S3:

AWS s3 provides a way to store data on internet. These are very realiable.

One important setting is to deploy a website. Which can be easily done in AWS.
This feature is called static website hosting and can be done by following the given steps.

S3: 

1. npm run build 

Step 2: Create an S3 Bucket 

1. Go to AWS Management Console > S3. 
2. Click Create bucket. 
3. Enter a unique bucket name (e.g., my-react-app-bucket). 
4. Choose your preferred region. 
5. For hosting a website, uncheck the "Block all public access" option (you will allow public read access for the website). 
6. Acknowledge the warning about public access. 
7. Create the bucket. 

Step 3: Configure Bucket for Static Website Hosting 

1. Open your bucket. 
2. Go to the Properties tab.
3. Scroll to Static website hosting. 
4. Select Enable. 
5. Choose Host a static website. 
6. Set the Index document to index.html. 
7. Set the Error document to index.html (important for React Router to handle routing). 
8. Save changes. 

Step 4: Set Bucket Policy for Public Read Access 

1. Go to the Permissions tab. 
2. Click Bucket Policy. 
3. Add this policy (replace YOUR_BUCKET_NAME): 

```json
{ 
  "Version": "2012-10-17", 
  "Statement": [ 
    { 
      "Sid": "PublicReadGetObject", 
      "Effect": "Allow", 
      "Principal": "*", 
      "Action": "s3:GetObject", 
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*" 
    } 
  ] 
}
```

