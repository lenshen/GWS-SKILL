---
name: gws-cli-workspace
description: Install, configure, troubleshoot, and use Google Workspace CLI (`gws`) on Windows for Google Workspace APIs. Use when the user asks to set up GWS CLI from googleworkspace/cli, install or verify gcloud dependencies, complete OAuth login/client setup, enable Workspace APIs, inspect `gws auth status`, or query Gmail, Drive, Calendar, Sheets, Docs, and related Workspace data from the command line.
---

# GWS CLI Workspace

## Core Workflow

Use official sources and verify current release details before installing or upgrading. Prefer the GitHub release zip on Windows when `npm` is unavailable or comes only from an embedded runtime. Keep install artifacts in a user-controlled directory unless the user asks for a system install.

1. Check local tools:

```powershell
node --version
npm --version
gcloud --version
gws --version
```

2. If `gws` is missing, download the latest Windows x64 release from `googleworkspace/cli`, download the matching `.sha256`, verify the hash, extract `gws.exe`, and add its directory to the user PATH.

3. If `gcloud` is missing and `gws auth setup` is needed, install Google Cloud CLI from the official installer. On Windows, a silent single-user install can use:

```powershell
GoogleCloudSDKInstaller.exe /S /singleuser /noreporting /nostartmenu /nodesktop /D=C:\CloudSDK
```

4. Authenticate gcloud:

```powershell
gcloud auth login --launch-browser --quiet
gcloud projects list --format=json
gcloud config set project <PROJECT_ID>
```

5. Run setup:

```powershell
gws auth setup --project <PROJECT_ID> --login
```

If setup says OAuth client creation requires manual Console setup, create/configure the Google Auth Platform brand and a Desktop app OAuth client. Save it as:

```text
C:\Users\<USER>\.config\gws\client_secret.json
```

Use `installed` JSON shape with `client_id`, `client_secret`, `auth_uri`, `token_uri`, `auth_provider_x509_cert_url`, `project_id`, and `redirect_uris`.

## OAuth Login

Avoid interactive scope pickers in non-interactive shells. Pass explicit scopes:

```powershell
gws auth login --scopes "https://www.googleapis.com/auth/drive,https://www.googleapis.com/auth/gmail.modify,https://www.googleapis.com/auth/gmail.send,https://www.googleapis.com/auth/spreadsheets,https://www.googleapis.com/auth/calendar,https://www.googleapis.com/auth/documents"
```

If a previous login was interrupted, check for stuck `gws` processes and stop only the stale `gws` login processes before retrying.

Success criteria:

```powershell
gws auth status
```

Look for `auth_method: oauth2`, `encrypted_credentials_exists: true`, `has_refresh_token: true`, `token_valid: true`, and the expected user email.

## API Enablement

`gws auth status` lists enabled APIs. If a command returns `accessNotConfigured`, enable the missing API with gcloud:

```powershell
gcloud services enable calendar-json.googleapis.com --project <PROJECT_ID>
```

Some APIs may require accepting Google or Workspace service terms in the Cloud Console. If CLI enablement fails with `UREQ_TOS_NOT_ACCEPTED`, open the URL in the error and accept the relevant terms, then retry.

## Verification Commands

After setup, test with low-impact reads:

```powershell
gws drive files list --params '{"pageSize":3,"fields":"files(id,name,mimeType)"}'
gws gmail users labels list --params '{"userId":"me"}'
gws calendar calendarList list --params '{"maxResults":3}'
gws schema drive.files.list
```

When responding to the user, summarize the result and avoid dumping sensitive credentials, full messages, or unnecessary personal data.

## Gmail Checks

For "today's email", use the user's current timezone/date and Gmail search syntax:

```powershell
gws gmail users messages list --params '{"userId":"me","q":"after:YYYY/M/D before:YYYY/M/D","maxResults":20}'
```

Fetch metadata first:

```powershell
gws gmail users messages get --params '{"userId":"me","id":"MESSAGE_ID","format":"metadata","metadataHeaders":["From","To","Subject","Date"]}'
```

Default to listing sender, subject, date/time, labels, and snippet. Do not read full bodies unless the user asks or the task requires it.

## References

For command snippets and troubleshooting patterns, read `references/commands.md`.
