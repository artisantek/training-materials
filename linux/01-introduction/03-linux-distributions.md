# Linux Distributions (Distros)

So if the Linux kernel is just the engine, and anyone can build a "car" around it — **who builds these cars, and which one should you use?**

That's where **distributions** come in.

---

## What is a Distribution?

A **Linux distribution** (distro) is a complete operating system built around the Linux kernel. Each distro bundles these layers differently:

<div align="center">
  <img src="../images/linux-building-blocks.png" alt="Linux Building Blocks" width="700" />
</div>

Think of it like **Android phones** — Samsung, OnePlus, and Pixel all run Android (the kernel), but each has its own UI, default apps, and update cycle. Linux distros work the same way.

---

## The Major Distro Families

Most Linux distros descend from a few **parent distributions**. Understanding these families helps you navigate the entire ecosystem.

### 🔵 Debian Family

The **stability-first** family. Known for rock-solid reliability and the `apt` package manager.

```
Debian
├── Ubuntu
│   ├── Linux Mint
│   ├── Pop!_OS
│   └── Elementary OS
├── Kali Linux (security/pentesting)
└── Raspberry Pi OS
```

| Distro | Best For | Package Manager |
|--------|----------|-----------------|
| **Debian** | Servers that need maximum stability | `apt` / `dpkg` |
| **Ubuntu** | General-purpose servers and desktops | `apt` / `dpkg` |
| **Ubuntu Server** | Cloud and server deployments (most popular on AWS) | `apt` / `dpkg` |

**Why it matters to you:** When you spin up an **EC2 instance** on AWS, you'll most likely choose **Ubuntu** or **Amazon Linux**. Ubuntu commands (`apt install`, `dpkg`) will be second nature.

---

### 🔴 Red Hat Family

The **enterprise** family. Backed by Red Hat (now IBM), focused on enterprise support and certifications.

```
Red Hat Enterprise Linux (RHEL)
├── CentOS (was free RHEL clone, now CentOS Stream)
├── Rocky Linux (community successor to CentOS)
├── AlmaLinux (another CentOS successor)
├── Fedora (cutting-edge, upstream of RHEL)
└── Amazon Linux 2 / Amazon Linux 2023
```

| Distro | Best For | Package Manager |
|--------|----------|-----------------|
| **RHEL** | Enterprise production environments (paid support) | `yum` / `dnf` / `rpm` |
| **Rocky Linux** | Free RHEL alternative for servers | `yum` / `dnf` / `rpm` |
| **Amazon Linux** | AWS-optimized workloads | `yum` / `dnf` / `rpm` |
| **Fedora** | Developers who want latest features | `dnf` / `rpm` |

**Why it matters to you:** Many production environments in large companies run **RHEL** or its free derivatives. **Amazon Linux** is AWS's own distro — optimized for EC2. You'll encounter `yum` / `dnf` commands here.

---

### 🟢 Other Notable Families

| Family | Key Distro | Known For | Package Manager |
|--------|-----------|-----------|-----------------|
| **Arch** | Arch Linux, Manjaro | Bleeding-edge, DIY | `pacman` |
| **SUSE** | openSUSE, SLES | Enterprise (European market) | `zypper` / `rpm` |
| **Alpine** | Alpine Linux | Ultra-minimal (~5MB base) | `apk` |

> 💡 **Alpine Linux** is extremely important in DevOps — it's the base image used in most **Docker containers** because of its tiny size. You'll see `FROM alpine` in Dockerfiles everywhere.

---

## Which Distros Will You Actually Use?

In the real world of DevOps and cloud engineering, you'll primarily work with these:

| Distro | Where You'll See It |
|--------|-------------------|
| **Ubuntu Server** | AWS EC2 (most common choice), DigitalOcean, personal projects |
| **Amazon Linux 2023** | AWS EC2 (AWS's recommended distro) |
| **RHEL / Rocky Linux** | Enterprise production environments |
| **Alpine Linux** | Docker base images |
| **Debian** | Stable servers, some Docker images |

### The Two Package Manager Commands You Must Know

Since most distros fall into either Debian or Red Hat family:

```bash
# Debian / Ubuntu family
sudo apt update            # Refresh package list
sudo apt install nginx     # Install a package
sudo apt remove nginx      # Remove a package

# Red Hat / Amazon Linux family
sudo yum install nginx     # Install a package (older)
sudo dnf install nginx     # Install a package (newer, replacing yum)
sudo yum remove nginx      # Remove a package
```

---

## How to Check What Distro You're Running

When you SSH into a server and need to know what you're working with:

```bash
# Works on most distros
cat /etc/os-release

# Example output on Ubuntu:
NAME="Ubuntu"
VERSION="22.04.3 LTS (Jammy Jellyfish)"
ID=ubuntu
ID_LIKE=debian

# Example output on Amazon Linux:
NAME="Amazon Linux"
VERSION="2023"
ID=amzn
ID_LIKE=fedora
```

```bash
# Quick one-liner to get just the name
cat /etc/os-release | grep "PRETTY_NAME"
# Output: PRETTY_NAME="Ubuntu 22.04.3 LTS"
```

---

## Release Models — LTS vs Rolling

Distros handle updates differently:

| Model | How It Works | Example |
|-------|-------------|---------|
| **LTS (Long Term Support)** | Major release every few years, supported for 5+ years with security patches | Ubuntu 22.04 LTS, RHEL 9 |
| **Rolling Release** | Continuous updates, always the latest version | Arch Linux, Fedora (semi-rolling) |
| **Fixed Release** | Regular release cycle with a defined support period | Debian (every ~2 years) |

**For servers, always use LTS releases.** You want stability and long-term security patches, not the latest experimental features.

```
Ubuntu Release Example:

Ubuntu 22.04 LTS "Jammy Jellyfish"
  │       │   │
  │       │   └── Long Term Support (5 years of updates)
  │       └────── Released in April (04)
  └────────────── Released in 2022
```

---

## Choosing a Distro — Decision Flowchart

```
What are you doing?
│
├── Deploying on AWS?
│   ├── Want AWS-optimized? → Amazon Linux 2023
│   └── Want community support? → Ubuntu Server LTS
│
├── Enterprise / Corporate?
│   ├── Need paid support? → RHEL
│   └── Need free RHEL-compatible? → Rocky Linux / AlmaLinux
│
├── Building Docker images?
│   └── Want minimal size? → Alpine Linux
│
├── Learning / Personal projects?
│   └── → Ubuntu Server LTS (most tutorials are written for it)
│
└── Desktop use?
    └── → Ubuntu Desktop / Fedora / Linux Mint
```

---

## Key Takeaways

- A **distribution** = Linux kernel + package manager + system tools + configuration. Same engine, different cars.
- The two families you'll use most: **Debian/Ubuntu** (`apt`) and **Red Hat/Amazon Linux** (`yum`/`dnf`).
- **Ubuntu Server LTS** is the most common choice for learning and general deployments.
- **Amazon Linux** is optimized for AWS workloads.
- **Alpine Linux** is the go-to for Docker container base images due to its tiny footprint.
- Always check what distro you're on with `cat /etc/os-release`.
- For production servers, **always use LTS releases**.

---

**← Previous:** [Why Linux for Deployment](./02-why-linux-for-deployment.md) · **Back to** [Linux Home](../README.md)
