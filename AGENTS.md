# AGENTS.md

This repository contains AWX (Ansible Tower) configuration files. Agents should follow these guidelines when operating here.

## Build/Lint/Test Commands

- **Install dependencies**: `make requirements_dev` or `pip install -r requirements/requirements_dev.txt`
- **Build**: `make docker-compose-build` (Docker-based development environment)
- **Linting**: `make api-lint` (includes black, flake8, yamllint) or `make black` for Python formatting only
- **Testing**: `make test` for all unit tests, `make test_coverage` for coverage, `make test_tox` for multi-Python version testing
- **Single test**: `py.test awx/main/tests/unit/specific_test.py::TestClass::test_method` or `make test_unit` for unit tests only

## Code Style Guidelines

- **Python formatting**: Black with 160 character line length, fast mode, string normalization skipped, excludes awx_collection
- **Linting**: Flake8 with specific error codes (F401,F402,F821,F823,F841,F811,E265,E266,F541,W605,E722,F822,F523,W291,F405)
- **YAML**: yamllint with line-length disabled, truthy checks disabled
- **Imports**: Grouped as Python stdlib, third-party packages, Django, Django REST Framework, AWX internal
- **Naming**: snake_case for variables/functions, PascalCase for classes, UPPER_CASE for constants
- **Error handling**: Use Django REST framework ValidationError, proper exception handling with try/except
- **Types**: Use type hints where appropriate, follow Django model conventions
- **Comments**: Minimal comments, focus on self-documenting code
- **Security**: Never commit secrets, use environment variables for sensitive data

## Development Workflow

- **Pre-commit hooks**: Automatically run black formatting on Python files
- **Testing**: Write unit tests for new functionality, update tests for bug fixes
- **Documentation**: Update docs for API changes, follow ReStructuredText format
- **Commits**: Use `git commit --signoff`, keep history clean with rebase not merge

No Cursor or Copilot rules found. Follow AWX best practices.</content>
<parameter name="filePath">AGENTS.md