# apk-static-analysis-keenu-1j4iq

## Overview

This n8n workflow automates the static security analysis of Android APKs. It handles APK acquisition (via Google Drive or package name), performs detailed scans using MobSF and JADX for decompilation, extracts hardcoded secrets and security issues based on a comprehensive set of regex patterns, generates a structured HTML and PDF report, and then archives and distributes these reports via Google Drive and email.

## Features

- Automated APK download from Google Drive or via package name using apkeep.
- Comprehensive static analysis leveraging a MobSF instance for general security posture.
- JADX decompilation for in-depth source code analysis.
- Custom secret extraction engine with extensive regex patterns, including specialized Fintech (Pakistan) patterns.
- Detection of hardcoded API keys, credentials, tokens, private keys, weak cryptography, logging issues, and AndroidManifest misconfigurations.
- Dynamic HTML report generation with summary and detailed findings.
- Conversion of HTML reports to PDF using a third-party API.
- Consolidated packaging of MobSF and custom secrets reports into a single tar.gz archive.
- Automated upload of archived reports to a designated Google Drive folder.
- Email notification with attached report for stakeholders.
- Scheduled execution for continuous or periodic application security scanning.
- Robust error handling and cleanup mechanisms for temporary files and directories.

## Services Used

- n8n (workflow automation platform)
- Docker (for containerized execution of command-line tools)
- MobSF (Mobile Security Framework, for static analysis via API)
- Google Drive (for APK input and report output storage)
- Gmail (for email notifications with reports)
- HTML2PDF.app (third-party API for HTML to PDF conversion)
- apkeep (Dockerized utility for downloading APKs)
- JADX (Java Decompiler, for decompiling APKs to source code)
- Standard Linux command-line utilities (e.g., `grep`, `sed`, `curl`, `jq`, `unzip`, `tar`)

## Trigger

The workflow is triggered by a Schedule Trigger set to run weekly, specifically on Monday at 11:00 AM, in the Asia/Karachi timezone.

## Prerequisites

- An n8n instance with the 'Execute Command' node enabled.
- A running Docker daemon accessible from the n8n host.
- A deployed MobSF (Mobile Security Framework) instance accessible via API at the configured URL.
- An API key for the html2pdf.app service (or a similar HTML to PDF conversion API).
- Google Drive OAuth2 credentials configured in n8n.
- Gmail OAuth2 credentials configured in n8n.
- Linux utilities like `jq`, `curl`, `unzip`, `grep`, `sed`, `wc`, `tar`, `ln`, `rm`, `mkdir`, `chmod`, `cp`, `head`, `basename`, `date` available in the n8n execution environment.

## Credentials

- Google Drive OAuth2 API (for downloading APKs and uploading reports)
- Gmail OAuth2 (for sending email notifications)
- MobSF API Key (for interacting with the MobSF API)
- HTML2PDF.app API Key (for converting HTML reports to PDF)

## Configuration

1. 1. **N8n Credentials**: Configure 'Google Drive OAuth2' and 'Gmail OAuth2' credentials in your n8n instance.
2. 2. **Edit Fields Node**: Update the 'Edit Fields' node parameters:
3.    - `package_name`: The Android package name (e.g., `com.example.app`) if using the `apkeep` download method.
4.    - `app_name`: A user-friendly name for the application.
5.    - `mobsf_api_key`: Your MobSF API key. (Note: This is also hardcoded in the 'Execute Command - MobSF Scan' node and should be kept consistent).
6.    - `mobsf_url`: The URL of your MobSF instance (e.g., `http://172.17.0.5:8000`). (Note: This is also hardcoded in the 'Execute Command - MobSF Scan' node).
7.    - `drive_link`: An optional Google Drive shareable link to an APK file. If provided, the workflow will download from Drive; otherwise, it will use `apkeep`.
8.    - `htmlpdf_api_key`: Your API key for html2pdf.app.
9. 3. **MobSF Setup**: Ensure your MobSF instance is running, accessible from the n8n host at the specified `mobsf_url`, and configured with the `mobsf_api_key`.
10. 4. **Docker Images**: Ensure Docker can pull and run the necessary images: `ghcr.io/efforg/apkeep:stable`, `alpine`, and `abh1sek/jadx`.
11. 5. **Google Drive Folder**: Verify the Google Drive folder ID `1VpgxK4O03Jrhr3DyBlLTL0jZD_avlYLG` exists and the configured Google Drive credential has write access to it.
12. 6. **Execution Environment**: Ensure the n8n execution environment has sufficient disk space in `/tmp` for temporary files (APKs, decompiled source, reports) and proper permissions for creating/deleting directories.

