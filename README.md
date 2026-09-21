# migdady5\n\n*This activity has been created as part of the 42 curriculum by amigdadi.*

# Born2beRoot

## Description

This project is about building a secure minimal Linux server inside a virtual machine. The goal is to understand the basics of system administration: virtualization, encrypted storage, users and groups, password rules, SSH, firewall configuration, sudo restrictions, and automated monitoring.

The server was installed without a graphical interface because it is meant to behave like a real server: small, controlled, and focused only on the services required by the subject.

## Instructions

### Start the virtual machine

Open VirtualBox and start the machine:

```bash
amigdadi42
```

### Connect with SSH

SSH is configured to use port `4242`, and root login through SSH is disabled.

From the host machine:

```bash
ssh amigdadi@localhost -p 4242
```

If SSH shows a host key warning because an old VM used the same address and port, remove the old key:

```bash
ssh-keygen -R "[localhost]:4242"
ssh-keygen -R "[127.0.0.1]:4242"
```

Then connect again:

```bash
ssh amigdadi@localhost -p 4242
```

### Useful verification commands

Check SSH:

```bash
sudo systemctl status ssh
```

Check UFW:

```bash
sudo ufw status verbose
```

Check AppArmor:

```bash
sudo aa-status
```

Check users and groups:

```bash
groups amigdadi
getent group sudo
getent group user42
```

Check password aging:

```bash
sudo chage -l amigdadi
```

Check partitions and LVM:

```bash
lsblk
sudo pvs
sudo vgs
sudo lvs
```

Run the monitoring script manually:

```bash
sudo /usr/local/bin/monitoring.sh
```

Check the scheduled cron task:

```bash
sudo crontab -l
sudo systemctl status cron
```

## Project Description

### Operating System Choice

I chose **Debian Stable** for this project.

Debian is a good choice for a first system administration project because it is stable, lightweight, well documented, and easy to maintain. It also allows a minimal installation without a graphical interface, which fits the purpose of a server machine.

Debian has some disadvantages too. Its stable repositories may contain older package versions compared to faster-moving distributions, because Debian prioritizes reliability over having the newest software. It is also community-driven, so it does not follow the same commercial support model as enterprise distributions.

I considered **Rocky Linux** as the alternative. Rocky Linux is strong for enterprise server environments and is compatible with the Red Hat ecosystem. However, for this project, Debian was more suitable because it is simpler for a beginner and the required tools are easier to configure.

### Main Design Choices

The virtual machine was configured with the following choices:

- Operating system: Debian Stable
- Virtualization tool: VirtualBox
- Hostname: `amigdadi42`
- Main user: `amigdadi`
- Required groups: `sudo` and `user42`
- Disk setup: encrypted LVM
- Security module: AppArmor
- Firewall: UFW
- SSH port: `4242`
- Monitoring: Bash script executed by cron

### Partitioning

The disk was configured using encrypted LVM.

Encryption protects the data stored inside the virtual machine. If the virtual disk is accessed without authorization, the encrypted partitions cannot be read without the passphrase.

LVM was used because it provides more flexible storage management than fixed traditional partitions. Logical volumes can be separated for different parts of the system, such as `/`, `/home`, and `swap`.

### Security Policy

The machine uses a strict security configuration:

- SSH runs on port `4242`
- SSH root login is disabled
- UFW is enabled
- Only port `4242` is allowed through the firewall
- AppArmor is enabled
- Password complexity rules are enforced with `libpam-pwquality`
- Password aging is configured
- sudo is restricted and logged
- A monitoring script runs every 10 minutes

Password rules include:

- Password expires every 30 days
- Minimum number of days before changing a password again: 2
- Warning before expiration: 7 days
- Minimum password length: 10 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one digit
- No more than 3 consecutive identical characters
- Password must not contain the username

Important files used for password configuration:

```bash
/etc/login.defs
/etc/security/pwquality.conf
/etc/pam.d/common-password
```

### Sudo Configuration

sudo was configured with stricter rules to reduce risk and improve traceability.

The sudo configuration includes:

- Maximum of 3 password attempts
- Custom error message
- Input and output logging
- Logs stored under `/var/log/sudo/`
- TTY mode required
- Restricted secure path

The sudo configuration was edited with:

```bash
sudo visudo
```

### User Management

A non-root user was created for normal administration:

```bash
amigdadi
```

