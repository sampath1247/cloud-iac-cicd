# cloud-iac-cicd
Infrastructure as Code (CloudFormation) + GitHub Actions for a serverless app (API Gateway, Lambda, DynamoDB, S3, Amplify) with S3 event‑triggered front‑end deploys.
# Project 3 — IaC + CI/CD on AWS (CloudFormation, Lambda, API Gateway, S3, Amplify)

Course: Cyber Infrastructure & Cloud Computing

This repository contains:
- IaC (CloudFormation templates) to provision the backend (API + Lambda) and frontend infra
- Two Lambdas:
  - Backend Lambda (e.g., proj3_ProcessImageFunction)
  - Deployment Lambda (StartAmplifyDeployment) that starts an Amplify deployment when an S3 object (index.zip) is uploaded
- Frontend (index.html)
- Java SDK app (CloudFormationDemo.java) that:
  - Deploys backend, then frontend, then Amplify stacks (guided via console prompts)
  - Sets S3→Lambda event notifications and bucket policies
  - Uploads artifacts to S3 (e.g., index.zip)
- GitHub Actions CI/CD:
  - Zip and deploy backend Lambda on changes under iac/backend/**
  - Zip and deploy deployment Lambda on changes under iac/frontend/**
  - Zip index.html and upload to S3 on changes to frontend/index.html (triggers S3 event → Lambda → Amplify deploy)
  - Build the Java SDK app

## Prerequisites
- AWS account + IAM user/role with permissions for Lambda, S3, CloudFormation, Amplify, API Gateway, DynamoDB (least privilege preferred)
- Secrets configured in this repo:
  - AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_REGION
  - S3_BUCKET (e.g., proj3-uco-cicc-media-group6)
  - S3_PREFIX (e.g., proj3/)
  - BACKEND_LAMBDA_NAME (e.g., proj3_ProcessImageFunction)
  - FRONTEND_LAMBDA_NAME (e.g., StartAmplifyDeployment)

## Repository structure
See the tree in the root README section.

## Workflows (CI/CD)
- deploy-backend-lambda.yml
  - On push to main affecting iac/backend/**:
  - Zip backend lambda_function.py → proj3_lambda_function.zip
  - Upload zip to s3://$S3_BUCKET/$S3_PREFIX/
  - Update Lambda code from S3 key
- deploy-frontend-lambda.yml
  - On push to main affecting iac/frontend/**:
  - Zip proj3_deployment_lambda.py → proj3_deployment_lambda.zip
  - Upload zip to s3://$S3_BUCKET/$S3_PREFIX/
  - Update Lambda code from S3 key
- upload-frontend-zip.yml
  - On push to main affecting frontend/index.html:
  - Zip index.html → index.zip
  - Upload to s3://$S3_BUCKET/$S3_PREFIX/
  - S3 event triggers StartAmplifyDeployment to kick off Amplify build
- java-ci.yml
  - Build the Java app with Maven (`cicc-sdk-demo`), no AWS calls

## Java SDK app
- `cicc-sdk-demo/src/main/java/edu/uco/cicc/CloudFormationDemo.java`:
  - deployCloudFormationTemplate(region, url, stackName) → deploy backend first, then frontend, then Amplify (prompts you to confirm outputs are wired)
  - setEventNotification(accountId, region, bucketName, lambdaFunctionName, functionArn, amplifyAppId)
  - uploadS3File(bucket, key, localPath, region)
- Run locally:
```bash
cd cicc-sdk-demo
./mvnw -q -DskipTests package
java -cp target/cicc-sdk-demo-1.0-SNAPSHOT.jar edu.uco.cicc.Main
```
- You’ll be prompted to paste outputs (e.g., API URL) between steps.

## S3 event → Lambda → Amplify
- The deployment Lambda (StartAmplifyDeployment) is triggered when `index.zip` appears in s3://$S3_BUCKET/$S3_PREFIX/
- It starts an Amplify deployment (App ID/branch via env or injected by the Java setup)
- Your API Gateway CORS should be set in the backend template (OPTIONS + allowed headers/methods/origins)

## Cost & teardown
- This project is designed to stay within AWS Free Tier for typical student usage (Lambda, API Gateway, S3, IAM; CloudFormation is free)
- After demo: delete CloudFormation stacks to delete all resources; remove custom IAM policies/roles; empty/delete buckets

## Notes
- Do not commit secrets; use GitHub Actions Secrets
- Large files (e.g., Report.docx ~50MB): consider converting to PDF or Git LFS
- Keep bucket names globally unique and consistent across templates and code

## License
MIT (see LICENSE)
