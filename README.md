# Ansible Quick Start — Inventory, SSH known_hosts, and Vault

This README shows a minimal workflow to:
- configure an inventory with groups and host IPs (including per-host variables like `ansible_user` and `ansible_password`),
- add the host's SSH key to `known_hosts`,
- create an Ansible Vault and store `ansible_become_pass` inside it.

## 1) Inventory example
Edit an inventory file (`hosts`) with a group and the host IP below it. You can specify per-host variables inline:

```ini
[linux]
#Example 192.168.195.131
```

Notes:
- Prefer not to store plaintext passwords in inventory. Use `ansible-vault` or `host_vars`/`group_vars` with vault instead.

## 2) Add SSH host key to known_hosts
Run the following to add the host's SSH key to the system `known_hosts` (requires `sudo`):

```bash
sudo ssh-keyscan -H "IP" | sudo tee -a /etc/ssh/ssh_known_hosts
```

This prevents interactive host-key prompts when Ansible connects via SSH.

## 3) Create an Ansible Vault
Create a vault file to store sensitive variables (e.g., become/sudo password):

```bash
ansible-vault create vault.yml
```

When prompted, enter a vault password. The editor will open; add the secret variables below.

## 4) Vault contents for sudo access
Inside the vault (e.g., `vault.yml`) add the following YAML so Ansible can escalate using the vaulted password:

```yaml
ansible_become_pass: passwowrd
ansible_user: username
ansible_password: password
```

Save and exit the editor. Your `vault.yml` will be encrypted on disk.

## 5) Using the vault when running playbooks
To run a playbook that needs the vaulted variables:

```bash
ansible-playbook site.yml --ask-vault-pass
# or with a password file (careful with file permissions):
ansible-playbook site.yml --vault-password-file ~/.vault_pass.txt
```

## 6) Quick tips
- Prefer SSH key authentication (`ansible_ssh_private_key_file`) over passwords where possible.
- Keep the vault password and any password files secure (`chmod 600`).
- Use `ansible-lint` or `ansible-playbook --check` to validate changes before applying.

---
