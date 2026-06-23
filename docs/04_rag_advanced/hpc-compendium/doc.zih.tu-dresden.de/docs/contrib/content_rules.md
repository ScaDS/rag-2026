# Content Rules

!!! cite "George Bernard Shaw"

    The golden rule is that there are no golden rules.

## Motivation and Rationale

This page holds rules regarding the layout, content, and writing of this
documentation. The goals are to provide a comprehensive, consistent and well-written
documentation that is pure joy to read and use. It shall help to find answers and provide knowledge
instead of being the bottleneck and a great annoyance. Therefore, we set up some rules which
are outlined in the following.

!!! tip

    Following these rules when contributing speeds up the review process. Furthermore, your
    changes will not be blocked by the automatic checks implemented in the CI pipeline.

## Responsibility and License

This documentation and the repository have two licenses (cf. [Legal Notice](../legal_notice.md)):

* All documentation is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
* All software components are licensed under [MIT license](../license_mit.txt).

These licenses also apply to your contributions.

!!! note

    If you contribute, you are fully and solely responsible for the content you create and have to
    ensure that you have the right to create it under the laws which apply.

If you are in doubt, please contact us either via
[GitLab Issue](https://gitlab.hrz.tu-chemnitz.de/zih/hpcsupport/hpc-compendium/-/issues)
or via [e-mail](mailto:hpc-support@tu-dresden.de).

## Quick Overview

* All documentation is written in [Markdown](#markdown).
* Use spaces (not tabs) both in Markdown files and in `mkdocs.yml`.
* Respect the line length limit of 100 characters (exception: links).
* Do not add large binary files or high-resolution images to the repository (cf.
   [adding images and attachments](#graphics-and-videos)).
* [Admonitions](#special-feature-admonitions) may be actively used for longer code examples,
   warnings, tips, important information, etc.
* Respect the [writing style](#writing-style) and the rules for
  [spelling and technical wording](#spelling-and-technical-wording).
* For code blocks:
    * Use [syntax highlighting and appropriate prompts](#code-blocks-and-command-prompts).
    * Respect [data privacy](#data-privacy-and-generic-names).
    * Stick to the [rules on optional and required arguments](#code-styling-rules).
* Save attachments, graphics and videos within the respective `misc` subdirectory.

## Detailed Overview

### Writing Style

* Assume that a future reader is eager to start typing commands. Thus, encourage the reader by
  addressing him/her directly:
    * Example: Use "You can/should ..." instead of "Users can/should ..."
    * Example: Use "Your contribution is highly welcome" instead of "Contributions from user-side
      are highly welcome"
* Be brief! Provide the main idea/commands first, and special cases later. If it is not necessary to
  know how a special piece of software works, don't explain it.
* Provide the often-used commands first.
* Use active over passive voice
    * Write with confidence. This confidence should be reflected in the documentation so that
      the readers trust and follow it.
    * Example: "We recommend something" instead of "Something is recommended."
* Capitalize headings, e.g. *Exclusive Reservation of Hardware*
* Give keywords in link texts, e.g. [Code Blocks](#code-blocks-and-syntax-highlighting) is more
  descriptive than [this subsection](#code-blocks-and-syntax-highlighting)
* Avoid using tabs both in Markdown files and in `mkdocs.yaml`. Type spaces instead.

### Pages Structure and New Page

The documentation and pages structure is defined in the configuration file
[`mkdocs.yml`](https://gitlab.hrz.tu-chemnitz.de/zih/hpcsupport/hpc-compendium/-/blob/main/doc.zih.tu-dresden.de/mkdocs.yml):

```Markdown
nav:
  - Home: index.md
  - Application for Login and Resources:
    - Overview: application/overview.md
    - Terms of Use: application/terms_of_use.md
    - Request for Resources: application/request_for_resources.md
    - Project Request Form: application/project_request_form.md
    - Project Management: application/project_management.md
    - Acknowledgement: application/acknowledgement.md
  - Access to ZIH Systems:
    - Overview: access/overview.md
  [...]
```

Follow these two steps to **add a new page** to the documentation:

1. Create a new Markdown file under `docs/subdir/file_name.md` and put the documentation inside.
The sub-directory structure represents different topics of the documentation. Try to fit the
contribution into the existing structure. The file name should reflect the title of the
documentation page, i. e. `shortened_page_heading.md`.
1. Add `subdir/file_name.md` to the configuration file `mkdocs.yml` by updating the navigation
   section. Yes, the order of files is crucial and defines the structure of the content. Thus,
   carefully consider the right spot and section for the new page.

Make sure that the new page **is not floating**, i.e., it can be reached directly from
the documentation structure.

### Outdated Pages

Sometimes, whole pages become irrelevant, e. g., a page that describes hardware of a cluster that
is switched off. In such cases, move the page to the archive, using the following steps:

1. Move the file to directory `archive`.
1. In the moved file, add the following to the beginning of the file:

    ```Markdown
    ---
    search:
      boost: 0.00001
    ---
    ```

    If the file already has a similar header, just update the number.

1. Append `(Outdated)` to the topmost heading in the file.
1. Update links to the moved file in all other files.
1. Update links in the moved file to other files.
1. Move the corresponding navigation entry in `mkdocs.yml` somewhere below the `Archive` entry.

### Markdown

All documentation is written in Markdown. Please keep things simple, i.e., avoid using fancy
Markdown dialects.

#### Brief How-To on Markdown

| Purpose | Markdown | Rendered HTML |
|---|---|---|
| Bold text | `**Bold Text**`  | **Bold Text**  |
| Italic text | `*Italic Text*`   | *Italic Text*  |
| Headings | `# Level 1`, `## Level 2`, `### Level 3`, ...   |  |
| External link | `[website of TU Dresden](https://tu-dresden.de)` | [website of TU Dresden](https://tu-dresden.de) |
| Internal link | `[Slurm page](../jobs_and_resources/slurm.md)` | [Slurm page](../jobs_and_resources/slurm.md)|
| Internal link to section | `[section on batch jobs](../jobs_and_resources/slurm.md#batch-jobs)` | [section on batch jobs](../jobs_and_resources/slurm.md#batch-jobs) |

Further tips can be found on this
[cheat sheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).

#### Attachments

Of course, you can provide attachments in sections and pages.
Such attachment documents may contain information that are more detailed and go far beyond the
scope of the compendium, e.g. user manuals for application-specific software.

Save attachments within the `misc` subdirectory of the corresponding section.

!!! note "Syntax for attachments"

    The syntax for attachments is the very same as for links. As the attachment is within the `misc`
    subdirectory, you can refer to it as local file.

    ```markdown
    [<description>](misc/<attachment_file_name>)
    ```

    Since the `<description>` is rendered as link text, you should choose a clear and precise text:

    ```markdown
    [slides of HPC introduction](misc/HPC-Introduction.pdf)
    ```

#### Graphics and Videos

Please use graphics and videos for illustration purposes and to improve comprehensibility.
All graphics and attachments are saved within `misc` directory of the respective subdirectory in
`docs`.
For video attachments please use either webm or mp4 format. We make use of the
[mkdocs-video extension](https://github.com/soulless-viewer/mkdocs-video).

!!! note "Syntax for graphics and videos"

    The syntax to insert a **graphic** into a page is

    ```markdown
    ![Alternative text](misc/graphics_file.png)
    ```

    The syntax to insert a **video** attachment into a page is

    ```html
    ![type:video](misc/terminate-virtual-desktop-dcv.mp4)
    ```

It is possible to add captions for tables and figures using `{: summary="This is a table caption"}`.
The `summary` and `align` parameters can be combined as well:
`{: summary="This is a table caption" align="top"}`.

##### Resizing and Alignment of Graphics

In general, graphics and images should be added to the repository with the desired size.

!!! warning

    Do not add large binary files or high-resolution images to the repository. See this valuable
    document for [image optimization](https://web.dev/fast/#optimize-your-images).

We recommend the well-know Linux package [ImageMagick](https://imagemagick.org/) for resizing
graphics.

!!! example "Resize image using ImageMagick"

    The command

    ```console
    marie@local$ magick cluster.jpeg -resize 600 cluster_600.jpeg
    ```

    will resize the graphic `cluster.jpeg` to a width of 600 pixels keeping the aspect ratio.
    Depending on the resolution of the original file, the resulting file can be way smaller in terms
    of memory foot print.

Nevertheless you can explicitly specify the size a graphic. The syntax is as follows

```markdown
![Alternative text](misc/graphics_file.png){: style="width:150px"}
```

By default, graphics are left-aligned. In most cases, this is not elegant and you probably wish to
center-align your graphics. **Alignment** of graphics can be controlled via the `{: align=<value>}`
attribute. Possible values are `left`, `right` and `center`. **Note:** It is crucial to
have `{: align=center}` on a new line and the value without quotation marks.

Resize and alignment specification can be combined as depicted in the following example.

!!! example "Resize image to 150px width and specify alignment"

    The three tabs show the Markdown syntax to resize the image of the beautiful
    [cluster `Barnard`](../jobs_and_resources/hardware_overview.md#barnard) to a height of 150
    pixels keeping the aspect ratio and left, center and right-align it, respectively.

    === "Scale and default-align"

        ```markdown
        ![Beauty Barnard](misc/barnard.jpeg){: style="height:150px"}
        ```

        ![Beauty Barnard](misc/barnard.jpeg){: style="height:150px"}

    === "Scale and center-align"

        ```markdown
        ![Beauty Barnard](misc/barnard.jpeg){: style="height:150px"}
        {: align="center"}
        ```

        ![Beauty Barnard](misc/barnard.jpeg){: style="height:150px"}
        {: align="center"}

    === "Scale and right-align"

        ```markdown
        ![Beauty Barnard](misc/barnard.jpeg){: style="height:150px"}
        {: align="right"}
        ```

        ![Alternative text](misc/barnard.jpeg){: style="height:150px"}
        {: align="right"}

#### Special Feature: Admonitions

[Admonitions](https://squidfunk.github.io/mkdocs-material/reference/admonitions/), also known as
call-outs, may be actively used to highlight examples, warnings, tips, important information, etc.
Admonitions are an excellent choice for including side content without significantly interrupting
the document flow.

Several different admonitions are available within the used
[material theme](https://squidfunk.github.io/mkdocs-material/), e.g., `note`, `info`, `example`,
`tip`, `warning`, and `cite`. Please refer to the
[documentation page](https://squidfunk.github.io/mkdocs-material/reference/admonitions/#supported-types)
for a comprehensive overview.

!!! example "Syntax"

    All admonitions blocks start with `!!! <type>` and the following content block is indented by
    (exactly) four spaces.
    If no title is provided, the title corresponds to the admonition type.

    ```markdown
    !!! note "Descriptive title"

        Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod
        tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At
        vero eos et accusam et justo duo dolores et ea rebum.
    ```

!!! note Folding

    Admonitions can be made foldable by using `???` instead of `!!!`. A small toggle on the right
    side is displayed. The block is open by default if `???+` is used. Long code examples should be
    folded by default.

### Spelling and Technical Wording

To provide consistent and high-quality documentation, and help users to find the right pages,
there is a list of conventions w.r.t. spelling and technical wording.

* Language settings: en_us

| Do | Don't |
|----|-------|
| I/O | IO |
| Slurm | SLURM |
| filesystem(s) | file system(s) |
| ZIH system(s) | Taurus, HRSK II, our HPC systems, etc. |
| workspace | work space |
|       | HPC-DA |
| cluster `romeo` | ROMEO cluster, romeo cluster, `romeo` cluster, "romeo" cluster, etc. |

### Code Blocks and Command Prompts

* Use ticks to mark code blocks and commands, not an italic font.
* Specify language for code blocks ([see below](#code-blocks-and-syntax-highlighting)).
* All code blocks and commands should be runnable from a login node or a node within a specific
  cluster (e.g., `alpha`).
* It should be clear from the [prompt](#list-of-prompts), where the command is run (e.g., local
  machine, login node, or specific cluster).

#### Code Blocks and Syntax Highlighting

Providing code blocks and snippets is the meat and bones of this documentation.
Code blocks and command examples should give the general idea of invocation and be as precise as
possible, i.e., allowing for copy-and-paste. Please mark replaceable code parts and optional and
required arguments as outlined in the section [required and optional arguments](#code-styling-rules)
below. Long, non-meaningful output should be omitted.

We make use of the extension
[pymdownx.highlight](https://squidfunk.github.io/mkdocs-material/reference/code-blocks/) for syntax
highlighting. There is a complete list of supported
[language short codes](https://pygments.org/docs/lexers/).

??? note "Syntax for command line"

    For normal commands executed in the terminal, use the language short code `console`.

    ````markdown
    ```console
    marie@login$ module list
    [...]
    ```
    ````

??? note "Syntax for job files and scripts"

    Use the language short code `bash` for job files and shell scripts.

    ````markdown
    ```bash
    #!/bin/bash
    #SBATCH --nodes=1
    #SBATCH --time=01:00:00
    #SBATCH --output=slurm-%j.out

    module load foss

    srun a.out
    ```
    ````

??? note "Syntax for Python"

    `python` for Python source code:

    ````markdown
    ```python
    from time import gmtime, strftime
    print(strftime("%Y-%m-%d %H:%M:%S", gmtime()))
    ```
    ````

    And `pycon` for Python console:

    ````markdown
    ```pycon
    >>> from time import gmtime, strftime
    >>> print(strftime("%Y-%m-%d %H:%M:%S", gmtime()))
    2021-08-03 07:20:33
    ```
    ````

??? note "Line numbers"

    More sugar can be applied by adding line numbers.

    ````markdown
    ```bash linenums="1"
    #!/bin/bash

    #SBATCH --nodes=1
    #SBATCH --ntasks=23
    #SBATCH --time=02:10:00

    srun a.out
    ```
    ````

    _Result_:

    ![lines](misc/lines.png)

    Specific Lines can be highlighted by using

    ````markdown
    ```bash hl_lines="2 3"
    #!/bin/bash

    #SBATCH --nodes=1
    #SBATCH --ntasks=23
    #SBATCH --time=02:10:00

    srun a.out
    ```
    ````

    _Result_:

    ![lines](misc/highlight_lines.png)

#### Data Privacy and Generic Names

Where possible, replace login, project name, and other private data with clearly recognizable
placeholders. In particular, use the generic placeholders depicted in the following table.
The table also holds a second placeholder, if, e.g., you need a second login to formulate an example.

| Description | Placeholder | 2nd Placeholder |
|---|---|---|
| Username | Marie | Martin |
| Login | `marie` | `martin` |
| E-mail | marie@tu-dresden.de | martin@tu-dresden.de |
| Project title | `p_number_crunch` | `p_long_computations` |
| Workspace title | `number_crunch` | `long_computations` |
| Job ID | `123456` | `456789` |
{: summary="Generic placeholders", align="bottom"}

!!! example "Output of `ls` command"

    The following code listing depicts the usage of the generic user names and projects as well as
    recognizable placeholders for files and directory names.

    ```console
    marie@login$ ls -l
    drwxr-xr-x   3 marie p_number_crunch      4096 Jan 24  2020 code
    drwxr-xr-x   3 marie p_number_crunch      4096 Feb 12  2020 data
    -rw-rw----   1 marie p_number_crunch      4096 Jan 24  2020 readme.md
    ```

!!! info "Marie"

    We choose *marie* as generic login and placeholder. There is no magic story on this decision.
    Feel free to associate this generic login for example with
    physicist and chemist [Marie Curie](https://en.wikipedia.org/wiki/Marie_Curie),
    and [Marianne](https://en.wikipedia.org/wiki/Marianne), symbol of France standing for liberty,
    equality and fraternity.

    The very same holds for the generic login *martin*.

#### Placeholders

Placeholders represent arguments or code parts that can be adapted to the user's needs. Use them to
give a general idea of how a command or code snippet can be used, e. g. to explain the meaning of
some command argument:

```bash
marie@login$ sacct -j <job id>
```

Here, a placeholder explains the intention better than just a specific value:

```console
marie@login$ sacct -j 4041337
```

Please be aware, that a reader often understands placeholders easier if you also give an example.
Therefore, always add an example!

#### Mark Omissions

If showing only a snippet of a long output, omissions are marked with `[...]`.

#### Code Styling Rules

* Stick to the Unix rules on optional and required arguments, and selection of item sets:

    * `<required argument or value>`
    * `[optional argument or value]`
    * `{choice1|choice2|choice3}`

* Please use following style guidelines while writing code blocks:

    * Shell: [Shell style guide](https://google.github.io/styleguide/shellguide.html)
    * Python: [PEP-0008 style guide](https://peps.python.org/pep-0008/)
    * MATLAB: [MATLAB programming style guide](https://www.researchgate.net/publication/316479241_Best_practices_for_scientific_computing_and_MATLAB_programming_style_guidelines)
    * R: [R style guide](https://google.github.io/styleguide/Rguide.html)
    * C++: [C++ style guide](https://google.github.io/styleguide/cppguide.html)
    * Java: [Java style guide](https://google.github.io/styleguide/javaguide.html)

#### List of Prompts

We follow these rules regarding prompts to make clear where a certain command or example is executed.
This should help to avoid errors.

| Host/Partition         | Prompt           |
|------------------------|------------------|
| Localhost              | `marie@local$`   |
| Login nodes            | `marie@login$`   |
| Arbitrary compute node | `marie@compute$` |
| Compute node `Capella` | `marie@capella$` |
| Login node `Capella`   | `marie@login.capella$`  |
| Compute node `Barnard` | `marie@barnard$` |
| Login node `Barnard`   | `marie@login.barnard$`  |
| Compute node `Alpha`   | `marie@alpha$`   |
| Login node `Alpha`     | `marie@login.alpha$`   |
| Node `Julia`           | `marie@julia$`   |
| Compute node `Romeo`   | `marie@romeo$`   |
| Login node `Romeo`     | `marie@login.romeo$`   |
| Partition `dcv`        | `marie@dcv$`     |

* **Always use a prompt**, even if there is no output provided for the shown command.
* All code blocks which specify some general command templates, e.g. containing `<` and `>`
  (see [placeholders](#placeholders) and [Code Styling Rules](#code-styling-rules)), should use
  `bash` for the code block. Additionally, an example invocation, perhaps with output, should be
  given with the normal `console` code block. See also
  [Code Block description below](#code-blocks-and-syntax-highlighting).
* Using some magic, the prompt as well as the output is identified and will not be copied!
* Stick to the [generic user name](#data-privacy-and-generic-names) `marie`.

#### Long Options

The general rule is to provide long over short parameter names where possible to ease
understanding. This holds especially for Slurm options, but also other commands.

??? example

    | Do | Don't |
    |----|-------|
    | `srun --nodes=2 --ntasks-per-node=4 [...]`| `srun -N 2 -n 4 [...]` |
    | `module load [...]` | `ml [...]` |

#### Equal Signs in Command-Line Options

Some tools with CLI (command-line interface) prefer specification of the argument with an equal
sign (`=`) between the option name and the value, e.g. `--long_option=value`. Others prefer a
whitespace, e.g. `--long_option value`. We respect the design decisions of the tool
developers and document the desired mimic for [long options](#long-options). If you are in doubt,
calling `tool --help` might provide the answer.

| Tool  | Preference             |
|-------|------------------------|
| Slurm | w/ Equal sign          |
| HPC-Workspace | w/o equal sign |

### Customize Search

The
[documentation for the search plugin](https://squidfunk.github.io/mkdocs-material/setup/setting-up-site-search/)
of the material theme is quite comprehensive. The search is realized as client-side search using the
open-source tool on [lunr](https://lunrjs.com/). The ranking of pages in search results bases on
so-called scoring. Please refer to
[lunrjs documentation](https://lunrjs.com/guides/searching.html#scoring) for details.

From time to time it might be necessary to **tweak the search priority of certain pages**.
For example, pages from the archive section should be ranked very low in search results. This can be
achieved by adding the front matter `search.boost` property added to the top of the Markdown file of
interest:

```Markdown
---
search:
  boost: 2
---

# Document Title

[...]
```

The documentation of this plugin gives no range for the boost values. We recommend to use this
feature carefully starting with low values.
