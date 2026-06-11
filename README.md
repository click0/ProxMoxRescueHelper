# ProxRescue

English | [Русский](README_RU.md) | [Українська](README_UK.md)

Proxmox Products Installer in Rescue Mode for Hetzner

Description

This script is designed to install Proxmox products (Proxmox Virtual Environment, Proxmox Backup Server, Proxmox Mail Gateway, Proxmox Datacenter Manager) in rescue mode on Hetzner servers. It allows you to select the product to install, configure VNC connection settings, choose UEFI or Legacy BIOS boot, and apply common post-install optimizations. Additionally, the script can launch the installed Proxmox system, allowing you to connect via VNC or noVNC.

Quick Start

ProxRescue is a single self-contained script — no need to clone the repository, only one file is required.

Run directly in the Hetzner rescue system:

    bash -c "$(curl -fsSL https://raw.githubusercontent.com/Proxmoxinfo/ProxMoxRescueHelper/refs/heads/main/ProxRescue.sh)"

To pass command-line flags with the one-liner, add `_` as a placeholder for `$0`:

    bash -c "$(curl -fsSL https://raw.githubusercontent.com/Proxmoxinfo/ProxMoxRescueHelper/refs/heads/main/ProxRescue.sh)" _ -pve -auto -dns 8.8.8.8

Or download the script once and run it with flags as needed:

    curl -fsSL -o ProxRescue.sh https://raw.githubusercontent.com/Proxmoxinfo/ProxMoxRescueHelper/refs/heads/main/ProxRescue.sh && chmod +x ProxRescue.sh
    ./ProxRescue.sh -pve -auto -dns 8.8.8.8

Requirements

Before running the script, ensure that your system has the following packages installed:

    curl
    sshpass
    dialog
    git

Installation of Required Packages

If the required packages are not installed, the script will attempt to install them automatically.

Usage

Run the script with the appropriate parameters to install the selected Proxmox product or to configure the system:

Command Line Parameters

Installation:

    -pve: Install Proxmox Virtual Environment.
    -pbs: Install Proxmox Backup Server.
    -pmg: Install Proxmox Mail Gateway.
    -pdm: Install Proxmox Datacenter Manager.

Post-install (applied automatically after installation):

    -fix-sources: Fix Debian base sources (deb.debian.org).
    -no-sub: Switch Enterprise repos to no-subscription + remove subscription nag.
    -upgrade: Run apt update + dist-upgrade (requires -no-sub).
    -disable-ha: Disable HA services (single-node PVE only).
    -auto: Apply all post-install optimizations above without prompting.

