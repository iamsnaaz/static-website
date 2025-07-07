Architecture Summary

GitHub Repo (index.html, error.html)
        ↓
  CodePipeline (with CodeBuild)
        ↓
  S3 Bucket (Static Website Hosting Enabled)
        ↓
CloudFront (Caches Content)

1. GitHub Repo Structure

/my-static-site/
├── index.html
├── error.html
├── buildspec.yml

Ensure the CodeBuild role has permissions:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "cloudfront:CreateInvalidation"
      ],
      "Resource": "*"
    }

    
├── error.html
├── buildspec.yml
📄 2. buildspec.yml
yaml
Copy
Edit
version: 0.2

phases:
  install:
    runtime-versions:
      nodejs: 18
  build:
    commands:
      - echo "Starting deployment..."
  post_build:
    commands:
      - aws s3 cp index.html s3://YOUR_BUCKET_NAME/
      - aws s3 cp error.html s3://YOUR_BUCKET_NAME/
      - aws cloudfront create-invalidation --distribution-id YOUR_DIST_ID --paths "/*"
artifacts:
  files:
    - index.html
    - error.html
Replace:

YOUR_BUCKET_NAME → your actual S3 bucket name

YOUR_DIST_ID → your CloudFront Distribution ID

🔐 3. IAM Role for CodeBuild
Ensure the CodeBuild role has permissions:

json
Copy
Edit
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "cloudfront:CreateInvalidation"
      ],
      "Resource": "*"
    }
  ]
}
🛠️ 4. CodePipeline Setup
🔹 Step-by-Step in Console:
Step 1: Source Stage
Source provider: GitHub

Select your repo and branch (e.g., main)

Output artifact: SourceArtifact

Step 2: Build Stage
Build provider: CodeBuild

Create new project:

Use buildspec.yml

Environment: Linux, standard, Node.js runtime

Input: SourceArtifact

Step 3: Deploy Stage
❌ Skip — the deploy is already handled inside CodeBuild

✅ 5. S3 Bucket Settings
Enable static website hosting

Index document: index.html

Error document: error.html

Make the bucket public or use CloudFront with OAC

Ensure correct bucket policy (ask if you need help)

✅ 6. CloudFront Setup
Origin Domain: your-bucket.s3-website-<region>.amazonaws.com

Origin Protocol Policy: HTTP only

Default Root Object: index.html

Custom Error Responses (optional):

403/404 → /error.html or /index.html if SPA
