---
title: "[DRAFT] Utilize GitLab Pipeline to Automate Workflow"
# description: "How I passed the AWS Developer - Associate Certification Exam (DVA-C02)"
dateString: July 2024
draft: false
tags: ["GitLab", "pipeline", "GCP"]
weight: 102
cover:
    image: "/blog/gitlab-pipeline-infra/cover.png"
---

## Introduction

As a platform team, our goal is to empower developers with easy-to-use tools for creating infrastructure. Our Internal Developer Portal offers a suite of tools to enhance developers' daily workflows. In this article, .........................

## Background

- Our cloud provider is Google Cloud Platform (GCP)
- We use Terraform for infrastructure configuration
- This article focuses on VM-based applications

While we won't dive deep into the Terraform details in this article, it's important to note that we create infrastructure by defining Terraform resources.

## Base Infrastructure Components

For each application, we define the following base infrastructure:

1. Service Account
2. Instance Template
3. Instance Group
4. Backend Service
5. URL Map

## The Onboarding Process

### Step 1: User Input

Users begin by filling out a form in our Developer Portal, providing:

- Application name
- Instance template configurations:
  - Machine type
  - Disk size
  - Ubuntu image version
- Instance group configurations:
  - Maximum replicas
  - Minimum replicas

### Step 2: Pipeline Trigger

Upon form submission, we trigger a GitLab pipeline using GitLab's trigger API. This process involves the following:

#### Prerequisites

Before triggering the pipeline, we need to generate a trigger token. This can be done using the GitLab API for creating trigger token. For more information, refer to the [GitLab documentation on creating trigger tokens](https://docs.gitlab.com/ee/api/pipeline_triggers.html#create-a-trigger-token).

#### Triggering the Pipeline

We use a cURL command to trigger the pipeline:

```bash
curl --request POST \
"https://gitlab.example.com/api/v4/projects/<project_id>/trigger/pipeline?token=<token>&ref=<ref_name>"
```

Where:
- `<token>` is your trigger token
- `<ref_name>` is a branch or tag name (e.g., `main`)
- `<project_id>` is your project ID (e.g., `123456`), which can be found on the repository page

#### Passing User Input

We pass the user-provided values in the request body as a JSON object. Our script later parses this JSON to extract the necessary information.

#### Pipeline Actions

Once triggered, the pipeline:
1. Manages the state of base infrastructure creation
2. Raises a Merge Request (MR) to our Infrastructure as Code (IaC) repository

### Step 3: Automated Resource Creation

Our pipeline includes a script that:

1. Raises the MR to GitLab
2. Creates resources in a defined sequence
3. Automates the entire process

The script creates the resources in sequence, and the entire process is automated within the pipeline.

DRAFT
DRAFT
DRAFT
DRAFT
DRAFT
DRAFT
