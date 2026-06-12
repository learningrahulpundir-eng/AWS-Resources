# Host a React App with CloudFront

## Goal

This guide shows how to:
- create a React production build
- upload it to Amazon S3
- serve it as a static website
- use CloudFront for CDN delivery

---

## 1. Create a React App

If you do not already have a React app, run:

```sh
npm create vite@latest my-app
cd my-app
npm install
```

---

## 2. Build the App

Generate the production files:

```sh
npm run build
```

This creates a folder named `dist/` containing:
- `index.html`
- `static/`

---

## 3. Create an S3 Bucket

1. Open the AWS Console.
2. Go to S3.
3. Click Create bucket.
4. Use a unique bucket name, for example:
   - `my-react-app-demo-123`
5. Make sure the option "Block all public access" is unchecked.

---

## 4. Upload the Build Files

1. Open the bucket.
2. Click Upload.
3. Upload the contents of the `build/` folder.
4. Do not upload the folder itself; upload its files directly.

You should upload:
- `index.html`
- the `static/` folder

---

## 5. Allow Public Read Access

Go to:
- Permissions → Bucket Policy

Add this policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-react-app-demo-123/*"
    }
  ]
}
```

Save the policy.

---

## 6. Enable Static Website Hosting

Go to:
- Properties → Static website hosting

Set:
- Index document: `index.html`
- Error document: `index.html`

This is important for React Router and page refreshes.

After enabling it, copy the website URL. Your React app is now live.

---

## 7. Create a CloudFront Distribution

1. Open CloudFront.
2. Click Create distribution.
3. Set the origin to your S3 website endpoint:
   - `bucket-name.s3-website-region.amazonaws.com`
4. Use the following setting:
   - Viewer protocol policy: Redirect HTTP to HTTPS
5. Create the distribution.

It may take 5–10 minutes to deploy.

---

## 8. Access the App Through CloudFront

Once the distribution is ready, use the CloudFront URL:

```text
https://dxxxxx.cloudfront.net
```

Benefits:
- faster delivery worldwide
- cached at edge locations

---

## 9. Clear the Cache After Deployment

When you deploy a new version of the app:

1. Open CloudFront.
2. Go to Invalidations.
3. Create a new invalidation for:

```text
/*
```

This clears old cached files.

---

## 10. Clean Up

To avoid charges:
- delete the CloudFront distribution
- delete the S3 bucket

