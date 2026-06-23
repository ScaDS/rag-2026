# Pipeline Checks

!!! cite "Martin Fowler [1](https://martinfowler.com/bliki/FrequencyReducesDifficulty.html)"

    If it hurts, do it more often.

Because it is important to maintain a high-quality documentation, each and every commit and merge
request is running through various steps performing consistency and regression checks.

This page covers the CI/CD pipeline and the implemented checks, i. e., what our pipeline runs under
the hood and how the checks are configured and implemented. The targeted audience is contributors
and maintainers.

## Pipeline Overview

## Pipeline Configuration

The pipeline steps and rules are defined within a single file called `.gitlab-ci.yml`. By managing
the `.gitlab-ci.yaml` file within in the
[separate project `hpc-compendium-ci`](https://gitlab.hrz.tu-chemnitz.de/zih/hpcsupport/hpc-compendium-ci)
with quite strict access rules, we can protect it from unintended change. Please refer to the
official
[GitLab documentation of this pattern](https://docs.gitlab.com/ee/ci/environments/deployment_safety.html#protect-gitlab-ciyml-from-change)
for further information.

## Pipeline Schedules

Pipeline schedules allow to run GitLab CI/CD pipelines at regular interval. Currently, we have two
schedules configured as described in the following.

### Regular Deployment of Main Branch

The schedule `Merge to main branch every Monday morning` runs every Monday at 6 am (CET) . It merges
the `preview` branch into the `main` branch and deploys the updated `main` branch to the web server
making all updates to the `preview` branch from the previous week available to the users.

### Half-Yearly Check of Pages

Our goal is to manually review each page of the compendium regularly to ensure that we find mistakes
that may have been missed by the regular CI checks. Every half year, we generate an issue in GitLab
with a check box for each page to support that. Various contributors can then review the pages and
click such a checkbox when they have reviewed the content of the corresponding page.

??? info "Info for Maintainers"

    If the half-yearly check produces a warning "invalid token" or "token expired", please check
    whether the token is still valid on
    [Access token settings](https://gitlab.hrz.tu-chemnitz.de/zih/hpcsupport/hpc-compendium/-/settings/access_tokens).
    If the token is not valid or does not exist, create a new one with the following settings:

    | Name                    | Scopes | Role     | Expiry Date            |
    |-------------------------|--------|----------|------------------------|
    |`Page check reminder bot`| `api`  |`Reporter`|(select latest possible)|

    ![Token settings example in GitLab](misc/half_yearly_settings.png)

    Copy the value of the generated token and go to
    [CI variables](https://gitlab.hrz.tu-chemnitz.de/zih/hpcsupport/hpc-compendium/-/settings/ci_cd#js-cicd-variables-settings).
    Paste the generated token as the value of variable `SCHEDULED_PAGE_CHECK_PAT`.

    ![Variable `SCHEDULED_PAGE_CHECK_PAT`](misc/half_yearly_variable.png)

## Pipeline Checks

When a change is pushed to the repository and a merge request is created, the changed files are
automatically tested. The following automatic checks are currently set up:

- Structural tests:
    - Single page: Test that a page does not appear twice in the table of contents
    - Floating pages: Test that all pages are referenced in the table of contents
    - Headings: Test that an entry in the table of contents matches the heading of the page
- Page content tests:
    - Spelling: Test that a change does not increase spelling mistakes
    - Wording: Test that certain words or word groups are avoided, e. g., "work in progress"
    - Examples for templates: Test that an example exists for each template with placeholders
- Integrity tests:
    - Max depth: Test that there is only one level of subdirectories between the root directory and
      any markdown file
    - Links: Test that targets of links exists
    - Footer: Test that the footer contains links to certain pages
    - Valid Markdown: Test that files contain valid markdown
    - Size: Test that an added file does not exceed a certain size
    - Empty pages: Test that a change does not add empty pages
    - Code style: Test that bash scripts follow a defined style

More information can be found below.

### Link Checking

The ZIH HPC Compendium has a lot of internal and external references and links in order
to connect documentation and allow for further reading.

For link validation, we make use of the
[mkdocs-htmlproofer-plugin](https://github.com/manuzhang/mkdocs-htmlproofer-plugin).
It is configured in the `mkdocs.yml` file of this project (cf.
[section Technical Setup](howto_contribute.md#technical-setup)) and allows to specify rules to
exclude certain URLs and URL patterns, and provides options to specify if internal links and/or
external links should be checked and other things.

Since checking all (internal and external) references is time consuming, we check as follows:

- Validate internal and external links when the
  [Regular Deployment of Main Branch](#regular-deployment-of-main-branch) is executed.
- Otherwise, validate only internal links.
