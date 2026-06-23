# Contribute via Local Clone

In the following, it is outlined how to contribute to the
[HPC documentation](https://compendium.hpc.tu-dresden.de/) of
[TU Dresden/ZIH](https://tu-dresden.de/zih/) via a local clone of the Git repository. Although, this
document might seem very long describing complex steps, contributing is quite easy - trust us.

## Initial Setup of your Local Clone

Please follow this standard Git procedure for working with a local clone:

1. Fork the project on
[https://gitlab.hrz.tu-chemnitz.de/zih/hpcsupport/hpc-compendium](https://gitlab.hrz.tu-chemnitz.de/zih/hpcsupport/hpc-compendium)
or request access to the project.
1. Change to a local (unencrypted) filesystem. (We have seen problems running the container on an
ecryptfs filesystem. So you might want to use e.g. `/tmp` as the start directory.)
1. Clone the Git repository:
    1. `git clone git@gitlab.hrz.tu-chemnitz.de:zih/hpcsupport/hpc-compendium.git`
    1. If you forked the repository, instead use:
        - `git clone git@gitlab.hrz.tu-chemnitz.de:<YOUR_LOGIN>/hpc-compendium.git`
1. Change into the new directory:
    - `cd hpc-compendium`
1. If you forked the repository, add the original repository as a so-called remote:
    - `git remote add upstream-zih git@gitlab.hrz.tu-chemnitz.de:zih/hpcsupport/hpc-compendium.git`

## Working with your Local Clone

1. Whenever you start working on an issue, first make sure that your local data is up to date:
    1. `git checkout preview`
    1. `git pull origin preview`
    1. `git pull upstream-zih preview` (only required when you forked the project)
1. Create a new feature branch for you to work in. Ideally, name it like the file you want to
modify or the issue you want to work on, e.g.:
`git checkout -b 174-check-contribution-documentation` for issue 174 with title "Check contribution
documentation". (If you are uncertain about the name of a file, please look into `mkdocs.yaml`.)
1. Improve the documentation with your preferred editor, i.e. add new files and correct mistakes.
1. Use `git add <FILE>` to select your improvements for the next commit.
1. Commit the changes with `git commit -m "<DESCRIPTION>"`. The description should be a meaningful
description of your changes. If you work on an issue, please also add "Closes 174" (for issue 174).
1. Push the local changes to the GitLab server, e.g. with
`git push origin 174-check-contribution-documentation`.
1. As an output you get a link to create a merge request against the preview branch.
1. When the merge request is created, a continuous integration (CI) pipeline automatically checks
your contributions. If you forked the repository, these automatic checks are not available, but you
can [run checks locally](#run-the-proposed-checks-inside-container).

!!! tip

    When you contribute, please follow our [content rules](content_rules.md) to make incorporating
    your changes easy. We also check these rules via continuous integration checks and/or reviews.
    You can find the details and commands to [preview your changes](#start-the-local-web-server) and
    [apply checks](#run-the-proposed-checks-inside-container).

## Merging of Forked Repositories

When you have forked the repository as mentioned above, the process for merging is a bit different
from internal merge requests. Because branches of forks are not automatically checked by CI,
someone with at least developer access needs to do some more steps to incorporate the changes of
your MR:

1. The developer informs you about the start of merging process.
1. The developer needs to review your changes to make sure that your changes are specific and don't
   introduce problems, such as changes in the Dockerfile or any script could.
1. The developer needs to create a branch in our repository. Let's call this "internal MR branch".
1. The developer needs to change the target branch of your MR from "preview" to "internal MR branch".
1. The developer needs to merge it.
1. The developer needs to open another MR from "internal MR branch" to "preview" to check whether
   the changes pass the CI checks.
1. The developer needs to fix things that were found by CI.
1. The developer informs you about the MR or asks for your support while fixing the CI.

When you follow our [content rules](content_rules.md) and
[run checks locally](#run-the-proposed-checks-inside-container), you are making this process
faster.

## Tools to Ensure Quality

Assuming you already have a working Docker installation and have cloned the repository as mentioned
above, a few more steps are necessary.

Build the Docker image. This might take a bit longer, as `mkdocs` and other necessary software
needs to be downloaded, but you have to run it only once in a while.
Building a container could be done with the following steps:

```console
marie@local$ cd hpc-compendium
marie@local$ doc.zih.tu-dresden.de/util/download-newest-mermaid.js.sh
marie@local$ docker build -t hpc-compendium .
```

To avoid a lot of retyping, set the following Git aliases once inside your local Git clone:

```console
marie@local$ git config alias.wikiscript '!docker run --name=hpc-compendium --rm -w /docs --mount src=${PWD},target=/docs,type=bind hpc-compendium'
marie@local$ git config alias.wiki '!docker run --name=hpc-compendium -p 8000:8000 --rm -w /docs --mount src=${PWD}/doc.zih.tu-dresden.de,target=/docs,type=bind hpc-compendium'
marie@local$ git config alias.wiki-preview '!docker run --name=hpc-compendium -p 8000:8000 --rm -w /docs --mount src=${PWD}/doc.zih.tu-dresden.de,target=/docs,type=bind hpc-compendium mkdocs serve -a 0.0.0.0:8000 --livereload'
```

## Working with the Docker Container

Here is a suggestion of a workflow which might be suitable for you.

### Start the Local Web Server

The command(s) to start the dockerized web server is this:

```console
marie@local$ git wiki-preview
```

You can view the documentation via `http://localhost:8000` in your browser, now.

!!! note

    You can keep the local web server running in this shell to always have the opportunity to see
    the result of your changes in the browser. Simply open another terminal window for other
    commands.
    If you cannot see the page in your browser, check if you can get the URL for your browser's
    address bar from a different terminal window:

    ```console
    marie@local$ echo http://$(docker inspect -f "{{.NetworkSettings.IPAddress}}" $(docker ps -qf "name=hpc-compendium")):8000
    ```

You can now update the contents in you preferred editor. The running container automatically takes
care of file changes and rebuilds the documentation whenever you save a file.

With the details described below, it will then be easy to follow the guidelines for local
correctness checks before submitting your changes and requesting the merge.

### Run the Proposed Checks Inside Container

In our continuous integration (CI) pipeline, a merge request triggers the automated check of

* correct links,
* correct spelling,
* correct text format.

These checks ensure a high quality and consistency of the content and follow our
[content rules](content_rules.md). If one of them fails, the merge request will not be accepted. To
prevent this, you can run these checks locally and adapt your files accordingly.

You are now ready to use the different checks, however we suggest to try the pre-commit hook.

#### Pre-commit Git Hook

We have several checks on the Markdown sources to ensure for a consistent and high quality of the
documentation. We recommend to automatically run checks whenever you try to commit a change. In this
case, failing checks prevent commits (unless you use option `--no-verify`). This can be accomplished
by adding a pre-commit hook to your local clone of the repository. The following code snippet shows
how to do that:

```console
marie@local$ cp doc.zih.tu-dresden.de/util/pre-commit .git/hooks/
```

!!! note

    The pre-commit hook only works, if you can use Docker without using `sudo`. If this is not
    already the case, use the command `adduser $USER docker` to enable Docker commands without
    `sudo` for the current user. Restart the Docker daemons afterwards.

Read on if you want to run a specific check.

#### Linter

If you want to check whether the Markdown files are formatted properly, use the following command:

```console
marie@local$ git wiki markdownlint docs
```

#### Spell Checker

For spell-checking a single file, e.g.
`doc.zih.tu-dresden.de/docs/software/big_data_frameworks.md`, use:

```console
marie@local$ git wikiscript doc.zih.tu-dresden.de/util/check-spelling.sh doc.zih.tu-dresden.de/docs/software/big_data_frameworks.md
```

For spell-checking all files, use:

```console
marie@local$ git wikiscript doc.zih.tu-dresden.de/util/check-spelling.sh -a
```

This outputs all words of all files that are unknown to the spell checker.
To let the spell checker "know" a word, append it to
`doc.zih.tu-dresden.de/wordlist.aspell`.

#### Check Pages Structure

The script `util/check-no-floating.sh` first checks the hierarchy depth of the pages structure and
the second check tests if every Markdown file is included in the navigation section of the
`mkdocs.yaml` file. Invoke it as follows:

```console
marie@local$ git wikiscript doc.zih.tu-dresden.de/util/check-no-floating.sh doc.zih.tu-dresden.de
```

#### Link Checker

!!! quote "Unknown programmer"

    No one likes dead links.

Therefore, we check the internal and external links within the Markdown source files.
With the script `doc.zih.tu-dresden.de/util/check-links.sh`, you can either check
[a single file](#single-file), [all modified files](#all-modified-files) or
[all files](#all-files) of the compendium.

##### Single File

To check the links within a single file, e.g.
`doc.zih.tu-dresden.de/docs/software/big_data_frameworks.md`, use:

```console
marie@local$ git wikiscript doc.zih.tu-dresden.de/util/check-links.sh doc.zih.tu-dresden.de/docs/software/big_data_frameworks.md
```

##### All Modified Files

The script can also check the links in all modified files, i.e., Markdown files which are part
of the repository and different to the `preview` branch. Use this script before committing your
changes to make sure your commit passes the CI/CD pipeline.

```console
marie@local$ git wikiscript doc.zih.tu-dresden.de/util/check-links.sh -c
```

##### All Files

Checking the links of all Markdown files takes a moment of time:

```console
marie@local$ git wikiscript doc.zih.tu-dresden.de/util/check-links.sh -a
```
