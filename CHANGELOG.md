# Changelog

All notable changes to this project will be documented in this file.

## [1.1.0] - 2026-04-04

### Added
- **GuardDuty Module:** Added aws-guardduty-reports.yaml to extract findings filtered by `Product Name: GuardDuty`.
- **IAM Access Analyzer Module:** Added aws-guardduty-reports.yaml to extract findings filtered by `Product Name: IAM Access Analyzer`.

### Changed
- **Documentation:** Updated README.
- **Modules:** Minor updates for consistency.
- **Design diagram:** Added Jira ticket notification for auditing purposes.

---

## [1.1.0] - 2026-01-07

### Added
- **Route 53 Inventory Module:** Added `aws-route53-query.yaml` to automate DNS record extraction across the AWS Organization.
- **Cross-Account Permissions:** Added `aws-route53-query-basic-iam-grants.yaml` to enable the Audit account to "reach into" spoke accounts via StackSets.
- **S3 Pre-signed Delivery:** Implemented a direct S3 download workflow via SNS email notifications (valid for 12 hours) for the Route 53 ZIP archives.

### Changed
- **Documentation:** Updated README to distinguish between CloudFront-delivered CSVs and S3-delivered ZIP files.

---

## [1.0.0] - 2026-01-03

### Added
- **Project Consolidation:** Grouped two separate SecurityHub CSPM repositories into this unified **AWS Security Reporting Suite**.
- **Security Hub Module:** Added `aws-securityhub-reports.yaml` to extract findings filtered by `Product Name: SecurityHub`.
- **Inspector Module:** Added `aws-inspector-reports.yaml` to extract findings filtered by `Product Name: Inspector`.
- **CloudFront Integration:** Implemented OAC/OAI restricted distributions with Signed URL support for secure Confluence embedding.
- **Slack Notifications:** Added automated Slack alerts containing Confluence navigation links.
- **Initial Documentation:** Created a comprehensive guide for OpenSSL key-pair generation and Confluence macro setup.

### Changed
- **Rebranding:** Renamed the repository from service-specific titles to a broader "Suite" naming convention to support modular growth.