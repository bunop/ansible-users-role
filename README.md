ansible-users-role
===================

Manages system users, groups, root/user SSH keys, dotfiles and (optionally) per-user
Conda environment activation. Root and every listed user get their own `authorized_keys`
and can opt in to [bunop/dotfiles](https://github.com/bunop/dotfiles) via symlinks.

Requirements
------------

None beyond a Debian/Ubuntu-based target. If Conda-related tasks should run, Conda must
already be installed at `miniconda_installation_dir` (e.g. via
[galaxyproject.miniconda](https://github.com/galaxyproject/ansible-role-galaxy-miniconda)) —
the role detects this automatically and skips those tasks otherwise.

Role Variables
---------------

Defined in `defaults/main.yml`:

| Variable | Default | Description |
| --- | --- | --- |
| `root_keys` | `[]` | List of public SSH keys to add for the `root` user |
| `root_keys_path` | `/root/.ssh/authorized_keys` | Destination file for `root_keys`; override for setups with a different root-key mechanism (e.g. a cluster-wide path) |
| `users` | `[]` | List of user accounts to create, each `{name, ssh-keys: [...], dotfiles: true/false}` |
| `custom_groups` | `[]` | List of extra group names to create, each with a shared `g+ws` `/home/<group>` directory |
| `user_groups` | `{}` | Maps a group name to the list of users appended to it (group must already exist) |
| `dotfiles_version` | `ubuntu` | Branch of `bunop/dotfiles` checked out for users with `dotfiles: true` |
| `miniconda_installation_dir` | `/usr/local/anaconda` | Path where Conda is installed; must match the value used to install Conda |
| `miniconda_conda_bin` | `{{ miniconda_installation_dir }}/bin` | Used to detect whether Conda is installed before running Conda-related tasks |

Example user entry:

```yaml
users:
  - name: alice
    ssh-keys:
      - ssh-ed25519 AAAA... alice@laptop
    dotfiles: true
```

Additional information
-----------------------

- Root gets its SSH keys and, unconditionally, the `root` dotfiles from
  `tasks/root.yml`.
- Every entry in `users` gets its account, `.ssh` folder and SSH keys from
  `tasks/users.yml`; dotfiles are only installed when `dotfiles: true` is set on
  that user.
- `custom_groups` and `user_groups` let you create extra groups (with a shared,
  setgid `/home/<group>` directory) and attach existing users to them.
- Conda tasks (`tasks/miniconda.yml`) only run when a `conda`/`mamba` binary is found
  under `miniconda_conda_bin`; they add a `/etc/profile.d` snippet and run
  `conda init bash` once per user (root included).

Dependencies
------------

None.

Example Playbook
-----------------

```yaml
- hosts: servers
  roles:
    - role: users
      tags: users_role
      vars:
        root_keys:
          - ssh-ed25519 AAAA... admin@workstation
        users:
          - name: alice
            ssh-keys:
              - ssh-ed25519 AAAA... alice@laptop
            dotfiles: true
```

License
-------

MIT

Author Information
-------------------

Paolo Cozzi ([@bunop](https://github.com/bunop))
