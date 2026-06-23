# How-To Contribute

!!! cite "Chinese proverb"

    Ink is better than the best memory.

In this section you will find information about the technical setup of this documentation, the
content rules that apply, the Git workflow, and specific ways to contribute.

Your contributions are highly welcome. This can range from fixing typos, improving the phrasing and
wording to adapting examples, command lines and adding new content. Our goal is to provide a
general, consistent and up to date documentation. Thus, it is by no means a static documentation.
Moreover, is is constantly reviewed and updated.

## Technical Setup

This documentation is written in Markdown and translated into static html pages using
[mkdocs](https://www.mkdocs.org/). The single configuration file `mkdocs.yml` contains the page
structure as well as the specification of the theme and extensions.

We manage all essential files (Markdown pages, graphics, configuration, theme, etc.) within a
[public Git repository](https://gitlab.hrz.tu-chemnitz.de/zih/hpcsupport/hpc-compendium),
allowing for collaborative working and revision control. GitLab's features offer different
possibilities of contribution and ensure up-to-date and consistent content by including a review
process. There are three possible ways how you can contribute to this documentation.
These are described below.

!!! tip "Before you start"

    Before you start your very first commit, please make sure that you are familiar with our
    [Git workflow](#git-workflow) and that you have at least skimmed through the
    [Content Rules](content_rules.md).

## Git Workflow

We employ a so-called Git feature workflow with a development branch. In our case, the working branch
is called `preview` and is kept in parallel to the `main` branch.

All contributions, e.g., new content, improved wording, fixed typos, etc., are added to separate
feature branches which base on `preview`. If the contribution is ready, you will have to create a
merge request back to the `preview` branch. A member of the ZIH team will review the changes
(four-eyes principle) and finally merge your changes to `preview`. All contributions need to pass
through the CI pipeline consisting of several checks to ensure compliance with the content rules.
Please, don't worry too much about the checks. The ZIH staff will help you with that. You can find
more information about the [CI/CD pipeline](#cicd-pipeline) in the eponymous subsection.

In order to publish the updates and make them visible in the compendium,
the changes on `preview` branch are either automatically merged into the `main` branch on every
Monday via a pipeline schedule, or manually by admin staff. Moreover, the `main` branch is deployed
to [https://compendium.hpc.tu-dresden.de](https://compendium.hpc.tu-dresden.de) and always reflects
a production-ready state. Manual interventions are only necessary in case of merge conflicts.
This process is handled by the admins.

???+ note "Graphic on Git workflow"

    The applied Git workflow is depicted in the following graphic. Here, two feature branches `foo`
    and `bar` are created basing on `preview`. Three individual commits are added to branch `foo`
    before it is ready and merged back to `preview`. The contributions on `bar` consist only one
    commit. In the end, all contribution are merged to the `main` branch.

    ```mermaid
    %% Issues:
    %% - showCommitLabel: false does not work; workaround is to use `commit id: " "`%%
    %% - Changing the theme does not effect the rendered output. %%
    %%{init: { 'logLevel': 'debug', 'theme': 'base', 'gitGraph': {'showCommitLabel': false} }%%
    gitGraph
        commit
        branch preview
        checkout preview
        commit
        branch foo
        checkout foo
        commit
        commit
        checkout preview
        branch bar
        checkout bar
        commit
        checkout preview
        merge bar
        checkout foo
        commit
        checkout preview
        merge foo
        checkout main
        merge preview
    ```

## Content Rules

To ensure a high-quality and consistent documentation, and to make it easier for readers to
understand all content, we have established [Content rules](content_rules.md). Please follow
these rules regarding Markdown syntax and writing style when contributing! Furthermore, reviewing
your changes takes less time and your improvements appear faster on the official documentation.

!!! note

    If you contribute, you are fully and solely responsible for the content you create and have to
    ensure that you have the right to create it under applicable laws.

## Contribute via Issue

You can contribute to the documentation using
[GitLab's issue tracking system](https://gitlab.hrz.tu-chemnitz.de/zih/hpcsupport/hpc-compendium/-/issues).
For that, open an issue to report typos and missing documentation or request for more precise
wording etc. ZIH staff will get in touch with you to resolve the issue and improve the
documentation.

??? tip "Create an issue in GitLab"

    ![GIF showing how to create an issue in GitLab](misc/create_gitlab_issue.gif)
    {: align=center}

!!! warning "HPC support"

    Non-documentation issues and requests need to be send as ticket to
    [hpc-support@tu-dresden.de](mailto:hpc-support@tu-dresden.de).

## Contribute via Web IDE

If you have a web browser (most probably you are using it to read this page) and want to contribute
to the documentation, you are good to go. GitLab offers a rich and versatile web interface for
working with repositories. To start fixing typos and edit source files, you can find more
information on the page [Contributing via web browser](contribute_browser.md).

## Contribute via Local Clone

For experienced Git users, we provide a Docker container that includes all the checks of the CI
engine used in the backend. Using them should ensure that merge requests are not blocked due to
automatic checks.  The page [Contributing via local clone](contribute_container.md) provides you
with the details about how to set up and use your local clone of the repository.

## CI/CD Pipeline

All contributions need to pass through the CI pipeline which consists of various checks to ensure
that the [content rules](content_rules.md) have been followed.

The stages of the CI/CD pipeline are defined in a `.gitlab.yaml` file. For security reasons, this
file is maintained in a second, private repository.
