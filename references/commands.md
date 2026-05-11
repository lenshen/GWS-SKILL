# GWS CLI Workspace Commands

## Install GWS From GitHub Release

```powershell
$version = "v0.22.5"
$dir = ".\tools\gws"
New-Item -ItemType Directory -Force -Path $dir | Out-Null
Invoke-WebRequest -Uri "https://github.com/googleworkspace/cli/releases/download/$version/google-workspace-cli-x86_64-pc-windows-msvc.zip" -OutFile "$dir\google-workspace-cli-x86_64-pc-windows-msvc.zip"
Invoke-WebRequest -Uri "https://github.com/googleworkspace/cli/releases/download/$version/google-workspace-cli-x86_64-pc-windows-msvc.zip.sha256" -OutFile "$dir\google-workspace-cli-x86_64-pc-windows-msvc.zip.sha256"
Get-FileHash "$dir\google-workspace-cli-x86_64-pc-windows-msvc.zip" -Algorithm SHA256
Get-Content "$dir\google-workspace-cli-x86_64-pc-windows-msvc.zip.sha256"
Expand-Archive -Force -Path "$dir\google-workspace-cli-x86_64-pc-windows-msvc.zip" -DestinationPath $dir
.\tools\gws\gws.exe --version
```

Always verify the latest version and asset names before using this snippet.

## Add User PATH

```powershell
$gwsDir = "C:\Users\<USER>\Documents\New project\tools\gws"
$gcloudBin = "C:\CloudSDK\google-cloud-sdk\bin"
$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
$parts = @()
if (-not [string]::IsNullOrWhiteSpace($userPath)) {
  $parts = $userPath -split ";" | Where-Object { -not [string]::IsNullOrWhiteSpace($_) }
}
foreach ($p in @($gwsDir, $gcloudBin)) {
  if ($parts -notcontains $p) { $parts += $p }
}
setx PATH ($parts -join ";")
```

New shells are usually required before PATH updates are visible.

## Manual OAuth Client JSON Shape

```json
{
  "installed": {
    "client_id": "<CLIENT_ID>",
    "project_id": "<PROJECT_ID>",
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
    "client_secret": "<CLIENT_SECRET>",
    "redirect_uris": ["http://localhost"]
  }
}
```

Save as `%USERPROFILE%\.config\gws\client_secret.json`.

## Useful Gmail Queries

```powershell
gws gmail users messages list --params '{"userId":"me","q":"after:2026/5/11 before:2026/5/12","maxResults":20}'
gws gmail users messages get --params '{"userId":"me","id":"MESSAGE_ID","format":"metadata","metadataHeaders":["From","To","Subject","Date"]}'
gws gmail users labels list --params '{"userId":"me"}'
```

## Common Failure Modes

- `npm` missing while `node` exists: Node may be embedded by an app runtime. Use the release zip instead of npm.
- `No GCP project configured`: run `gcloud projects list` and `gcloud config set project <PROJECT_ID>`, or pass `--project`.
- `OAuth client creation requires manual setup`: configure Google Auth Platform brand and Desktop OAuth client in Console.
- `UREQ_TOS_NOT_ACCEPTED`: accept the service/API terms in Cloud Console.
- `accessNotConfigured`: enable the named API with `gcloud services enable`.
- Interrupted login leaves `gws` processes: inspect and stop stale `gws` processes before retrying.
