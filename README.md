# install_pip

![ansible-lint workflow](https://github.com/straysheep-dev/ansible-role-install_pip/actions/workflows/ansible-lint.yml/badge.svg)

An Ansible Role that installs [pip](https://pip.pypa.io), and pip packages.

> [!NOTE]
> **About this Fork**
>
> The original source for this role is here: [geerlingguy/ansible-role-pip](https://github.com/geerlingguy/ansible-role-pip)
>
> The changes (so far) are:
>
> - Extending the optional variables for `ansible.builtin.pip` under `defaults/main.yml`
> - Matching the formatting of the roles in [straysheep-dev/ansible-configs](https://github.com/straysheep-dev/ansible-configs).

> [!IMPORTANT]
> **AI-assisted Authorship**
>
> Drafts, examples, and research generated using [Claude](https://claude.com/product/overview), both in the web interface and via [Claude Code](https://code.claude.com/docs/en/overview) after ingesting the existing [ansible-configs](https://github.com/straysheep-dev/ansible-configs) codebase and reviewing the direction in a CLAUDE.md file.
>
> Every file has been reviewed and adjusted as needed by the author.

## Requirements

On RedHat/CentOS, you may need to have EPEL installed before running this role. You can use the `geerlingguy.repo-epel` role if you need a simple way to ensure it's installed.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

`pip_package`, The package used to install pip via the system package manager. For older systems that don't have Python 3 available, you can set this to `python-pip`.

```yml
pip_package: python3-pip
```

`venv_package`, The package used to install `venv` support on Debian/Ubuntu targets. Only installed when `pip_install_packages` is populated.

```yml
venv_package: python3-venv
```

`pip_externally_managed`, Set to `false` **by default** to remove `/usr/lib/python3.x/EXTERNALLY-MANAGED` ([PEP 668](https://peps.python.org/pep-0668/)), allowing pip installs outside a venv. Prefer a venv when possible. This is often required even for `--user` installs.

```yml
pip_externally_managed: false
```

`pip_executable`, The pip executable used for non-venv installs. Auto-derived from `pip_package` (`pip3` for `python3-*` packages, `pip` otherwise). Override per-item with `item.executable` when multiple interpreters are present (e.g. `pip3.13`, `pip2` on Kali).

```yml
pip_executable: pip3
```

`pip_install_packages`, A list of packages to install with pip. Each item maps directly to parameters of [`ansible.builtin.pip`](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/pip_module.html). Examples:

```yml
pip_install_packages:

  # Minimal, latest package version, user install
  - name: requests
    extra_args: --user

  # Pinned system install using become (PEP 668 override, prefer venv instead)
  - name: ansible-lint
    version: "24.2.0"
    state: present              # present | latest | absent | forcereinstall
    umask: "0022"               # octet string; required for readable system-wide installs
    break_system_packages: true

  # venv install, virtualenv_command requires an absolute path (use ansible_python.executable for this)
  - name: scapy
    state: present
    virtualenv: "{{ ansible_env.HOME }}/venv/scapy"
    virtualenv_command: "{{ ansible_python.executable }} -m venv"
    virtualenv_site_packages: true   # inherit system packages into venv; default: false

  # Legacy Python 2 venv (Kali), virtualenv_command and virtualenv_python are mutually exclusive
  - name: some_python2_package
    state: present
    executable: pip2            # per-item override; ignored when virtualenv is set

  # requirements.txt install, mutually exclusive with name; pair with chdir for relative paths
  - requirements: /opt/myapp/requirements.txt
    chdir: /opt/myapp
    virtualenv: /opt/myapp/venv
    virtualenv_command: "{{ ansible_python.executable }} -m venv"
    extra_args: "--extra-index-url https://my-domain/pypi/myapp"
    umask: "0022"               # octet string; required for readable system-wide installs

```

## Dependencies

None.

## Example Playbook

This playbook is how you would setup an Ansible dev system.

```yml
- hosts: all
  vars:
    pip_install_packages:
      - name: docker
        extra_args: "--user"
      - name: molecule
        extra_args: "--user"
      - name: ansible
        extra_args: "--user"
      - name: ansible-lint
        extra_args: "--user"
      - name: "molecule-plugins[docker]"
        extra_args: "--user"
    docker_users:
      - user1

  roles:
    - role: straysheep_dev.install_pip
      become: false
    - role: straysheep_dev.install_docker
      become: true

```

Run the playbook with:

```bash
ansible-playbook -i "localhost," -c local --ask-become-pass -v ./playbook-devops.yml
```

## License

[MIT](./LICENSE)

## Author Information

This role was created in 2017 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).

This role was forked to build and learn from in 2025 by [straysheep-dev](https://github.com/straysheep-dev/ansible-configs).
