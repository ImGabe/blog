+++
title = "Problems I don't miss"
slug = "problemas-que-nao-sinto-saudades"

date = 2024-11-16

[taxonomies]
tags = [ "linux", "nixos", "nix" ]
+++

# Remembering the Chaos of the `apt`

Yesterday, I helped a friend install Elementary OS on her laptop. The installation process went smoothly until we reached the part where we needed to install programs like Docker, some JetBrains IDEs, PostgreSQL, and other tools. That’s when I remembered the chaos of managing packages with `apt`.

<!-- more -->

I’ve been using NixOS for a few years and have gotten used to configuring my system declaratively. If I wanted to install steam, idea-ultimate, and docker, I would do something like this:

```nix
home.packages = with pkgs; [
    jetbrains.idea-ultimate
];

programs.steam.enable = true;
virtualisation.docker.enable = true;
```

However, in Elementary OS (Ubuntu-based), there are several ways to install packages:

- Some are available as .deb files, but they often require manual dependency installation.
- Others come as .AppImage, but need additional setup for system integration.
- Some software requires adding PPAs (Personal Package Archive).

It took me a while to help her install everything. That was on me, but I don’t miss dealing with this kind of hassle.

## The Problem with PPAs

One of the most frustrating moments was trying to install pgAdmin 4, following the official tutorial:

```
# Install the repository's public key:
curl -fsS https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo gpg --dearmor -o /usr/share/keyrings/packages-pgadmin-org.gpg

# Create the repository configuration file:
sudo sh -c 'echo "deb [signed-by=/usr/share/keyrings/packages-pgadmin-org.gpg] https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list && apt update'

# Install pgAdmin 4:
sudo apt install pgadmin4
```

The problem came from this part:

```
curl -fsS https://.../pgadmin4/apt/$(lsb_release -cs)
```

The `lsb_release -cs` command returns the system's codename. In `Ubuntu 22.04` LTS-based distributions like Elementary OS, it should be `"jammy"`. However, `Elementary OS 7.1` has the codename `"horus"`, which is not recognized by the pgAdmin repository, resulting in an invalid link in the PPA configuration.

The solution was to manually edit the repository file and replace `"horus"` with `"jammy"`. A simple fix, but one that went unnoticed and ended up requiring manual intervention. What should have been an automated script turned into an unnecessary hassle. This experience reminded me how much I appreciate using NixOS.