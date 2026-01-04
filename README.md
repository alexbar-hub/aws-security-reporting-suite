# AWS Security Reporting Suite

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![CloudFormation](https://img.shields.io/badge/CloudFormation-FF9900?style=flat&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/cloudformation/)

Automatically generate and publish AWS Security Hub (CSPM) and Amazon Inspector findings for collaboration in a secure, human-readable format.

## Table of contents
- [AWS Security Reporting Suite](#aws-security-reporting-suite)
  - [Table of contents](#table-of-contents)
  - [📖 Introduction](#-introduction)
  - [🚀 What it does](#-what-it-does)
  - [📐 Design](#-design)
    - [Assumptions](#assumptions)
    - [Architecture](#architecture)
  - [🛠️ Automated Resources (common to both templates)](#️-automated-resources-common-to-both-templates)
  - [Deployment](#deployment)
    - [Manual steps - Prerequisites](#manual-steps---prerequisites)
    - [Automated steps](#automated-steps)
    - [Manual steps - Post-deployment](#manual-steps---post-deployment)
  - [⚖️ License](#️-license)
  - [Credits](#credits)

## 📖 Introduction
Maintaining security visibility across an organization is difficult when stakeholders lack direct access to the AWS Console. This repository automates the creation of weekly security reports and publishes them via **Amazon CloudFront** using **Signed URLs**. This allows teams to integrate live security data directly into documentation platforms like **Confluence**.

## 🚀 What it does
This solution deploys an end-to-end serverless pipeline:
* **Extraction:** A Lambda function queries AWS Security Hub CSPM to extract findings where the `Product Name` is specifically filtered for:
  *  `aws-securityhub-reports.yaml`: SecurityHub
  *  `aws-inspector-reports.yaml`: Inspector
* **Storage:** Findings are converted to CSV and stored in a hardened S3 bucket.
* **Security:** Reports are served via CloudFront; direct S3 access is blocked.
* **Delivery:** Provides signed URLs for secure embedding in external tools.

## 📐 Design
### Assumptions
* An **Audit Account** is configured as the Delegated Administrator for both Security Hub and Amazon Inspector.
* Security Hub is configured with **Cross-Region Aggregation**.

### Architecture

![SecurityHub CSPM Reports Design](images/aws-security-reporting.drawio.png)
*Figure 1: SecurityHub CSPM Reports Design*

[Design Source - draw.io](https://github.com/alexbar-hub/aws-security-reporting/blob/main/images/aws-security-reporting.drawio)

## 🛠️ Automated Resources (common to both templates)
* **S3 Bucket:** Stores reports with an automated Lifecycle Policy (90 days to Glacier, 5-year deletion).
* **AWS Lambda:** Extracts findings and generates the CSV files.
* **Amazon SNS:** Notifies teams via Email/HTTPS when new reports are available.
* **EventBridge Rule:** Triggers the generation logic on a weekly schedule.
* **CloudFront:** Distribution with custom key, group, and cache policy.

**Report Output:**
* **Timestamped Archive:** `2024-09-26-05:42:12-SecurityHub_Findings.csv` and `2024-09-26-05:42:12-Inspector_Findings.csv`
* **Latest Date/Time Static Files:** `SecurityHub_Findings_Latest_Date_Time.csv` and `Inspector_Findings_Latest_Date_Time.csv` (used to show in Confluence when the latest report was generated).
* **Latest Static Files:** `SecurityHub_Findings_Latest.csv` and `Inspector_Findings_Latest.csv` (used for Confluence presentation).

At the end of the execution, once the reports above are created, a message containing the Confluence link is posted automatically to the designated Slack channel (this applies to both templates).


## Deployment

### Manual steps - Prerequisites
Generate a private/public key pair using OpenSSL and save it somewhere safe:
```bash
openssl genrsa -out custom-name-private_key.pem 2048
openssl rsa -pubout -in custom-name-private_key.pem -out custom-name-public_key.pem
```

### Automated steps
Deploy the template, making sure to provide (in addition to the other parameters) the **public key** created above. All fields are mandatory.

### Manual steps - Post-deployment
* generate a signed CloudFront URL for the S3 objects as it follows:
  * connect to the audit account via cli
  * run the following command to generate the signed URLs for both the Date-Time and the Findings reports, and for both report types.

```bash
$ aws cloudfront sign \
>  --url https://<your-distribution-id>.cloudfront.net/SecurityHub_Findings_Latest.csv \
>  --key-pair-id <Your-CloudFront-Key-Pair-Id> \
>  --private-key file://custom-name-private_key.pem \
>  --date-less-than 2099-10-19
https://<your-distribution-id>.cloudfront.net/SecurityHub_Findings_Latest.csv?Expires=4096051200&Signature=something-something-something-something&Key-Pair-Id=my-key

```

Confluence Setup:

* **Create Sections:** Add a heading or section called SecurityHub Report
* **Direct Download:** Create a standard text link (e.g., "Download Latest CSV") and attach the **Findings report** signed URL to it.
* **Timestamp:** Add a **Table from CSV** macro and paste the signed URL for the **Date-Time** file. This provides a clean header showing when the data was last refreshed (in UTC).
* **Data Table:** Add a second **Table from CSV** macro and paste the signed URL for the **Findings report** file to display the findings directly on the page.
* **Repeat:** Follow the same steps for the other report sets.

## ⚖️ License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.

## Credits
* [Bryan Chua - ykbryan](https://github.com/ykbryan) - Original author of the lambda function that I repurposed, available [here](https://github.com/ykbryan/lambda-get-securityhub-findings).
