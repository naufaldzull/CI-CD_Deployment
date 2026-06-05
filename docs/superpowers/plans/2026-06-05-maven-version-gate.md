# Maven Version Gate Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fail pull request builds and push deployments when `pom.xml` keeps the same Maven `<version>` as the comparison commit.

**Architecture:** Add one GitHub Actions step before Maven build in `.github/workflows/maven.yml` and one step before deploy in `.github/workflows/maven-publish.yml`. The PR step compares against `origin/main`; the deploy step compares against `HEAD^`; each exits non-zero when versions match.

**Tech Stack:** GitHub Actions, Bash, Maven project metadata in `pom.xml`.

---

### Task 1: Add Version Gate

**Files:**
- Modify: `.github/workflows/maven.yml`
- Modify: `.github/workflows/maven-publish.yml`

- [ ] **Step 1: Add a PR-only version check before build**

```yaml
    - name: Require pom.xml version bump
      if: github.event_name == 'pull_request'
      shell: bash
      run: |
        git fetch origin main --depth=1
        current_version="$(grep -m1 '<version>' pom.xml | sed -E 's/.*<version>([^<]+)<\/version>.*/\1/')"
        base_version="$(git show origin/main:pom.xml | grep -m1 '<version>' | sed -E 's/.*<version>([^<]+)<\/version>.*/\1/')"

        echo "Current version: ${current_version}"
        echo "Main version: ${base_version}"

        if [ "${current_version}" = "${base_version}" ]; then
          echo "::error::pom.xml <version> must be changed before this pull request can pass deployment checks."
          exit 1
        fi
```

- [ ] **Step 2: Run Maven test**

Run: `mvn -B test`

Expected: `BUILD SUCCESS`

- [ ] **Step 3: Review diff**

Run: `git diff -- .github/workflows/maven.yml .github/workflows/maven-publish.yml`

Expected: The build workflow contains one PR-only version gate before `Build and test`, and the publish workflow contains one push-only version gate before `Publish to GitHub Packages Apache Maven`.
