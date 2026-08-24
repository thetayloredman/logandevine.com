+++
title = "Debian + Nix: Stability Meets Reproducibility"
date = "2026-08-05"
authors = ["Logan Devine"]

[taxonomies]
tags = ["debian", "nix"]
+++

By now, most people have heard about NixOS. "I'm using Nix" has become the very flex that "I use arch btw" had always strived to be, with none of the meme.

I've been using Debian for years. I run it everywhere, from my desktop to all of my production servers. It brings a familiar, slow-changing system, but the point-release model comes with a major downside for development: you are often working with years old toolchains.

# LLVM 19!?

This issue first arose for me when I was working on [Zirco](/projects/zirco/), which currently pins LLVM 22:

```toml,linenos
[dependencies.inkwell]
version = "0.9.0"
features = [
    "llvm22-1",
    "llvm22-1-prefer-static",
]
```

The Debian Trixie repositories however, only ship LLVM from 17 to 19:

```bash
$ apt list -a 'llvm-*-dev'
llvm-17-dev/stable 1:17.0.6-22+b2 amd64
llvm-18-dev/stable 1:18.1.8-18+b1 amd64
llvm-19-dev/stable,now 1:19.1.7-3+b1 amd64 [installed,automatic]
```

Well... LLVM doesn't provide many great options for system-wide installations here. There is a stage2 apt repository available, but I had various library issues with that in the past. So, I reached for one of the tools I had heard quite a bit about: Nix.

# Wait, Nix without NixOS?

Many people forget that NixOS is a Linux distribution _built around_ the Nix package manager, which is a great tool and can be used standalone on any operating system (even macOS and Windows)!

Nix is fundamentally built around being a reproducible build system, ensuring that it produces software, environments, or even full operating system configurations that are identical across machines.

This solves one particular issue every developer will be familiar with:

# It works on my machine!

The classic excuse, and yet a real problem. The declarative nature of Nix helps solve this issue: one can define their entire development environment (packages, libraries, their versions, and even environment variables) in a single file, and share it with others working on the project. This
ensures that for anyone working on the project, the environment is identical, and therefore eliminates fighting to find missing libraries or other mismatches between contributors.

There are some caveats here, such as environment provided by the host operating system, but these issues are generally far less severe than what you'd encounter with a traditional package manager.

You may ask, if you are using Nix, why not just use NixOS? Well, I like Debian. I like when repos move slowly, except for when I need bleeding edge development tools. Nix gives me the best of both worlds: A system which is stable, and **temporary** environments when I need something newer.

> If you already know Nix, I recommend skipping to [The real benefits](#the-real-benefits).

So, now that you've seen the benefits of running Nix, let's get a simple development environment (with LLVM 22, of course) up and running on Debian!

# Installing Nix

Follow the [proper installation instructions](https://nixos.org/download/) for your OS.

For Debian, as long as you have root on the system, that's as follows:

```bash
$ curl --proto '=https' --tlsv1.2 -L https://nixos.org/nix/install | sh -s -- --daemon
$ nix --version
nix (Nix) ...
$ mkdir -p ~/.config/nix/
$ echo "experimental-features = nix-command flakes" >> ~/.config/nix/nix.conf
```

# A gentle introduction to Flakes

You may have noticed that we enabled two experiments in the `nix.conf` file above. Although flakes are officially still experimental, they have been widely adopted by the Nix community and are often the recommended way of managing a Nix project.

To start, create a new directory for your project and create a `flake.nix` file inside.

```nix,linenos,hl_lines=4-6
{
  description = "My first reproducible devshell!";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
  };

  outputs = { nixpkgs, self }: {};
}
```

A Flake is like Nix's manifest file. It describes a list of input flakes (things like `nixpkgs`, the Nix package repository, or other Nix dependencies) and a list of outputs it produces. This is all, of course, written in the Nix programming language!

In our case, we bring in `NixOS/nixpkgs`, specifically the `unstable` channel which is Nix's rolling-release branch.

## Defining a `devShell`

Inside of `outputs`, we'll define a single development shell, for the `x86_64-linux` system.

```nix,linenos,hide_lines=1 9,hl_lines=3-7,linenostart=7
{
  outputs = { nixpkgs, self }: {
    devShells.x86_64-linux.default =
      let pkgs = import nixpkgs { system = "x86_64-linux"; };
      in pkgs.mkShell {
        buildInputs = with pkgs; [];
      };
  };
}
```

A few notable lines here:

- `devShells.x86_64-linux.default`: defines the specific output we're declaring. In many larger projects, these are automatically generated for all supported system types, but this'll do for now.
- `let pkgs = import nixpkgs { ... }; in`: nixpkgs in particular is not built for flakes yet, so it is "imported" in this way. We also need to tell nixpkgs our `system`, which is our architecture and OS.
- `pkgs.mkShell`: creates a development shell.
- `buildInputs = with pkgs; [];`: declares the packages we want available in our devShell, which for now is nothing.

## Bringing in LLVM 22

Now, we tell Nix to add the packages for LLVM 22 we want: let's say `llvm`, `libllvm`, `clang`, and `lld` (as these are what we use with Zirco).

```nix,linenos,hide_lines=1 10,hl_lines=3-8,linenostart=11
{ thisisjustsosyntaxhighlightinglooksright = let meow = "meow";
      in pkgs.mkShell {
        buildInputs = with pkgs; [
          llvmPackages_22.llvm
          llvmPackages_22.libllvm
          llvmPackages_22.clang
          llvmPackages_22.lld
        ];
      }
;}
```

Now, we can enter our devShell:

```bash
$ clang --version
clang: Command not found
$ nix develop
$ clang --version
clang version 22.0.0 ...
```

Nix has now grabbed the packages we needed and stored them in `/nix`, and temporarily set up the environment for us (e.g. `$PATH`) so that we can use this toolchain. Notice that none of this touches your system - no package is installed into `/usr`. When we exit the shell, our environment is back to normal &mdash; no more fighting with the system package manager, and you haven't tainted your system!

# The real benefits

Cool, we have a working development environment. But there's a lot more to Nix than just that.

Nix also lets you:

- Define your "dotfiles" and other system-wide configuration in a reproducible way (using `home-manager`).
- Build and deploy software that works no matter where it is built
- Declare global service configurations and manage your entire infrastructure with no Ansible
- Much, much more!

# Conclusion

I installed Nix because I needed LLVM 22 on Debian, but I stayed because of how it empowered me to write software and manage my systems in a way where they work almost everywhere. Debian still remains my system of choice, but Nix has become a tool I use daily.

There's much more to it: I haven't talked about Home Manager, packaging for Nix, and more, but we'll get to that in future posts.

In the meantime, if you're new to Nix, you should give [Nix Pills](https://nixos.org/guides/nix-pills/) a read, and if you're a seasoned Nix/NixOS user, maybe try it out on top of your distro of choice. You'd might be surprised at what that balance can do.
