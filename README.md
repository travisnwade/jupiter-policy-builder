# InfoSec Policies, Standards and Procedures (PSP) Builder

This provides a CLI tool and library for generating and maintaining an
organization's full policies, standards, and procedures in support of modern
security operations and compliance.

It will default to outputting documentation in Markdown format. This project
also supports the use of `Mkdocs` to convert Markdown to HTML, e.g.:
https://security.lifeomic.com/psp/. `pandoc` (not supported) may optionally
be used to convert from Markdown to PDF format.

First-time users of the tool can choose one of the execution methods below
and run `psp build` to be interactively prompted for configuration values.
This will generate a `config.json` file that may be used to speed-up future
invocations.

## Building and deploying policies

### From docker zipfile

If you've received this project as a zip intended for Docker usage, simply:

  1. `unzip pspbuilder-docker.zip`
  1. `docker build -t pspbuilder .`
  1. `docker run -it -v$(pwd):/opt --rm pspbuilder psp build -c config.json`
  1. `docker run -it -v$(pwd):/opt --rm pspbuilder mkdocs build -f docker-mkdocs.yml`

This will generate a `docs` directory containing Markdown files, and a `site`
directory containing HTML files which may be statically served.

These static files may be uploaded to a webserver, served from S3, etc. To view
them locally, do:

  1. `cd site`
  1. `python3 -m http.server 8000`
  1. `open http://localhost:8000`

### Local CLI prerequisites and first steps

#### From zipfile

If you received this project as a zipfile intended for local installation,
you'll need to install the latest Node v9 executable, via:

  1. Install NVM with: `curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash`
  1. Install Node with `nvm install $(cat .nvmrc)`

#### From source

If you have cloned this repository, you'll need to:

  1. Ensure you have the [latest Node v9](https://nodejs.org/en/download/).
  1. Execute `yarn install` to install the Node package dependencies.
  1. Execute `yarn mkdocs` to install mkdocs.

*★ Note: On a fresh macOS machine run the following commands to set up MkDocs.*

~~~bash
pip install --upgrade pip
pip install virtualenv
pip install mkdocs mkdocs-material
~~~

See http://www.mkdocs.org for more info.

#### Locally building the policies

PSP-Builder is used to build markdown source files from templates, using
configurable values.

To build all markdown files, do:

`./bin/psp build`

If you have previously saved a configuration JSON file, and wish to rebuild the
PSP docs against this configuration, do:

`./bin/psp build -c path/to/your/config.json`

#### Locally serving the static site

MkDocs is used to build a static site to serve the documentation pages. To view
the site locally, run `./bin/mkdocs serve -f docs/mkdocs.yml` in the project
root directory.

For further instructions, see [MkDocs website](http://www.mkdocs.org/).

Additionally, it uses `mkdocs-material` theme.  Instructions can be found
[here](https://squidfunk.github.io/mkdocs-material/getting-started/).

*★ Note: Four spaces are used instead of two in nested lists to get around a
rendering issue in MkDocs.*

### Generating PDF and Word documents

`Pandoc` can be used to automatically convert the markdown files into PDF or
Word documents.  This requires `pandoc` to be installed separately on your local
system. Follow the installation instructions here:

[https://pandoc.org/installing.html](https://pandoc.org/installing.html)

NOTE: on OSX systems, you will likely also need to install XeLaTeX from here:
[http://www.texts.io/support/0001/](http://www.texts.io/support/0001/)

**Example script for generating individual PDF policy documents:**

```bash
#!/bin/bash
cd ./docs
mkdir pdf
for filename in *.md; do
    pandoc $filename -f markdown -t latex -o ./pdf/$filename.pdf
done
```

**Example script for generating a combined PDF policy document:**

```bash
pandoc intro.md model.md *.md -f markdown -t latex --pdf-engine=xelatex --toc -o ./pdf/infosec-policies.pdf
```

**Example script for generating Word documents:**

```bash
mkdir docx
pandoc intro.md model.md *.md -f markdown -t docx --toc -o ./docx/infosec-policies.docx
```

### Publishing policies and procedures to JupiterOne

If you have an account on the JupiterOne security platform (https://jupiterone.io),
you can run the following command to publish the contents of your policies and
procedures to your JupiterOne account, so that you and others in your organization
can access them online.

```bash
./bin/psp publish -a j1accountId -c path/to/your/config.json -t ./templates -u j1userId
```

The `publish` command will prompt you to enter the password for your JupiterOne
user account. You can also supply the API Token instead of a password with the
`-k | --api-token` option.

Your JupiterOne user must have administrator privilege to publish the contents.

### Generating Self Assessment Reports

The current version of the policy builder supports generating a lightweight
HIPAA self assessment report, based on a few key questions and the adoption of
policies and procedures.

```bash
./bin/psp assess --standard hipaa --config <location_of_your_json_config_file> [options]
```

The above command generates a HIPAA self assessment report in the
`./assessments` directory. The report contains mapping of your adopted
policies/procedures to each specific HIPAA regulation requirement. It also
contains placeholders, where applicable, for you to provide additional details.
Gaps identified will be called out in the command line output as well as in the
report itself.

## Contents

Each policy and procedure is included as its own markdown file. We recommended
all policies to be adopted for your security program. The `config.json` file
includes the procedures/controls you choose to adopt, which will be included in
the final policy docs by the `psp-builder`.

### Structure

`./templates/`

- This directory contains the modular templates for policies and procedures
- It is generated upon first run of the policy builder (`./bin/psp-builder`)
- If you need to fine tune your policy and procedures, edit the individual
  markdown files inside this directory to preserve changes for subsequent runs.

`./partials/`

- This directory contains the modular policy and procedure markdown files with
  your organizational details incorporated.
- The partials directory will be generated after you run the `psp-builder` for
  the first time.

`./docs/`

- This directory contains the policy documents with the applicable/adopted
  procedures merged in.

`./site/`

- This is the default location where `mkdocs` generates the static HTML pages
  from you policy docs.
- You can then publish the site to your desired target, such as an S3 bucket or
  an internal website.

## Advanced configuration

You may also edit your `config.json` file directly to provide input to the
configurable questions. The `config.json` file contains the following sections:

`organization`

- You can edit this section directly to provide answers to the questions that
  are promptly by the `psp-builder` CLI.
- You can also add your own custom variables in this section -- e.g.
  `"variableName": "value"`. You will need to make sure to add the same
  variable(s) to your templates in the format of `{{variableName}}`.
- Note that if the variable is a URL/URI, you will need to add the `&` symbol to
  your variables in the templates -- e.g. `{{&variableURL}}`.

`standards`

- Contains references to various compliance standards and frameworks.
- **DO NOT** edit this section.

`policies`

- Contains all available policies and the corresponding procedures that
  implements and enforces each policy.
- It is recommended that all policies to be included in your security program
  and documentation, therefore, you should **NOT** edit this section.

`procedures`

- Contains the individual procedure documentations.
- Each procedure includes the following

  - A `summary` to provide high level guidance about that particular procedure
    and its implementation. This is for your reference.
  - A set of `resources` that will help with implementation and/or selecting a
    third party vendor solution.
  - A `provider` property, pre-populated with the recommended solution, if
    applicable. You may update this to the solution you have actually selected.
    The policy builder will update the documentation text within that procedure
    accordingly.
  - If you choose to exclude a procedure from your final policy documentation,
    you may set the `adopted` flag to `false`. The policy builder with skip
    those when compiling the policies.
