# dummy-imposter-commit-action

An educational GitHub Action that demonstrates the "imposter commit" vulnerability in GitHub Actions.

## The Problem Demonstrated by This Repository

This repository has a **v1 tag** that points to commit `891dbcd4d1bf6c2ac2a7b9d8d6656583f3b5cc9a`.

**Here's the issue**: That commit does NOT exist in this repository (`step-security/dummy-imposter-commit-action`). Instead, it only exists in a fork at `varunsh-coder/dummy-imposter-commit-action`.

Despite this, if you use this action with the v1 tag:

```yaml
uses: step-security/dummy-imposter-commit-action@v1
```

Or directly reference the commit SHA:

```yaml
uses: step-security/dummy-imposter-commit-action@891dbcd4d1bf6c2ac2a7b9d8d6656583f3b5cc9a
```

GitHub Actions will successfully fetch and execute code from the fork, even though that commit was never merged into the parent repository. The workflow appears to use code from `step-security`, but actually runs code from the fork.

## What Are Imposter Commits?

Imposter commits exploit how GitHub shares commit data between forks and parent repositories. An attacker can:

1. Fork a legitimate repository
2. Add malicious code and commit it to their fork
3. Create a tag or reference in the parent repository pointing to that fork commit
4. Victims use what appears to be the official repository but execute attacker code

## Why This is Dangerous

- **Bypasses trust**: Workflows appear to use official, trusted actions
- **Bypasses security controls**: GitHub Actions' "allowed actions" policies are circumvented because the workflow references the parent repo name
- **Hard to detect**: The commit SHA or tag looks legitimate
- **Secrets exposure**: Malicious code can exfiltrate GitHub secrets, tokens, and sensitive data

## Real-World Incidents

This vulnerability has been exploited in multiple high-profile attacks against popular GitHub Actions:

### tj-actions/changed-files (March 2025, CVE-2025-30066)

- **Action**: `tj-actions/changed-files` - used in over 23,000 repositories
- **Attack method**: Attackers compromised a Personal Access Token for the @tj-actions-bot account and retroactively updated multiple version tags to point to malicious commit `0e58ed8671d6b60d0890c21b07f8835ace038e67`
- **Payload**: Python script that read GitHub Actions Runner.Worker process memory from `/proc/[pid]/mem` to extract CI/CD secrets
- **Impact**: Secrets were base64-encoded and printed to workflow logs, exposing credentials in plaintext
- **Detection**: StepSecurity's Harden-Runner detected unexpected outbound traffic to `gist.githubusercontent.com`
- **Timeline**: Attack began March 14, 2025 at 16:00 UTC; GitHub removed the action March 15, 2025

Read more: [Harden-Runner detection: tj-actions/changed-files action is compromised](https://www.stepsecurity.io/blog/harden-runner-detection-tj-actions-changed-files-action-is-compromised)

### reviewdog/action-setup (March 2025)

- **Action**: `reviewdog/action-setup` - widely-used automated code review tool
- **Attack method**: Attackers updated the v1 tag to point to an imposter commit containing a base64-encoded Python script
- **Payload**: Script accessed Runner.Worker process memory to extract and print CI/CD secrets to workflow logs
- **Impact**: Compromised v1 tag was referenced by multiple composite Actions, expanding the blast radius
- **Timeline**: Narrow compromise window on March 11, 2025, between 18:42 and 20:31 UTC
- **Response**: Maintainers rotated secrets, revoked access tokens, and moved to commit-SHA-based pinning

These incidents demonstrated how imposter commits bypass PR reviews and normal code scrutiny, making them particularly dangerous.

Read more: [The GitHub Warning Everyone Ignores: 'This Commit Does Not Belong to Any Branch'](https://www.stepsecurity.io/blog/the-github-warning-everyone-ignores-this-commit-does-not-belong-to-any-branch)

## About This Project

This is an **educational project** that safely demonstrates the imposter commit vulnerability. The v1 tag intentionally points to a fork commit to show how this attack works in practice.

**⚠️ Warning**: Creating imposter commits for malicious purposes is unethical and may violate laws and GitHub's Terms of Service. This demonstration is for defensive security education only.

## Learn More

- [Chainguard: What the Fork? Imposter Commits in GitHub Actions](https://www.chainguard.dev/unchained/what-the-fork-imposter-commits-in-github-actions-and-ci-cd)
