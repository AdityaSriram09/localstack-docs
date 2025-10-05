# Introduction

In this tutorial, you’ll learn how to build a serverless image resizing pipeline using S3 + Lambda on LocalStack, with hot reload support so you can iterate quickly without full redeploys.

The architecture is:

Upload an image to an input S3 bucket.

A Lambda, triggered by the S3 event, resizes the image and writes it to an output S3 bucket.

You can modify the Lambda code locally and, via hot reload support, changes will reflect immediately (no full redeploy).

This pattern is useful for generating thumbnails, avatars, image processing pipelines, etc.

# Prerequisites

Before you begin, ensure you have the following:

* LocalStack Pro (hot reload features require the Pro edition)

* Docker (for running LocalStack)

* Node.js and/or Python (depending on the runtime in your repo)

* AWS CLI or awslocal (for deploying / interacting with LocalStack)

* make or shell tooling (optional but helpful)

* Basic familiarity with AWS Lambda, S3, event-driven architectures

# Architecture Diagram

Below is a simplified flow:
```
Client → upload image → S3 (input bucket)
         ↓ (S3 trigger)
     Lambda (resizer)
         ↓
    S3 (output/resized bucket)

```
You can also incorporate error handling (SNS, alerts) or a UI layer (presigned upload) depending on your setup.

# Steps
## 1. Clone the repository
```
git clone https://github.com/localstack-samples/sample-lambda-s3-image-resizer-hot-reload.git  
cd sample-lambda-s3-image-resizer-hot-reload  
```
## 2. Install dependencies

If Python is used:
```
python -m venv .venv  
source .venv/bin/activate  
pip install -r requirements-dev.txt  
```

If there are Node.js parts:
```
cd <node-function-folder>
npm install
```
## 3. Start LocalStack

Make sure your LOCALSTACK_AUTH_TOKEN is set. Then:
```
localstack auth set-token <your-token>
localstack start
```
## 4. Build and Deploy Lambdas
```
deployment/build-lambdas.sh  
deployment/awslocal/deploy.sh  
```

These scripts will package your lambda code and deploy to LocalStack (creating buckets, setting up triggers, etc.). After deployment, note the Lambda function URLs or ARNs printed by the script.

## 5. Configure the Web UI (if present)

If the sample repo includes a web UI, open it (e.g. via S3 static website endpoint in LocalStack). Configure it by inputting the Lambda URLs / endpoints that you obtained above.

## 6. Upload a Sample Image

Use the UI or awslocal + AWS CLI to upload an image (e.g. test.jpg) into the input bucket. This should trigger the resize Lambda, and produce a resized version in the output bucket.

## 7. Enable Hot Reload

To enable hot reload on a Lambda, you’ll use a special “hot-reload” S3 bucket that points to your local function code. Example command:
```
awslocal lambda update-function-code --function-name <FunctionName> \
  --s3-bucket hot-reload --s3-key "$(pwd)/lambdas/<function-folder>"
```

Now, when you change the handler code in your local folder (e.g. handler.py), LocalStack will detect changes and refresh the running function without a full redeploy.

# Testing the Application
## 1. Initial Upload Test

* Upload an image (e.g. sample.jpg)

* Wait for Lambda to process (LocalStack handles invocation)

* Download the resized image from the output bucket

* Verify that its dimensions match your expectations (e.g. width/height constraints)

## 2. Hot Reload Test

* Edit your Lambda code (for example, change target size or add a watermark)

* Save the changes

* Re-upload a new image (or same filename)

* Download from the output bucket again

* Confirm that your updated logic is applied (new resized size or watermark)

## 3. Automated Integration Tests

If the repo includes a test suite (e.g. via pytest):
```
pytest tests/
```

These tests should cover end-to-end behavior:

* Uploading test images

* Verifying output in the target bucket

* Edge cases (non-image uploads, errors, alerts)

* Add assertions for new behavior after hot reload, etc.

# Conclusion

In this tutorial, you’ve built a serverless image processing pipeline on LocalStack:

* Event-driven flow: S3 trigger → Lambda → output bucket

* Hot reload support for fast development iteration

* A foundation you can extend (e.g. add watermarking, format conversion, error alerts)

This pattern is applicable to many real-world use cases—thumbnails, image pipelines, and other serverless media workflows.
