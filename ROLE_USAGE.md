### Using our roles in your playbooks

We recommend using [Ansible Galaxy](https://galaxy.ansible.com/ui/) to install our roles.

An example of a playbook using Punkt.de's open-source roles can be seen here: [ansible-proserver-template](https://github.com/punktDe/ansible-proserver-template)

If you prefer to create your playbook from scratch:

1. Add the role to the playbook's `requirements.yml` as follows:
```yaml
roles:
  - name: nginx # arbitrary name
    src: https://github.com/punktDe/ansible-proserver-nginx
    version: "1.1.0" # git tag or ref
```
We use Git sources instead of Galaxy due to a [timeout issue](https://github.com/ansible/galaxy/issues/2302) affecting Ansible Galaxy downloads.

2. Configure the Galaxy roles to be installed to the playbook's `roles` subfolder, instead of $HOME/.ansible, by adding the following to `ansible.cfg`:
```
[defaults]
roles_path = ./roles
```

3. Add the `role/<rolename>` path to `.gitignore`, so that the role contexts don't get commited with the playbook

4. Install the role using Ansible Galaxy:
```bash
ansible-galaxy install -r requirements.yml
```
In case you've changed the tag/ref of a role, you'll need to use the `--force` parameter in order for it to be re-downloaded.

5. Use the role in your playbook

There are two options:

**Option 1:** Adding the role directly to the playbook's YAML file:
```yaml
- name: Nginx
  hosts: nginx
  become: true
  tags:
    - nginx
  roles:
    - nginx
```

**Option 2:** Adding the role as a dependency to another role (`roles/<role name>/meta/defaults.yml`):
```
---
dependencies:
  - role: nginx
```

In the latter case, the role that lists other roles as dependencies will also **inherit their variables and handlers**.

For instance, if your Neos/Typo3 role templates out an Nginx configuration, you'll be able to utilize the `{{ nginx.prefix.config }}` variable by adding the Nginx role as a dependency.

Additionally, you'll be able to restart Nginx on configuration changes.

For example:
```
- name: Template neos nginx configurations
  notify: Restart nginx # inherited from the nginx role
  ansible.builtin.template:
    src: nginx/http.d/app.conf.j2
    dest: "{{ nginx.prefix.config }}/http.d/app.conf" inherited from the nginx role
    mode: "0644"
```
