# AWS CloudFront

## What is CloudFront?

AWS CloudFront is a Content Delivery Network (CDN) service that delivers websites, images, videos, and APIs from edge locations closest to users.

### Simple idea

Instead of serving all requests from the original server, CloudFront caches content at edge locations around the world. This makes content load faster and reduces pressure on the origin server.

---

## How CloudFront works

1. A user opens your website.
2. The request goes to the nearest CloudFront edge location.
3. If the file is already cached, CloudFront serves it immediately.
4. If not, CloudFront fetches it from the origin (such as S3, EC2, or a load balancer).
5. The content is cached for future requests.

---

## Key components

### 1. Edge locations
- Servers placed globally
- Deliver content with low latency
- Improve speed for users in different regions

### 2. Origin
The origin is where your original content lives, for example:
- S3 bucket
- EC2 instance
- Load balancer
- External web server

### 3. Distribution
A CloudFront distribution is the configuration that connects your origin to the CloudFront CDN.
It gives you a public URL such as:

https://d123abc.cloudfront.net

---

## Main features

- Fast performance
- Caching for repeated requests
- HTTPS support
- DDoS protection through AWS services
- Signed URLs and access control
- Global delivery network

---

## When to use CloudFront

Use it when:
- You have users in many countries
- You need fast page loading
- You serve static or dynamic content
- You stream videos or deliver images

It may not be necessary for:
- Small local applications
- Internal-only systems

---

## Without CloudFront vs With CloudFront

### Without CloudFront
User (India) → Server (US)
- Slower response time

### With CloudFront
User → Nearest edge location
- Faster response time

---

## Common use cases

- Static website hosting with S3
- Video streaming
- Image delivery
- API acceleration
- Web application delivery

---

## Simple pricing idea

You usually pay for:
- Data transfer
- Number of requests

AWS also offers a free tier for some usage.

---

## Demo: Host a static website with S3 and optional CloudFront

### Goal
- Host a static website using S3
- Optionally improve performance using CloudFront

---

## Part 1: Host a website using S3

### Step 1: Create an S3 bucket
1. Open the AWS Console.
2. Search for S3.
3. Click Create bucket.
4. Enter a bucket name, for example:
   - my-static-website-demo-123
5. Choose a region.
6. Click Create bucket.

### Step 2: Disable Block Public Access
1. Open the bucket.
2. Go to Permissions.
3. Click Block public access.
4. Click Edit.
5. Uncheck Block all public access.
6. Save the changes.

### Step 3: Upload your website file
Create an HTML file named index.html:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My Website</title>
  </head>
  <body>
    <h1>Welcome 🚀</h1>
    <p>This is my static website hosted on AWS S3.</p>
  </body>
</html>
```

Upload the file to the bucket.

### Step 4: Add a bucket policy
Go to Permissions → Bucket policy and add the following policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-static-website-demo-123/*"
    }
  ]
}
```

Save the policy.

### Step 5: Enable static website hosting
1. Go to Properties.
2. Open Static website hosting.
3. Click Edit.
4. Enable the setting.
5. Set Index document to index.html.
6. Save the changes.

### Step 6: Open your website
Copy the S3 website URL, for example:

http://my-static-website-demo-123.s3-website-ap-south-1.amazonaws.com

Open it in your browser. Your website should load.

---

## Part 2: Optional CloudFront setup

### Step 7: Create a CloudFront distribution
1. Open CloudFront.
2. Click Create distribution.
3. Set the origin to your S3 website endpoint.
4. Choose Viewer protocol:
   - Redirect HTTP to HTTPS
5. Click Create.

Wait 5–10 minutes for the distribution to deploy.

### Step 8: Use the CloudFront URL
After deployment, copy the CloudFront domain, for example:

https://d123abc.cloudfront.net

Open it in the browser. Your website is now served through CloudFront.

### Step 9: Clear cache if needed
If your updated content does not appear:
1. Open CloudFront.
2. Go to Invalidations.
3. Create a new invalidation.

---

## Final summary

The basic flow is:
1. Create an S3 bucket
2. Upload your website files
3. Enable public access
4. Enable static website hosting
5. Access the website
6. Optionally add CloudFront for better performance