This user belongs to:

```bash
sudo
user42
```

The root account exists, but direct SSH login as root is disabled. This avoids exposing the most privileged account over the network.

### Services Installed

The main packages installed on the machine are:

```bash
sudo
openssh-server
ufw
cron
libpam-pwquality
apparmor
apparmor-utils
```

Each package has a specific purpose:

- `sudo`: controlled administrative privileges
- `openssh-server`: remote access through SSH
- `ufw`: firewall management
- `cron`: scheduled execution of the monitoring script
- `libpam-pwquality`: password complexity rules
- `apparmor` and `apparmor-utils`: security profiles and AppArmor status tools

## Technical Comparisons

### Debian vs Rocky Linux

Debian is a community-driven distribution that uses `.deb` packages and the `apt` package manager. It is known for stability, simplicity, and a large software repository. It is suitable for minimal servers because it can be installed with only the required components.

Rocky Linux is an enterprise-focused distribution compatible with Red Hat Enterprise Linux. It uses `.rpm` packages and the `dnf` package manager. It is a strong choice for enterprise environments, especially where RHEL compatibility is needed.

For this project, Debian was chosen because it is easier to configure for a beginner, has clear documentation, and is recommended for new system administration learners.

### AppArmor vs SELinux

AppArmor and SELinux are Linux security modules. They add an extra layer of access control beyond normal Linux permissions.

AppArmor uses path-based profiles. A profile defines what a specific program is allowed to access by referring to file paths. This makes AppArmor easier to read and manage.

SELinux uses label-based access control. Files, processes, ports, and other resources have security labels, and access is controlled through policies. SELinux is very powerful, but it is more complex to configure and troubleshoot.

Debian uses AppArmor, so AppArmor was used in this project.

### UFW vs firewalld

UFW is a simple frontend for managing firewall rules. It is commonly used on Debian-based systems and is easy to configure with simple commands such as allowing or denying a port.

firewalld is commonly used on Red Hat-based systems such as Rocky Linux. It supports zones and dynamic rule management, which is useful for systems that need different firewall behavior depending on the network environment.

For this project, UFW was the better choice because the setup is simple: block unnecessary incoming traffic and allow only SSH on port `4242`.

### VirtualBox vs UTM

VirtualBox is a general-purpose virtualization tool used to create and manage virtual machines. It provides virtual disks, network modes, snapshots, and a clear interface for configuring VM resources.

UTM is mainly used on macOS, especially on Apple Silicon machines. It uses Apple virtualization features and QEMU, which makes it useful when VirtualBox is not suitable.

VirtualBox was used for this project because it is supported by the subject and works well for running a Debian virtual machine.

## Monitoring Script

A Bash script named `monitoring.sh` was created and scheduled with cron.

The script displays system information every 10 minutes on all terminals. It reports:

- System architecture and kernel version
- Number of physical CPUs
- Number of virtual CPUs
- RAM usage and percentage
- Disk usage and percentage
- CPU load
- Last boot time
- LVM status
- Number of established TCP connections
- Number of logged-in users
- IPv4 address and MAC address
- Number of sudo commands executed

Cron entry:

```bash
*/10 * * * * /usr/local/bin/monitoring.sh
```

## Resources

Resources used during this project:

- Debian documentation
- Debian Administrator's Handbook
- VirtualBox documentation
- OpenSSH manual pages
- UFW manual pages
- AppArmor documentation
- Linux manual pages:
  - `man sshd_config`
  - `man sudoers`
  - `man chage`
  - `man crontab`
  - `man passwd`
  - `man pwquality.conf`

## AI Usage

AI was used as a learning assistant during the project.

It helped with:

- Explaining Linux commands and configuration files
- Understanding SSH, UFW, sudo, AppArmor, cron, and password policy settings
- Troubleshooting command errors during setup
- Organizing this README according to the project requirements

All commands and configurations were tested manually inside the virtual machine.

## Submission

Only these files should be submitted in the Git repository:

```bash
README.md
signature.txt
```

The virtual machine itself must not be pushed to the repository.

To generate the virtual disk signature on Linux, go to the folder where the VM disk is stored and run:

```bash
sha1sum amigdadi42.vdi
```

Copy the SHA-1 output into:

```bash
signature.txt
```

Important: starting the virtual machine again can change the disk signature, so the signature should be generated at the final stage before submission.
