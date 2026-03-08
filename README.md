# Building an Ubuntu Server Image with Packer

This guide walks through building a golden Ubuntu server image using Packer with autoinstall, and an Ansible provisioner. The resulting image can be used as a base template for Terraform to provision VMs.

---



### Required Packer plugins

Install the QEMU and Ansible plugins by running:

```bash
packer init .
```

### SSH key

Packer will automatically generate a temporary SSH key for the build. No manual key configuration is required unless you want to use your own key

---

## Project Structure

```
packer/
  main.pkr.hcl        # Packer build config
  http/
    user-data         # Autoinstall config **update this file**
    meta-data         # Can be left empty
  ansible/
    playbook.yml      # Ansible provisioner playbook
```

---

##  Required: Update user-data Before Building

Before running a build you **must** update the `http/user-data` file with your own credentials. Do not use the default values in production.

### 1. Set your username

```yaml
identity:
  hostname: ubuntu-server   # change to your desired hostname
  username: ubuntu          # change to your desired username
```

### 2. Set your password

Generate a hashed password using:

```bash
mkpasswd -m sha-512 yourpassword
```

Then paste the output into `user-data`:

```yaml
identity:
  password: "$6$your_hashed_password_here"
```

### 3. Add your SSH public key

```yaml
ssh:
  install-server: true
  authorized-keys:
    - "ssh-rsa AAAA..."    # paste your public key here
  allow-password-authentication: false
```

---


## Building the Image

### 1. Initialise plugins

```bash
packer init .
```

### 2. Validate the config

```bash
packer validate .
```

### 3. Build the image

```bash
packer build .
```

The build will:
1. Boot the Ubuntu ISO
2. Run the autoinstall unattended install
3. Reboot into the installed OS
4. SSH in and run the Ansible playbook
5. Clean up cloud-init state
6. Save the image to `output/ubuntu-server.qcow2`

For me the build time: 10 - 15 minutes

---


## Notes

- The output image is a QCOW2 file that can be used directly with the Terraform libvirt provider as a base volume.