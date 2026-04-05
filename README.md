# On-Premise SMB Shares to AWS Storage Gateway (S3) Data Transfer Automation

A robust, automated solution for securely synchronizing files from on-premises SMB file shares to AWS S3 using **AWS DataSync**, **Lambda**, **EventBridge**, **Systems Manager (SSM)**, and **Storage Gateway**.

---

## Overview

This project implements a fully automated data transfer pipeline that synchronizes files from two on-premises locations to AWS S3. The solution ensures reliable, scheduled transfers, automatic cache refresh for Storage Gateway, and safe deletion of source files after successful transfer.

**Key Features:**
- Scheduled full directory synchronization every 2 hours
- Automatic Storage Gateway cache refresh after successful transfers
- Automated deletion of successfully transferred files from on-premises shares
- Comprehensive monitoring and alerting
- Detailed reporting and audit trails

---

## Architecture
<img width="1173" height="912" alt="image" src="https://github.com/user-attachments/assets/ff4cd9b4-4c1a-4f04-be63-9c4de9a15c18" />


The solution integrates the following AWS services:
- **AWS DataSync** – High-performance data transfer from SMB shares to S3
- **Amazon S3** – Destination storage with organized prefixes
- **AWS Lambda** – Handles cache refresh and delete manifest processing
- **Amazon EventBridge** – Triggers Lambda functions on DataSync success events
- **AWS Systems Manager** – Executes secure PowerShell scripts for file deletion
- **AWS Storage Gateway** – Provides on-demand cache refresh capability
- **Amazon CloudWatch + SNS** – Monitoring and alerting

### High-Level Flow
1. DataSync tasks run on schedule and transfer files to S3
2. On success, EventBridge triggers Lambda to refresh Storage Gateway cache
3. Lambda parses DataSync reports and creates delete manifests
4. SSM Run Command executes deletion of only successfully transferred files from source SMB shares

---

## Key Components

### On-Premises
- **Site 1 (Madison)**: SMB Share – `D:\AWSSync\` on host `host-1`
- **Site 2 (San Diego)**: SMB Share – `S:\AWSSync\` on host `host-2`
- DataSync Agents deployed at both locations

### AWS Components
- **S3 Destination Bucket** with prefixes:
  - `/data-from-madison/`
  - `/data-from-sandiego/`
- Two DataSync Tasks (one per site)
- Four Lambda functions (Cache Refresh + Delete automation for each site)
- Reports stored in dedicated S3 report prefixes

---

## Implementation Details

### 1. DataSync Configuration
- **Transfer Mode**: Full directory sync (files + folders)
- **Schedule**: Every 2 hours
- **Reports**: Automatically generated and stored in S3

### 2. Cache Refresh Automation
- EventBridge rules listen for DataSync `SUCCESS` events
- Triggers site-specific Lambda functions to refresh Storage Gateway cache for the corresponding S3 prefix

### 3. Source File Deletion Automation
- Lambda functions parse DataSync reports
- Generate delete manifests
- SSM Run Command executes `delete-files.ps1` PowerShell script
- Only files confirmed as successfully transferred are deleted from on-premises shares

---

## Monitoring & Alerting

- **CloudWatch Alarms** for DataSync task failures and Lambda error rates
- **SNS Notifications** sent to operations team on critical events
- Detailed CloudWatch Logs for all Lambda functions and DataSync executions

---

## Validation & Testing

The implementation includes:
- Successful DataSync task executions
- Verified S3 file uploads
- Lambda invocation logs
- SSM execution reports for file deletions

---

## Troubleshooting

### Common Issues & Resolutions

**DataSync Task Failures**
- Check Execution History for error details
- Verify IAM roles, network connectivity, and SMB share accessibility

**Lambda Function Errors**
- Review CloudWatch Logs for the specific function
- Check IAM permissions and timeout settings

**SSM Run Command Failures**
- Validate SSM Agent status on target instances
- Ensure proper IAM roles and PowerShell script execution

---

## License
This project is for demonstration and portfolio purposes. Feel free to use as a reference for building similar automation solutions.
