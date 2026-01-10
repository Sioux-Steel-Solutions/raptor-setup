# Raptor Pi Setup

Ansible playbook to configure fresh Rev Pi OS for raptor-controls kiosk mode.

## What This Configures

| Component | Configuration |
|-----------|---------------|
| Password | Standardized to `b8rqse` (from factory default) |
| Auto-login | Boots directly to desktop as `pi` user |
| Chromium | Opens `raptor-controls.vercel.app` on boot |
| raptor-core | Cloned from GitHub, running via docker-compose |
| Docker | Installed, enabled, containers set to restart always |
| GPU | CMA-256 for better graphics performance |
| CPU | 2.4GHz, performance governor |
| Swap | Zram compressed swap enabled |
| Hostname | Unique per Pi (`raptor-01`, `raptor-02`, etc.) |

## Prerequisites

### Install Ansible on your Mac:

```bash
brew install ansible
```

### Install required Ansible collections:

```bash
ansible-galaxy collection install community.docker
```

## Usage

### 1. Add Your Pi to Inventory

Edit `inventory.yml` - each Pi needs its IP and a unique ID:

```yaml
all:
  vars:
    ansible_user: pi
  hosts:
    raptor-01:
      ansible_host: 192.168.1.101  # Pi's IP address
      raptor_id: "01"
    raptor-02:
      ansible_host: 192.168.1.102
      raptor_id: "02"
    raptor-03:
      ansible_host: 192.168.1.103
      raptor_id: "03"
```

### 2. Run the Playbook

Since each Pi ships with a **different factory password**, use `--ask-pass`:

```bash
cd raptor-pi-setup

# Configure a single Pi (enter factory password when prompted)
ansible-playbook -i inventory.yml playbook.yml --ask-pass --ask-become-pass --limit raptor-01

# After first run, password is standardized to 'b8rqse'
# So for subsequent runs or other Pis with same password:
ansible-playbook -i inventory.yml playbook.yml -e "ansible_password=b8rqse ansible_become_password=b8rqse"
```

### 3. What Happens

1. Connects to Pi with factory password
2. Sets standardized password (`b8rqse`)
3. Installs packages (docker, chromium, git, etc.)
4. Clones `raptor-core` from GitHub
5. Runs `docker-compose up -d --build`
6. Configures auto-login + kiosk browser
7. Sets GPU/CPU performance settings
8. Sets unique hostname (`raptor-01`, etc.)
9. Reboots Pi

After reboot, Pi will:
- Auto-login as `pi`
- Open Chromium to `raptor-controls.vercel.app`
- Have `raptor-core` running in Docker

## Making Changes to the "Master Image"

1. Make changes on your dev Pi
2. Update `playbook.yml` to reflect those changes
3. Commit to GitHub
4. Run playbook on new Pis

```bash
git add .
git commit -m "Added new feature to Pi setup"
git push
```

## Updating Existing Pis

Ansible is idempotent - just run the playbook again:

```bash
# Update all Pis
ansible-playbook -i inventory.yml playbook.yml -e "ansible_password=b8rqse ansible_become_password=b8rqse"

# Update specific Pi
ansible-playbook -i inventory.yml playbook.yml --limit raptor-02 -e "ansible_password=b8rqse ansible_become_password=b8rqse"
```

## Updating raptor-core Only

If you just pushed changes to raptor-core and want to update Pis:

```bash
ansible all -i inventory.yml -m shell -a "cd ~/raptor-core && git pull && docker-compose up -d --build" -b -e "ansible_password=b8rqse ansible_become_password=b8rqse"
```

## Troubleshooting

### Can't connect to Pi
- Make sure Pi is on the network
- Check IP address is correct in inventory
- Try: `ping <pi-ip-address>`

### Factory password doesn't work
- Rev Pis ship with unique passwords printed on label
- Use `--ask-pass` and enter the password from the label

### Docker-compose fails
- SSH in manually: `ssh pi@<ip>`
- Check logs: `cd ~/raptor-core && docker-compose logs`