## Usage

1. 1. **Input Configuration**: Before execution, modify the 'Edit Fields' node to specify the target APK's `package_name` or `drive_link`.
2. 2. **Manual Execution**: To run the analysis immediately, click the 'Execute Workflow' button in the n8n editor.
3. 3. **Scheduled Execution**: The workflow is configured to automatically trigger every Monday at 11:00 AM (Asia/Karachi timezone) based on the 'Schedule Trigger' node.
4. 4. **Report Retrieval**: Upon successful completion, the generated MobSF and secrets reports (PDFs) will be archived and uploaded to the configured Google Drive folder. An email with the combined report attached will also be sent to `farhad.ahmed@keenu.pk` (configurable in 'Send a message' node).

## Troubleshooting

- **No APK Found**: If the workflow fails with 'ERROR: No APK/XAPK found...', check the `package_name` (for apkeep) or `drive_link` (for Google Drive). Ensure the Google Drive link is publicly accessible or correctly shared with the service account.
- **MobSF Upload Failed**: If MobSF upload fails, verify the `mobsf_url` and `mobsf_api_key` are correct and that the MobSF instance is running and reachable from the n8n host. Check the MobSF server logs for errors.
- **Decompilation Failed**: JADX errors ('CRITICAL: Decompilation failed!') can occur with highly obfuscated or corrupted APKs. Review the n8n execution output for JADX specific error messages.
- **PDF Conversion Issues**: If the 'Secrets to PDF' node fails, check the `htmlpdf_api_key`. Examine the raw output from the HTML2PDF API call for specific error messages.
- **No PDFs to Package**: This indicates either the MobSF report or the Secrets report was not generated. Review logs of preceding nodes ('MobSF Scan', 'Extract Secrets', 'Secrets to PDF') for failures.
- **Permission Errors**: Errors related to file system permissions (e.g., `Permission denied`) often indicate that the n8n user lacks necessary rights to create/write files in `/tmp/apk_analysis_keenu`. Ensure Docker has appropriate permissions and volumes are mounted correctly.
- **Hardcoded MobSF Values**: Note that the 'Execute Command - MobSF Scan' node uses hardcoded `API_KEY` and `MOBSF_URL` values. If you update these in 'Edit Fields', you must also manually update them within the script itself for MobSF integration to function correctly.

## Security Notes

- **Hardcoded API Keys**: The MobSF API key and URL are hardcoded directly within the 'Execute Command - MobSF Scan' script, which is a security anti-pattern. Ideally, these should be dynamically pulled from n8n's 'Credentials' system.
- **Sensitive Data Handling**: The workflow processes and generates reports containing highly sensitive information (e.g., hardcoded credentials, API keys). All outputs (Google Drive uploads, email attachments) should be handled with extreme care due to the 'TLP:RED — Confidential' classification indicated in the report.
- **Insecure File Permissions**: The script uses `chmod 777` for temporary report directories. While these are temporary, `777` grants global read/write/execute permissions, which is a security risk. Consider using more restrictive permissions like `700` or `755` if possible.
- **Docker Container Security**: Running various Docker images from external registries (`apkeep`, `jadx`) introduces a dependency on their security. Ensure these images are trusted and regularly updated.
- **`/tmp` Directory Usage**: Using `/tmp` for sensitive analysis artifacts can be risky if the environment is not properly secured, as data in `/tmp` might be accessible to other processes or users.
- **Cleanup Robustness**: While extensive cleanup is performed, ensure no sensitive data persists on the n8n host or in Docker volumes inadvertently.
