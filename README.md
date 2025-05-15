# Black Duck Software Integration Lab 4

The goal of this lab is to provide hands on experience integrating a **Black Duck SCA** scan into a **GitHub** workflow using the [Black Duck Security Scan Action](https://github.com/marketplace/actions/black-duck-security-scan) and demonstrating its post scan capabilities. As part of the lab, we will:
1. setup a GitHub workflow and execute a full scan
2. break the build based on a [policy](https://documentation.blackduck.com/bundle/bd-hub/page/Policies/Overview.html) defined in Black Duck Hub
3. review the findings in Black Duck Hub
4. review the Fix PR for automatic vulnerability remediation
5. review the code scanning findings in the GitHub Advanced Security tab
6. introduce a vulnerable code change, triggering a PR scan which adds a comment to the Pull Request

This repository contains everything you need to complete the lab except for the prerequisites listed below.

# Prerequisites

1. [signup](https://github.com/signup) for a free GitHub Account
2. access to a Black Duck SCA instance

# Clone repository

1. Clone this repository into your GitHub account via _GitHub → New → Import a Repository_
   - enter https://github.com/blackduck-se/bd-integrations-lab4.git
   - enter repository name, e.g. hello-java
   - leave as public (required for GHAS on free accounts)

# Black Duck Hub

1. [create](https://documentation.blackduck.com/bundle/bd-hub/page/UsersAndGroups/MyAccessTokens.html) a Black Duck Access Token. _Hub → User → Access Tokens → Create Token → Read and Write Access_
2. [create](https://documentation.blackduck.com/bundle/bd-hub/page/Policies/Overview.html) a Black Duck Policy. _Hub → Manage → Policies → Create Policy Rule_
   - Name: No Exploitable Vulnerabilities
   - Severity: BLOCKER
   - Category: Security
   - Scan Modes: Full, Rapid
   - Conditions: Exploit Available equals Yes

# Setup workflow

1. Confirm all GitHub Actions are allowed via _GitHub → Project → Settings → Actions → General → Actions Permissions_
2. Confirm GITHUB_TOKEN has workflow read & write permissions via _GitHub → Project → Settings → Actions → General → Workflow Permissions_
3. Confirm GitHub Actions can create and approve pull requests via _GitHub → Project → Settings → Actions → General → Workflow Permissions_
4. Add the following variables, adding BLACKDUCK_API_TOKEN as a **secret** via _GitHub → Project → Settings → Secrets and Variables → Actions_
   - BLACKDUCK_URL
   - BLACKDUCK_API_TOKEN
5. Create a new workflow via _GitHub → Project → Actions → New Workflow → Setup a workflow yourself_

```
# example workflow for Black Duck SCA scans using the Black Duck Security Scan Action
# https://github.com/marketplace/actions/black-duck-security-scan
name: Black Duck SCA
on:
  push:
    branches: [ main, master, develop, stage, release ]
  pull_request:
    branches: [ main, master, develop, stage, release ]
  workflow_dispatch:
jobs:
  blackduck_sca:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Source
      uses: actions/checkout@v4
    - name: Setup Java JDK
      uses: actions/setup-java@v4
      with:
        java-version: 21
        distribution: temurin
        cache: maven
    - name: Maven Build
      run: mvn -B -DskipTests package
    - name: Black Duck SCA Scan
      uses: blackduck-inc/black-duck-security-scan@v2.1.1
      env:
        DETECT_PROJECT_NAME: ${{ github.event.repository.name }}
      with:
        blackducksca_url: ${{ vars.BLACKDUCK_URL }}
        blackducksca_token: ${{ secrets.BLACKDUCK_API_TOKEN }}
        blackducksca_scan_failure_severities: 'BLOCKER'
        blackducksca_fixpr_enabled: true
        blackducksca_prComment_enabled: true
        blackducksca_reports_sarif_create: true
        blackducksca_upload_sarif_report: true
        github_token: ${{ secrets.GITHUB_TOKEN }}
        # include_diagnostics: true
#    - name: Save Logs
#      if: always()
#      uses: actions/upload-artifact@v4
#      with:
#        name: bridge-logs
#        path: ${{ github.workspace }}/.bridge
#        include-hidden-files: true
```

# Full Scan

1. Monitor your workflow run and wait for the scan to complete via _GitHub → Project → Actions → Black Duck SCA → Most recent workflow run → blackduck_sca_ **Milestone 1** :heavy_check_mark:
2. Note that the workflow fails because of the policy we defined in Black Duck Hub. **Milestone 2** :heavy_check_mark:
3. Log into Black Duck Hub and review the findings. There should be a blocking policy violation on log4j. **Milestone 3** :heavy_check_mark:
4. View the Fix PR for the vulnerable log4j component via _GitHub → Project → Pull requests_ **Milestone 4** :heavy_check_mark:
5. View findings in GitHub Advanced Security tab via _GitHub → Project → Security → Code scanning_ **Milestone 5** :heavy_check_mark:

# PR scan

1. Edit pom.xml via _GitHub → Project → Code → pom.xml → Edit pencil icon upper right_
   - change log4j version from 2.14.1 to 2.15.0
2. Click on _Commit Changes_, select create a **new branch** and start a PR
3. Review changes and click on _Create Pull Request_
4. Monitor workflow run via _GitHub → Project → Actions → Black Duck SCA → Most recent workflow run → blackduck_sca_
5. Once workflow completes, navigate back to the PR and review the PR comment via _GitHub → Project → Pull requests_ **Milestone 6** :heavy_check_mark:

# Congratulations

You have now configured a Black Duck SCA workflow in GitHub and demonstrated all the functionality of the [Black Duck Security Scan Action](https://github.com/marketplace/actions/black-duck-security-scan). :clap: :trophy:
