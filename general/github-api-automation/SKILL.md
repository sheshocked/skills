---
name: github-api-automation
description: Automate GitHub releases, assets uploading, tags parsing, and runner tracking using octokit and raw curl endpoints.
category: general
tags: [github-api, octokit, releases, automation, bash, python]
---

# Github Api Automation

## When to Use
Use when implementing automatic release compilation pipelines (like uploading compiled VPN APK assets to GitHub releases dynamically on tag pushes).

## Prerequisites
- GitHub Personal Access Token (PAT).
- `curl` or `octokit` library configured.

## Workflow
1. Authenticate with GitHub endpoints using secure tokens.
2. Construct release payload data structures (tag name, target commit, release description).
3. Post data payload to establish release tag.
4. Stream compiled binary packages to the release assets upload endpoint.

## Key Patterns

### Python Release Asset Upload Script (upload_release.py)
```python
import os
import requests

GITHUB_TOKEN = os.environ.get("GITHUB_TOKEN")
REPO = "sheshocked/skills"
TAG = "v2.5.0"

headers = {
    "Authorization": f"token {GITHUB_TOKEN}",
    "Accept": "application/vnd.github.v3+json"
}

# 1. Create the Release
release_url = f"https://api.github.com/repos/{REPO}/releases"
release_payload = {
    "tag_name": TAG,
    "target_commitish": "main",
    "name": f"Release {TAG}",
    "body": "Automated masterpiece skills build compiled successfully.",
    "draft": false,
    "prerelease": false
}

r = requests.post(release_url, json=release_payload, headers=headers)
r.raise_for_status()
release_data = r.json()
upload_url_template = release_data["upload_url"] # E.g., "https://uploads.github.com/.../assets{?name,label}"

# Extract base upload URL
upload_url = upload_url_template.split("{")[0]

# 2. Upload Asset Binary
file_path = "/tmp/build.zip"
file_name = "build.zip"

with open(file_path, "rb") as f:
    upload_headers = {
        **headers,
        "Content-Type": "application/zip"
    }
    params = {"name": file_name}
    res = requests.post(f"{upload_url}", data=f, headers=upload_headers, params=params)
    res.raise_for_status()

print("Asset binary uploaded successfully to GitHub Release!")
```

## Pitfalls
- **Upload URL interpolation:** The `upload_url` returned from GitHub contains template parameters `{?name,label}`. Strip this string wrapper before triggering POST requests.
- **Token scope limit:** Restrict PAT token scope settings strictly to `public_repo` permissions to prevent wider account access exposures.

## Verification
- Query repo assets list using curl: `curl https://api.github.com/repos/sheshocked/skills/releases/tags/v2.5.0`.
- Verify uploaded file matches compiled MD5 hashes.