Connection:

    -p, --password PASSWORD: Specify a password for the VNC connection.
    -vport PORT: Set the port for noVNC (default 8080).
    -dns DNS_SERVER[,DNS_SERVER...]: Set one or more DNS servers, comma-separated (default: auto-detected from the rescue system's /etc/resolv.conf, fallback 1.1.1.1).
    -uefi: Force UEFI boot mode.
    -legacy: Force Legacy BIOS boot mode.

Other:

    -h, --help: Show the help message and exit.

If neither -uefi nor -legacy is given, the boot mode is auto-detected based on the rescue system's firmware.

If -dns is not given, the DNS server is auto-detected from the rescue system's /etc/resolv.conf (falling back to 1.1.1.1 if it cannot be detected).

Examples

    Install Proxmox Virtual Environment with a specified VNC password:

       ./ProxRescue.sh -pve -p yourVNCpassword

    Install Proxmox VE with all post-install optimizations applied automatically and a custom DNS server:

       ./ProxRescue.sh -pve -auto -dns 8.8.8.8

    Install Proxmox Backup Server and switch to no-subscription repos with a full upgrade:

       ./ProxRescue.sh -pbs -no-sub -upgrade

    Install Proxmox VE with several post-install fixes applied individually:

       ./ProxRescue.sh -pve -fix-sources -no-sub -upgrade -disable-ha

Main Menu

When running the script without parameters, the main menu will be displayed:

    1. Select disks for QEMU
    2. Install Proxmox (PVE, PBS, PMG, PDM)
    3. Run installed System in QEMU
    4. Toggle boot mode (UEFI / Legacy BIOS)
    5. Change VNC Password
    6. Change DNS server(s)
    7. Reboot
    8. Exit

The current boot mode (auto-detected or manually set) is shown at the top of the menu.

Features

    Self-Update Check:
        The script displays its current version on startup and in --help.
        On startup it checks the GitHub repository for a newer version and, if available,
        offers to download it and restart automatically with the same arguments.

    Automatic Installation of Proxmox Products:
        Choose from Proxmox Virtual Environment, Proxmox Backup Server, Proxmox Mail Gateway, or Proxmox Datacenter Manager.
        Automatically download the latest version of the selected product, or pick an older version from the list.
        ISO downloads use HTTPS (with HTTP fallback) and SHA256 checksum verification.

    Post-install Optimizations:
        Fix Debian base sources to use deb.debian.org.
        Switch Enterprise repositories to the no-subscription channel and remove the subscription nag (web and mobile UI).
        Run apt update + dist-upgrade.
        Disable High Availability services on single-node PVE installs.
        Each optimization can be applied interactively, via individual flags, or all at once with -auto.

    VNC Configuration:
        Set a custom VNC password for secure access, or change it later from the menu.
        Specify the noVNC port to avoid conflicts with existing services.

    Boot Mode (UEFI / Legacy BIOS):
        Auto-detected from the rescue system's firmware.
        Can be forced via -uefi / -legacy flags, or toggled interactively from the menu.

    DNS Configuration:
        DNS server(s) are auto-detected from the rescue system's /etc/resolv.conf (all valid nameserver entries, with the systemd-resolved stub resolver 127.0.0.53 resolved via /run/systemd/resolve/resolv.conf), falling back to 1.1.1.1 if none can be detected.
        Can be overridden via the -dns flag with one or more comma-separated IPs (e.g. -dns 8.8.8.8,1.1.1.1), with validation and fallback to 1.0.0.1 if no valid IP is given.
        All detected/specified DNS servers are written to /etc/resolv.conf on the installed system.

    Network Configuration:
        Automatically detect and configure network settings (bridge vmbr0).
        Provide the root password set during installation to apply the network configuration to the installed Proxmox system.

    Reboot Management:
        Option to reboot the server after installation or configuration changes.
        Ensure clean shutdown of QEMU and noVNC before rebooting.

    NoVNC Integration:
        Automatically set up and run noVNC for web-based VNC access.
        Cleanly stop noVNC sessions when no longer needed.

    Disk Selection:
        Manually select which disks to pass to QEMU via an interactive dialog.

    Interactive Menu:
        User-friendly menu interface for selecting installation and configuration options.
        Ability to run the script non-interactively with command-line parameters.

Notes

    The script automatically terminates all noVNC sessions and sends a quit command to the QEMU monitor on exit/start.
    KVM (/dev/kvm) is required and checked at startup.
    During the Proxmox installer (step inside noVNC/VNC), do NOT configure or change the network/IP settings
    (leave them as default/DHCP or whatever the installer pre-fills). Network configuration is handled
    automatically by the script afterwards (configure_network) — manually setting the IP at this stage will
    break the subsequent automatic network setup and post-install steps.

Acknowledgements

Part of the post-install optimization logic (repository switching and subscription nag removal) is adapted from [community-scripts.org](https://community-scripts.org).

MIT License

Copyright (c) 2026 Proxmox UA

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

Communities and Support

    Telegram: Proxmox_UA
    GitHub: https://github.com/Proxmoxinfo/ProxMoxRescueHelper
    Website: proxmox.info

This script is designed for installing Proxmox products in rescue mode on Hetzner servers.
