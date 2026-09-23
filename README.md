# Building https://der-control-modules.github.io

This repository contains documentation using MkDocs for [der-control-modules.github.io](https://der-control-modules.github.io). Follow these steps to set up and build the MkDocs documentation locally.

## Prerequisites

- Python 3.x installed
- Pip package manager

## Installation Steps

### 1. Clone the Repository

Clone this repository to your local machine:

```bash
git clone https://github.com/der-control-modules/der-control-modules.github.io
cd your-repo
```

### 2. Install Requirements

Install requirements using pip:

```bash
pip install -r requirements.txt
```

### 3. Preview the Documentation Locally

To view the documentation locally:

```bash
mkdocs serve
```

This command starts a development server and provides a local preview of the documentation. By default, it will be available at http://127.0.0.1:8000/.

### 4. Make Changes

Create a new branch/fork for changes that you make.  Edit the Markdown files in the docs directory to update the documentation content. Changes made to these files will be reflected in the local preview automatically.
Create a pull request to the main branch for inclusion in the site.

## Site Structure

The documentation pages live in the `docs` directory and are organized by `mkdocs.yml`:

| Page                                    | Content                                                                                  |
|-----------------------------------------|------------------------------------------------------------------------------------------|
| `docs/index.md`                         | Introduction, background, problem definition, and proposed solution.                     |
| `docs/interoperability-framework.md`    | Overview of the framework components.                                                    |
| `docs/interoperability-service.md`      | The Interoperability Service (protocol-agnostic mapping and transform service).          |
| `docs/interoperability.md`              | The original Interoperability Agent. **Deprecated**; retained for reference only.        |
| `docs/rt-control.md`, `docs/scheduler.md` | Control components.                                                                    |
| `docs/openfmb.md`                       | OpenFMB and its integration with the framework via message bus adapters.                 |
| `docs/test-cases.md`                    | Test cases, experimentation results, and the ongoing industry test setup.                |

Figures are stored in `docs/images`. The source presentation and figures used on the site are maintained in the
`presentation` folder of the `der-control` workspace.
