<!--
author: James Collier
email: james.collier@vib.be
language: en

font: Noto Sans Egyptian Hieroglyphs, Noto Sans Ogham
-->

# Easy reproducible development environments

> Or, how do I get the tools I need for the projects I'm working on without these tools conflicting between projects?


## A typical project

 {{0}}
A web application with 3 components:

1. An API server written in Python
2. A Typescript UI running in a web browser
3. A CLI interface written in Rust

 {{1}}
To work on any of these components I at least need:

 {{2}} Git

{{3}} A text editor with an LSP client

{{4}} But also a lot more...

## The API Server

When working on this I need:

* A text editor with LSP client
* A Python LSP
* PostgreSQL
* Python and dependencies defined in `pyproject.yml`
* Formatter, linter

## The client UI

To work on this component I need:

* A text editor with LSP client
* A Typescript LSP
* Node.js + NPM
* Dependencies defined in `package.json`
* Formatter, linter

## The CLI

For this I need:

* A text editor with LSP client
* Rust LSP
* Rust compiler and package manager
* Dependencies defined in `Cargo.toml`
* Formatter

## What's the problem?

* What if I need to develop from a new machine?
* What if my development laptop breaks?
* What if I need to onboard another developer?

## One of many...

{{0}}
Containers
==========
![Docker](media/docker.png)

{{1}}
Virtual Machines ![Virtual machine](media/VirtualBox.png)
================

{{2}}
Conda ![Conda](media/Conda.png)
=====

{{3}}
Ansible ![Ansible](media/Ansible.png)
=======

{{4}}
Guix  ![Guix](media/Guix.png)
====

{{5}}
Nix ![Nix](media/Nix.png)
===

## Devenv

![Devenv](media/Devenv.png)

[devenv.sh](https://devenv.sh) is a wrapper around Nix specifically to
create reproducible development environments

* Packages work back in time

  The package might not be available in distribution repositories anymore, but it will be available through Nix

* Inspect the dependencies (not like a container image)

Activate with:
==============
```bash
devenv shell  # Activate the development environment
```

## Direnv

[Direnv.net](https://direnv.net)

Per directory shell profile
===========================

* Set environment variables when entering a directory

* Load secrets from a password manager into environment variables

* Set up environment for _Devenv_ in the current shell

## Demo
{{0}}
!?[Direnv activation](media/terminal-demo-opt.mp4)

{{1}}
!?[Inside an editor](media/emacs-demo-opt.mp4)

## Start with Devenv
{{0}}
![devenv init](media/devenv-init.png)

{{1}}
```nix
{ pkgs, lib, config, inputs, ... }:
{
    packages = [ pkgs.git ];

    languages.python = {
        enable = true;
        version = 3.11.3;
        poetry.enable = true;
    };
}
```

## Run the app

```nix
{ pkgs, lib, config, inputs, ... }:
{
    packages = [ pkgs.git pkgs.entr ];

    languages.python = {
        enable = true;
        version = "3.11.3";
        poetry.enable = true;
    };

    tasks = {
        "ttfd:watch" = {
            exec = ''find -name *.py | entr -s "python -m pytest"'';
        };
    };
}
```

Run with:
=========
```bash
devenv tasks run ttfd:watch
```

## Run tests

```nix
{ pkgs, lib, config, inputs, ... }:
{
    packages = [ pkgs.git pkgs.entr ];

    languages.python = {
        enable = true;
        version = "3.11.3";
        poetry.enable = true;
    };

    tasks = {
        "ttfd:watch" = {
            exec = ''find -name *.py | entr -s "python -m pytest"'';
        };
    };

    enterTest = ''
        python -m pytest
    '';
}
```

Run with:
=========
```bash
devenv test
```

## Services

```nix
{ pkgs, lib, config, inputs, ... }:
{
    packages = [ pkgs.git pkgs.entr ];

    languages.python = {
        enable = true;
        version = "3.11.3";
        poetry.enable = true;
    };

    tasks = {
        "ttfd:watch" = {
            exec = ''find -name *.py | entr -s "python -m pytest"'';
        };
    };

    enterTest = ''
        python -m pytest
    '';

    services.postgres = {
        enable = true;
        package = pkgs.postgresql_15;
        initialDatabases = [{ name = "ttfd"; }];
    };
}
```

Run with:
=========
```bash
devenv up
```

## Summary
{{0}}
**Devenv** and **Direnv** are powerful tools with a lot of promise.

{{1}}
Has some advantages over alternatives

{{2}}
Has some disadvantages

{{3}}
Worth considering even for existing projects