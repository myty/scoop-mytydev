# Automated Agents

This repository uses several automated agents and workflows to maintain the Scoop bucket with minimal manual intervention.

## GitHub Actions Workflows

### 1. Excavator (Auto-Update Agent)

**Workflow:** [.github/workflows/excavator.yml](.github/workflows/excavator.yml)

**Purpose:** Automatically checks for new versions of manifests and creates pull requests with updates.

**Schedule:** Runs every 4 hours (at 20 minutes past the hour)

**Capabilities:**
- Scans all manifests in the `bucket/` directory
- Uses `checkver` configuration in each manifest to detect new versions
- Downloads and verifies file hashes
- Creates automated pull requests with version updates
- Can be triggered manually via workflow dispatch

**Requirements:**
- Manifests must have valid `checkver` and `autoupdate` sections
- Repository must have "Read and write permissions" enabled for workflows
- Uses `GITHUB_TOKEN` for authentication

**Configuration:**
- `SKIP_UPDATED: 1` - Skips manifests that are already up-to-date

### 2. CI/CD Testing Agent

**Workflow:** [.github/workflows/ci.yml](.github/workflows/ci.yml)

**Purpose:** Validates manifest files and runs tests on every push and pull request.

**Triggers:**
- Push to `main` or `master` branches
- Pull requests
- Manual workflow dispatch

**Test Matrix:**
- **WindowsPowerShell 5.1**: Tests compatibility with legacy Windows PowerShell
- **PowerShell 7+**: Tests compatibility with modern PowerShell Core

**Test Suite:**
- Manifest JSON schema validation
- URL accessibility checks
- Hash verification
- Manifest completeness checks
- Best practices validation

**Dependencies:**
- BuildHelpers module (2.0.1+)
- Pester testing framework (5.2.0+)
- Scoop core repository

### 3. Issue Handler Agent

**Workflow:** [.github/workflows/issues.yml](.github/workflows/issues.yml)

**Purpose:** Automatically processes and validates issues reported by users.

**Triggers:**
- When a new issue is opened
- When an issue is labeled with `verify`

**Capabilities:**
- Parses issue templates for manifest requests
- Validates manifest URLs and hashes
- Checks if manifests already exist in other buckets
- Provides automated feedback on issue validity
- Can automatically close duplicate or invalid issues

**Requirements:**
- Uses `GITHUB_TOKEN` for issue management
- Follows ScoopInstaller issue handling conventions

## Local Testing Scripts

### bin/test.ps1

**Purpose:** Run the complete test suite locally before pushing changes.

**Usage:**
```powershell
.\bin\test.ps1
```

**Requirements:**
```powershell
Install-Module -Name Pester -MinimumVersion 5.2 -MaximumVersion 5.99 -Scope CurrentUser
Install-Module -Name BuildHelpers -MinimumVersion 2.0.1 -Scope CurrentUser
```

### bin/checkver.ps1

**Purpose:** Manually check for version updates across all manifests.

**Usage:**
```powershell
# Check all manifests
.\bin\checkver.ps1

# Check specific manifest
.\bin\checkver.ps1 <manifest-name>

# Update manifests with new versions
.\bin\checkver.ps1 -u
```

### bin/checkhashes.ps1

**Purpose:** Verify file hashes in manifests match actual downloads.

**Usage:**
```powershell
.\bin\checkhashes.ps1
```

### bin/checkurls.ps1

**Purpose:** Verify all URLs in manifests are accessible.

**Usage:**
```powershell
.\bin\checkurls.ps1
```

### bin/formatjson.ps1

**Purpose:** Format JSON manifests to match Scoop style guidelines.

**Usage:**
```powershell
.\bin\formatjson.ps1
```

### bin/auto-pr.ps1

**Purpose:** Create automated pull requests for manifest updates.

**Usage:**
```powershell
.\bin\auto-pr.ps1
```

**Configuration:**
- Upstream: `myty/scoop-mytydev:master`
- Requires appropriate GitHub permissions

## Agent Configuration

### Enabling Automated Agents

1. **Enable GitHub Actions:**
   - Go to repository Settings → Actions → General
   - Under "Actions permissions", select "Allow all actions and reusable workflows"

2. **Grant Write Permissions:**
   - Go to Settings → Actions → General → Workflow permissions
   - Select "Read and write permissions"
   - Check "Allow GitHub Actions to create and approve pull requests"

3. **Configure Secrets (Optional):**
   - For rate-limited APIs, add `SCOOP_GH_TOKEN` personal access token
   - Go to Settings → Secrets and variables → Actions → New repository secret

### Disabling Specific Agents

To disable an agent, delete or rename its workflow file:
- Disable auto-updates: Delete `.github/workflows/excavator.yml`
- Disable CI tests: Delete `.github/workflows/ci.yml`
- Disable issue handling: Delete `.github/workflows/issues.yml`

Or disable via workflow file by commenting out the trigger section.

## Best Practices for Working with Agents

### For Manifest Authors

1. **Always include `checkver` and `autoupdate`** sections in manifests for Excavator compatibility
2. **Test locally first** using `.\bin\test.ps1` before pushing
3. **Format JSON** using `.\bin\formatjson.ps1` before committing
4. **Verify hashes** manually when autoupdate fails
5. **Review automated PRs** from Excavator before merging

### For Repository Maintainers

1. **Monitor Excavator runs** for patterns in failures
2. **Keep workflows updated** with latest ScoopInstaller/GithubActions versions
3. **Review and merge automated PRs** promptly to keep manifests current
4. **Configure branch protection** to require CI passing before merge
5. **Regularly update dependencies** (Pester, BuildHelpers modules)

## Troubleshooting

### Excavator Not Creating PRs

- Verify workflow permissions are set to "Read and write"
- Check if manifests have valid `checkver` and `autoupdate` sections
- Review workflow run logs in Actions tab
- Ensure `GITHUB_TOKEN` has sufficient permissions

### CI Tests Failing

- Run tests locally to reproduce: `.\bin\test.ps1`
- Check Pester and BuildHelpers module versions
- Verify manifest JSON syntax
- Check URL accessibility and hash validity

### Issue Handler Not Responding

- Verify workflow is enabled
- Check if issue follows expected format
- Review workflow run logs for errors
- Ensure `GITHUB_TOKEN` permissions include issue management

## Resources

- [Scoop Wiki](https://github.com/ScoopInstaller/Scoop/wiki)
- [App Manifests Documentation](https://github.com/ScoopInstaller/Scoop/wiki/App-Manifests)
- [Contributing Guide](https://github.com/ScoopInstaller/.github/blob/main/.github/CONTRIBUTING.md)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [ScoopInstaller/GithubActions](https://github.com/ScoopInstaller/GithubActions)
