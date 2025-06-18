# Troubleshooting Reusable Workflow Issues

## Common Issues and Solutions

### 1. "Workflow was not found" Error

**Error Message:**
```
failed to fetch workflow: workflow was not found
```

**Possible Causes and Solutions:**

#### A. Repository Reference Issue
**Problem:** Incorrect repository name or organization
**Solution:** Verify the repository reference format

```yaml
# Correct format
uses: kevlar-group/kevlar-github-actions/.github/workflows/test.yml@main

# Make sure:
# 1. Organization name is correct: 'kevlar-group'
# 2. Repository name is correct: 'kevlar-github-actions'
# 3. Branch/tag reference exists: '@main'
```

#### B. Workflow File Not Accessible
**Problem:** The workflow file exists but isn't accessible to external repositories
**Solution:** Check repository permissions and workflow configuration

1. **Verify the workflow has `workflow_call` trigger:**
```yaml
name: SonarQube Job
on:
  workflow_call:  # This is required for reusable workflows
    inputs:
      # ... inputs
```

2. **Check repository visibility:**
   - If the repository is private, ensure the calling repository has access
   - Consider making the repository public if workflows should be widely accessible

3. **Verify file path:**
   - Ensure the workflow file is in `.github/workflows/` directory
   - Check for typos in the filename

#### C. Branch/Tag Reference Issue
**Problem:** The referenced branch or tag doesn't exist
**Solution:** Use an existing branch or tag

```yaml
# Use a specific branch that exists
uses: kevlar-group/kevlar-github-actions/.github/workflows/test.yml@main

# Or use a specific tag
uses: kevlar-group/kevlar-github-actions/.github/workflows/test.yml@v1.0.0
```

### 2. Testing Workflow Access

To test if a workflow is accessible, try this minimal test:

```yaml
name: Test Workflow Access
on:
  workflow_dispatch:

jobs:
  test-access:
    uses: kevlar-group/kevlar-github-actions/.github/workflows/test.yml@main
    with:
      postgres-db: 'test_db'
    secrets: inherit
```

### 3. Step-by-Step Debugging

1. **Check Repository URL:**
   - Visit: `https://github.com/kevlar-group/kevlar-github-actions`
   - Verify the repository exists and is accessible

2. **Check Workflow Files:**
   - Navigate to: `.github/workflows/` in the repository
   - Verify the workflow file exists

3. **Check Branch:**
   - Ensure the `main` branch exists
   - Or use a different branch that exists

4. **Test with Public Repository:**
   - If the repository is private, try making it public temporarily
   - Or ensure proper access permissions are set

### 4. Alternative Solutions

#### A. Use Specific Commit SHA
Instead of using `@main`, use a specific commit SHA:

```yaml
uses: kevlar-group/kevlar-github-actions/.github/workflows/test.yml@abc123def456
```

#### B. Fork the Repository
If access issues persist, fork the repository to your organization:

```yaml
uses: your-org/kevlar-github-actions/.github/workflows/test.yml@main
```

#### C. Copy Workflows Locally
As a last resort, copy the workflow files to your repository:

```yaml
# Instead of using external workflow
uses: ./.github/workflows/test.yml
```

### 5. Repository Configuration Checklist

Ensure your reusable workflow repository has:

- [ ] Workflows in `.github/workflows/` directory
- [ ] `workflow_call` trigger in each reusable workflow
- [ ] Proper input/output definitions
- [ ] Accessible branch (main/master)
- [ ] Appropriate repository visibility settings
- [ ] No syntax errors in workflow files

### 6. Getting Help

If issues persist:

1. Check the GitHub Actions documentation on reusable workflows
2. Verify repository permissions and access
3. Test with a simple workflow first
4. Check GitHub's status page for any service issues
5. Open an issue in the reusable workflow repository 