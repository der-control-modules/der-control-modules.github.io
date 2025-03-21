# Python Code Formatting

In Python, adhering to consistent code formatting standards is crucial for readability and maintainability.
`yapf` is one of the tools that helps automate this process while following PEP8 guidelines.

## Using `yapf` for Python Code Formatting

[`yapf`](https://github.com/google/yapf) is a Python code formatter developed by Google that applies
opinionated formatting rules to your code. It's highly configurable and follows PEP8 conventions.

### Installation

You can install `yapf` via pip:

```bash
pip install yapf
```

In vscode, you can install the yapf formatter

<https://marketplace.visualstudio.com/items?itemName=eeyore.yapf>

## Benefits of Using yapf

- Consistency: yapf ensures a consistent code style throughout your project, reducing merge conflicts and readability issues.
- PEP8 Compliance: The tool follows PEP8 guidelines, ensuring code consistency across different contributors.
- Customization: While yapf comes with default settings, it allows configuration according to specific project needs.

## Yapf configuration

Our python projects use either a pyproject.toml file or a yapf style file for configuration settings.

### Settings for pyproject.toml file

```toml
[tool.yapfignore]
ignore_patterns = [
    ".venv/**",
    ".pytest_cache/**",
    "dist/**",
    "docs/**"
]

[tool.yapf]
based_on_style = "pep8"
spaces_before_comment = 4
column_limit = 120
split_before_logical_operator = true
```

### Settings for .style.yapf

```ini
[style]
based_on_style = pep8
spaces_before_comment = 4
column_limit = 120
split_before_logical_operator = true

[exclude]
patterns = [
    ".venv/**",
    ".pytest_cache/**",
    "dist/**",
    "docs/**"
]
```
