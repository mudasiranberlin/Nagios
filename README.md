# Nagios Core Installation

Installs **Nagios Core 4.5.13** and **Nagios Plugins 2.4.12** from source, with Apache/PHP, on RHEL-based systems (RHEL 9, Rocky, Alma, Amazon Linux 2023).

## Prerequisites

- A fresh RHEL-family server with `sudo`/root access
- Internet access to GitHub (source downloads)
- Ports 80/443 open if accessing the web UI remotely

## What This Installs

- Apache (`httpd`) and PHP — serves the Nagios web interface
- GCC, glibc, GD libraries — build dependencies for compiling Nagios from source
- A dedicated `nagios` system user and `nagcmd` group (shared between `nagios` and `apache` so the web server can issue external commands)
- Nagios Core, built and installed to `/usr/local/nagios`
- Nagios Plugins, the check scripts (`check_ping`, `check_http`, etc.) Nagios uses to monitor hosts and services

## Installation Steps

### 1. Become root

```bash
sudo -i
```

### 2. Install web server, PHP, and build dependencies

```bash
dnf install -y httpd php gcc glibc glibc-common gd gd-devel make wget unzip \
    perl postfix openssl-devel
```

### 3. Create the nagios user and command group

```bash
useradd nagios
passwd nagios
groupadd nagcmd
usermod -a -G nagcmd nagios
usermod -a -G nagcmd apache
```

You'll be prompted to set a password for the `nagios` system user — this is separate from the Nagios web login set up in step 5.

### 4. Download and build Nagios Core

```bash
mkdir -p ~/downloads && cd ~/downloads
wget -O nagioscore.tar.gz https://github.com/NagiosEnterprises/nagioscore/archive/refs/tags/nagios-4.5.13.tar.gz
wget -O nagios-plugins.tar.gz https://github.com/nagios-plugins/nagios-plugins/archive/refs/tags/release-2.4.12.tar.gz

tar zxvf nagioscore.tar.gz
cd nagioscore-nagios-4.5.13
./configure --with-command-group=nagcmd
make all
make install
make install-init
make install-config
make install-commandmode
make install-webconf
```

### 5. Set the web UI login and start Apache

```bash
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
systemctl enable httpd
systemctl restart httpd
```

This is the username/password you'll use to log into the Nagios web dashboard.

### 6. Build and install the plugins

```bash
cd ~/downloads
tar zxvf nagios-plugins.tar.gz
cd nagios-plugins-release-2.4.12
./tools/setup   # generates ./configure for this repo layout
./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-openssl
make
make install
```

### 7. Enable and start Nagios

```bash
systemctl enable nagios
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
systemctl start nagios
systemctl restart httpd
```

The `-v` command verifies the config file before starting the service — fix any reported errors before moving on.

## Troubleshooting

### `./tools/setup` fails with `aclocal`, `autoheader`, `automake`, `autoconf`: command not found

The plugins repo generates its `configure` script from autotools, which isn't installed by default. Install it, then rerun step 6:

```bash
dnf install -y autoconf automake libtool m4

cd ~/downloads/nagios-plugins-release-2.4.12
./tools/setup
./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-openssl
make
make install
```

### `autoconf`/`automake`/`libtool`/`m4` not found by `dnf`

On RHEL 9, Rocky, and Alma, these packages live in the CodeReady Builder (CRB) repo, which isn't enabled by default. Enable it and retry:

```bash
dnf install -y epel-release
dnf config-manager --set-enabled crb
dnf install -y autoconf automake libtool m4
```

(Not needed on Amazon Linux 2023 — these tools are already in the base repos.)

## Verifying the Install

- Check the config is valid: `/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg`
- Check the service is running: `systemctl status nagios`
- Access the web UI at `http://<server-ip>/nagios` and log in with the `nagiosadmin` credentials from step 5







#  For me  to rember
# Nagios

# Become root
sudo -i

# Install web server, PHP, and build dependencies
dnf install -y httpd php gcc glibc glibc-common gd gd-devel make wget unzip \
    perl postfix openssl-devel

# Create nagios user and command group
useradd nagios
passwd nagios
groupadd nagcmd
usermod -a -G nagcmd nagios
usermod -a -G nagcmd apache

# Download sources (current stable versions from GitHub)
mkdir -p ~/downloads && cd ~/downloads
wget -O nagioscore.tar.gz https://github.com/NagiosEnterprises/nagioscore/archive/refs/tags/nagios-4.5.13.tar.gz
wget -O nagios-plugins.tar.gz https://github.com/nagios-plugins/nagios-plugins/archive/refs/tags/release-2.4.12.tar.gz

# Build and install Nagios Core
tar zxvf nagioscore.tar.gz
cd nagioscore-nagios-4.5.13
./configure --with-command-group=nagcmd
make all
make install
make install-init
make install-config
make install-commandmode
make install-webconf

# Set up web login and restart Apache
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
systemctl enable httpd
systemctl restart httpd

# Build and install plugins
cd ~/downloads
tar zxvf nagios-plugins.tar.gz
cd nagios-plugins-release-2.4.12
./tools/setup   # generates ./configure for this repo layout
./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-openssl
make
make install

#  If Got Error Use this 
1. dnf install -y autoconf automake libtool m4

# 2.optional 
cd ~/downloads/nagios-plugins-release-2.4.12
./tools/setup
./configure --with-nagios-user=nagios --with-nagios-group=nagios --with-openssl
make
make install

# 3. optional 
dnf install -y epel-release
dnf config-manager --set-enabled crb
dnf install -y autoconf automake libtool m4

# Enable and start Nagios
systemctl enable nagios
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
systemctl start nagios
systemctl restart httpd
