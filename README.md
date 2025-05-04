# DevOps Pipeline with Google Cloud

This repository demonstrates building a continuous integration pipeline using Google Cloud services including Cloud Source Repositories, Cloud Build, build triggers, and Artifact Registry.

## Project Overview

The project implements a simple Python Flask web application that displays a greeting message. This application is containerized using Docker and deployed using Google Cloud's CI/CD pipeline.

## Prerequisites

- A Google Cloud account
- Basic knowledge of Git
- Understanding of Python and Docker

## Video Demonstration

I've created a video showing the full process of myself setting up and testing this CI/CD pipeline. The video demonstrates all the steps above and shows the application running on a Compute Engine VM.

https://www.youtube.com/watch?v=z2ZDLsc6TgI

## Project Setup Steps

### 1. Create a Git Repository

First, I created a Git repository in Cloud Source Repositories:
```bash
mkdir gcp-course
cd gcp-course
gcloud source repos clone devops-repo
cd devops-repo
```

### 2. Create a Simple Python Flask Application

The application consists of:
- `main.py`: Contains the Flask application code
- `templates/`: Directory with HTML templates
  - `layout.html`: Base template with common HTML structure
  - `index.html`: Template for the homepage
- `requirements.txt`: Lists Python dependencies

### 3. Define a Docker Build

Created a Dockerfile with the following content:
```dockerfile
FROM python:3.9
WORKDIR /app
COPY . .
RUN pip install gunicorn
RUN pip install -r requirements.txt
ENV PORT=80
CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 main:app
```

### 4. Manage Docker Images with Cloud Build and Artifact Registry

I created an Artifact Registry repository and built the Docker image:
```bash
# Create Artifact Registry repository
gcloud artifacts repositories create devops-repo \
    --repository-format=docker \
    --location="REGION"

# Configure Docker authentication
gcloud auth configure-docker "REGION"-docker.pkg.dev

# Build and push the image
gcloud builds submit --tag "REGION"-docker.pkg.dev/$PROJECT_ID/devops-repo/devops-image:v0.1 .
```

### 5. Automate Builds with Triggers

Set up a Cloud Build trigger to automatically build the Docker image when changes are pushed to the repository.

The `cloudbuild.yaml` file contains:
```yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'REGION-docker.pkg.dev/PROJECT_ID/devops-repo/devops-image:$COMMIT_SHA', '.']
images:
  - 'REGION-docker.pkg.dev/PROJECT_ID/devops-repo/devops-image:$COMMIT_SHA'
options:
  logging: CLOUD_LOGGING_ONLY
```

### 6. Test the Build Process

I tested the CI/CD pipeline by:
1. Making a change to the application code
2. Committing and pushing the change to the repository
3. Verifying that the trigger automatically built a new image
4. Deploying the new image to a Compute Engine VM to test the changes


## File Structure

```
├── main.py                 # Flask application
├── Dockerfile              # Docker configuration
├── requirements.txt        # Python dependencies
├── templates/              # HTML templates
│   ├── layout.html         # Base template
│   └── index.html          # Homepage template
└── cloudbuild.yaml         # Cloud Build configuration
```

## Notes

- Replace "REGION" with your Google Cloud region (e.g., us-central1)
- Replace "PROJECT_ID" with your Google Cloud project ID

## Learning Outcomes

Through this project, I learned:
- How to set up a Git repository in Cloud Source Repositories
- How to create a simple Flask application
- How to containerize an application with Docker
- How to use Cloud Build to build Docker images
- How to store Docker images in Artifact Registry
- How to set up automatic build triggers in Cloud Build
- How to deploy Docker containers to Compute Engine
