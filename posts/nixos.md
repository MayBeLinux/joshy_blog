# NixOS: A Different Way to Think About Your System

*Published April 15, 2026 · 8 min read*

Most Linux distributions manage packages the same way: you install something, it drops files across `/usr/bin`, `/lib`, `/etc`, and you hope nothing conflicts. NixOS throws all of that out.

## The Core Idea

NixOS is built on a single radical premise: **your entire system is a function**. You give it a configuration file, and it deterministically produces the same system every time, on any machine. No state accumulated over years of `apt install`. No "works on my machine." Just a pure, reproducible output.

This comes from the [Nix package manager](https://nixos.org/), which stores every package in its own isolated path under `/nix/store`, keyed by a hash of all its inputs:

```
/nix/store/9xndr8rcwrw3r8nxv0z9r5h5b2lhg9rw-firefox-124.0/
```

Two versions of the same package? Two different paths. No conflict possible.

## Declarative Configuration

On a traditional distro, your system state lives in your head (or a runbook). On NixOS, it lives in `/etc/nixos/configuration.nix`:

```nix
{ config, pkgs, ... }:

{
  environment.systemPackages = with pkgs; [
    neovim
    git
    ripgrep
    fd
  ];

  services.openssh.enable = true;

  networking.hostName = "josh-machine";

  users.users.josh = {
    isNormalUser = true;
    extraGroups = [ "wheel" "docker" ];
  };
}
```

Run `nixos-rebuild switch` and your system becomes exactly that. Add a package to the list, rebuild — it's installed. Remove it, rebuild — it's gone. Completely.

## Rollbacks

Here's where it gets genuinely impressive. Every rebuild creates a new **generation**. Your bootloader lists all of them. Broke something? Boot the previous generation. It's not a snapshot — it's the actual, complete previous system configuration, instantly bootable.

```bash
$ nix-env --list-generations
  42   2026-03-01  (current)
  41   2026-02-28
  40   2026-02-20
```

## Flakes

**Nix Flakes** are the modern way to pin dependencies. A `flake.nix` locks every input (nixpkgs version, other flakes) to an exact commit hash. Your dev environment from two years ago? Still reproducible, exactly.

```nix
{
  inputs.nixpkgs.url = "github:NixOS/nixpkgs/nixos-24.11";

  outputs = { self, nixpkgs }: {
    nixosConfigurations.josh-machine = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";
      modules = [ ./configuration.nix ];
    };
  };
}
```

## The Trade-offs

NixOS isn't for everyone. The learning curve is steep — the Nix language is unusual, error messages can be cryptic, and the mental model takes time to internalize. Some proprietary software fights against the isolated store model.

But once it clicks, the appeal is undeniable. Your entire system, version-controlled, portable, reproducible. Infrastructure-as-code, but for your laptop.

## Should You Try It?

If you're comfortable with Linux and curious about functional programming concepts applied to systems, yes — absolutely. Install it in a VM first. Spend a weekend with it. The [NixOS manual](https://nixos.org/manual/nixos/stable/) and [nixos.wiki](https://nixos.wiki) are solid starting points.

The way NixOS makes you think about state, reproducibility, and side effects has genuinely changed how I approach software in general. That alone makes it worth learning.
