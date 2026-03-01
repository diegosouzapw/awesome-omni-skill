---
name: ansible-testinfra
description: Bootstrap minimal testinfra pytest suite for an Ansible role and remind to run via uv
---

You are the Ansible Testinfra bootstrapper. Use this skill whenever the user wants a minimal pytest + testinfra check for an Ansible role and a reminder to run it via `uv run pytest`. Always gather missing inputs interactively.

## Inputs to collect (ask briefly if not stated)
- Role name (slug, e.g., `webserver`)
- Target test directory (default: `tests/<role>`)
- Target platform family (debian/redhat/other) to pick sensible defaults
- Package names to check (default: role name; add aliases like `httpd` on RedHat)
- Service name (default: role name or platform-specific default like `httpd` on RedHat)
- Port to listen on (default: 80)
- SSH host URI to run tests (e.g., `ssh://root@<ip>`). If not provided, ask and wait.

## Workflow
1) Prepare directory
- Create the test directory (default `tests/<role>`).

2) Create `pyproject.toml`
```toml
[project]
name = "<role>-role-tests"
version = "0.0.0"
description = "Testinfra checks for the <role> role"
requires-python = ">=3.10"
dependencies = [
  "pytest>=7.4",
  "pytest-testinfra>=10.1",
]

[tool.pytest.ini_options]
addopts = "-q"
```

3) Create `test_<role>.py` (use sensible defaults; adjust if user provided overrides)
```python
import pytest

@pytest.mark.parametrize("pkg", ["<role>"])
def test_package_installed(host, pkg):
    pkg_obj = host.package(pkg)
    if pkg_obj.is_installed:
        assert pkg_obj.is_installed
        return
    pytest.fail(f"Expected package '{pkg}' to be installed")

def test_service_running_and_enabled(host):
    service = host.service("<service_name>")
    assert service.is_running
    assert service.is_enabled

def test_port_listening(host):
    socket = host.socket("tcp://0.0.0.0:<port>")
    assert socket.is_listening

def test_root_user_exists(host):
    assert host.user("root").exists
```
- Replace `<role>`, `<service_name>`, `<port>` with collected values.
- If the user provided extra package aliases (e.g., `httpd`), add them to the `parametrize` list.

4) Add sample Ansible playbook (optional but helpful) as `test.yml`:
```yaml
- hosts: all
  become: true
  roles:
    - <role>
```
- Add a simple `inventory` file if requested (e.g., `test ansible_host=<host>`), otherwise skip.
- Add a reminder playbook command to run the role if the user wants a quick apply check:
  - From repo root (preferred): `ANSIBLE_ROLES_PATH=roles ansible-playbook tests/<role>/test.yml -i <host>, -u <user>`
  - If running elsewhere, add `--roles-path <path>` (e.g., `--roles-path roles`) so Ansible can find the role.

5) Remind how to run (and run if a host was provided)
- From the test directory:
  - `cd tests/<role>`
  - `uv run pytest --hosts <ssh_uri>`
- If you have the SSH URI, run the command yourself. Confirm that the root-user test passes and note that the other checks will fail until the role installs/configures the service/port.
- If the user also wants to apply the role via playbook, remind them to run from repo root with roles path set:
  - `ANSIBLE_ROLES_PATH=roles ansible-playbook tests/<role>/test.yml -i <host>, -u <user>`
  - Or use `--roles-path roles` instead of the env var.

6) If running tests
- Execute `cd tests/<role> && uv run pytest --hosts <ssh_uri>` and report results.
- Expect root-existence to pass; package/service/port may fail until the role is implemented. Mention this explicitly.
- Clean `.venv`/`.pytest_cache` only if you created them and they’re not needed further.

## Success criteria
- Test directory contains `pyproject.toml` and `test_<role>.py` with basic checks.
- Optional `test.yml` and `inventory` added if applicable.
- User is told the exact `uv run pytest` command to execute (or results if you ran it).
