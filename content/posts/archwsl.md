+++
title = 'archwsl'
date = 2025-10-14
draft = false
math = true
tags = ['computing']
summary = 'Installing Arch Linux on WSL.'
+++

This webpage was written using Arch on WSL set up using the steps outlined below!
There is [official documentation](https://wiki.archlinux.org/title/Install_Arch_Linux_on_WSL) for installing Arch Linux on WSL, but I think some items could be improved.
Before the [official](https://gitlab.archlinux.org/archlinux/archlinux-wsl) Arch Linux WSL project was released in April 2025, [yuk7](https://github.com/yuk7) maintained a fantastic community version called [ArchWSL](https://github.com/yuk7/ArchWSL).
The [docs](https://wsldl-pg.github.io/ArchW-docs/How-to-Setup/) for that project are helpful, but the order of things has changed a bit with the official packaged release, which is why I'm making this outline.

## Enabling virtualization

First, virtualization must be turned on in the UEFI or BIOS of your Windows machine, and the Virtual Machine Platform feature must be enabled.
See Microsoft's [docs](https://support.microsoft.com/en-us/windows/enable-virtualization-on-windows-c5578302-6e43-4b4b-a449-8ced115f58e1) for details.

## Installing WSL

In Windows 10 version 2004 and higher (Build 19041 and higher) or Windows 11, the Windows Subsystem for Linux Windows feature does not need to be enabled manually and WSL commands should work out of the box.
First, ensure WSL is up-to-date:

```powershell
wsl --update
```

You can then view the list of available distributions by typing:

```powershell
wsl -l -o
```

Now, run

```powershell
wsl --install -d archlinux
```

to install Arch Linux instead of the default distribution, which is Ubuntu.
More information can be found on [Microsoft's docs](https://learn.microsoft.com/en-us/windows/wsl/install).

## Setup

### Setting the root password

Once Arch has been downloaded and installed, the distro should launch and present a root console, which should look something like `[root@pcname username]#`.
Before doing anything else, update the root password:

```bash
passwd
```

### Initializing the pacman keyring

Since the pacman keyring requires elevated permissions to be created, we must use root to initialize it.
(Otherwise, we would need sudo, which needs pacman to be installed, which needs sudo, etc.)

```bash
pacman-key --init
pacman-key --populate
pacman -Sy archlinux-keyring
```

Once this is done, immediately run a full system upgrade:

```bash
pacman -Syu
```

### Installing sudo

While still logged in as root, install the sudo package:

```bash
pacman -S sudo
```

### Setting up the default user

Create the sudoers file:

```bash
mkdir /etc/sudoers.d
touch /etc/sudoers.d/wheel
```

Then, setup the wheel:

```bash
echo "%wheel ALL=(ALL) ALL" > /etc/sudoers.d/wheel
```

Add the new user:

```bash
useradd -m -G wheel -s /bin/bash {username}
```

Set a password for the new user:

```bash
passwd {username}
```

Exit the distro and set the default user when starting WSL:

```bash
exit
wsl --manage archlinux --set-default-user {username}
```

## Quality of life

### Setting the locale

To avoid issues with future package installs, it's best to set the locale now.
To set the locale to US English (or whatever locale you like, see [the wiki page](https://wiki.archlinux.org/title/Locale) for more), run:

```bash
sudo localectl set-locale LANG=en_US.UTF-8
```

### Updating pacman mirrors

First, back up the current mirrorlist in case of bad overwrites:

```bash
sudo cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
```

Install `reflector`:

```bash
sudo pacman -S reflector
```

Use reflector to save a list of the 20 most recently syncronized mirrors (located in the US and using the https protocol) into the pacman mirrorlist, sorted by download rate:

```bash
sudo reflector --verbose --latest 20 --country US --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

### Enabling extra pacman features

Install your editor of choice:

```bash
sudo pacman -S neovim
```

Turn on coloring and parallel downloads in pacman (parallel downloads should be enabled by default) by opening `/etc/pacman.conf` and uncommenting the two lines:

```bash
# Misc options
Color
...
ParallelDownloads = 5
```

For some reason, the Arch WSL install comes with `NoProgressBar` enabled, which I don't understand.
To enable progress bars on package installs, just comment out that line.

## Personalizing the shell

I happen to like the [starship.rs](https://starship.rs/) shell prompt, so I'll install it:

```bash
sudo pacman -S starship
```

Apply the changes:

```bash
source .bashrc
```

I like the [Starship Nerd Fonts preset](https://starship.rs/presets/nerd-font), which can be enabled like so:

```bash
starship preset nerd-font-symbols -o ~/.config/starship.toml
```

## Setting up git

Install:

```bash
sudo pacman -S git
```

Follow the steps in the [first time setup](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup). In short, run:

```bash
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
git config --global core.editor nvim
git config --global init.defaultBranch main
```

If you use GitHub and would like to keep your personal email private (i.e., not attach it to commits), go to [GitHub settings](https://github.com/settings/emails), set your email to private, and GitHub will give you an email specifically for authoring commits, merges, etc.

You can view the global config using:

```bash
git config --list
```

This might throw `error: cannot run less: No such file or directory`, which happens simply because `less` is not installed (the Arch WSL distro ships pretty much completely devoid of all common packages).
Just install it if needed.

### SSHing into GitHub

Use `ssh-keygen` to generate public and private SSH keys for your GitHub email:

```bash
ssh-keygen -t ed25519 -C "your_email@users.noreply.github.com"
```

Select a directory for the key to live in and create a password.
Get the public key using `cat ~/.ssh/id_ed25519.pub` and go to [GitHub's SSH settings](https://github.com/settings/keys) to add the public key.
Once it's added, you're good to go, but you'll have to re-enter your SSH password every time you push to or clone a repository, which can get annoying.
To permanently activate the key locally, get the `keychain` tool:

```bash
sudo pacman -S keychain
```

Add the following line pointing to to your local private key to `.bashrc`:

```bash
eval "$(keychain -q --eval ~/.ssh/id_ed25519)"
```

This will quietly execute the `keychain` command whenever a new shell is spawned and automatically make the key available for use.
To test if everything is working, try:

```bash
ssh -T git@github.com
```

You should get a message like `Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.`, which means everything is working.
Just make sure to clone repos using SSH instead of HTTPS, and your credentials will automatically be certified.

## My workflow

Everything included in and below this section pertains to my workflow specifically, which I'm documenting here mostly for my own sake.

### TL;DR and Man Pages

Documentation for command-line utilities is essential.
The [tealdeer](https://github.com/tealdeer-rs/tealdeer) package compiles community-made and easy to understand usage examples for a bunch of commands.
For more advanced usage, `man` pages are indispensable:

```bash
sudo pacman -S tealdeer man
```

The [wikiman](https://github.com/filiparag/wikiman) package provides the ability to browse the Arch and Gentoo wikis offline.
To install it and the offline Arch wiki:

```bash
sudo pacman -S wikiman arch-wiki-docs
```

### Hugo

This site is written in Hugo, and, as expected, relies on the Hugo package being installed.
I also use a makefile to automatically start a server instance with my preferred flags enabled, which needs make:

```bash
sudo pacman -S hugo make
```

### Python

Python is one of the few packages that comes pre-installed, but this is a system-wide install that should never be used to actually develop Python programs (it's used as a dependency for lots of other packages).
I quite like the `uv` tool, which is a package and project manager for Python written in Rust (so it's blazingly fast or whatever).
Install it with:

```bash
sudo pacman -S uv
```

{{< notice info >}}

Tools like `uv` and `rustup` have self-updating commands like `uv self update` that will not work if installed via pacman.
In my view, this is a good thing, since all updates should be handled by the system-wide package manager.

{{< /notice >}}

#### PySide6

When trying to run a program that uses PySide6, the error

```bash
ImportError: libGL.so.1: cannot open shared object file: No such file or directory
```

will come up if the requisite OpenGL libraries are not installed.
The [`glu`](https://archlinux.org/packages/extra/x86_64/glu/) package contains the `libGLU.so.1` file, which will work just fine.
Attempting to run the program again will produce the error

```bash
ImportError: libfontconfig.so.1: cannot open shared object file: No such file or directory
```

Just install `fontconfig`.
The next is

```bash
ImportError: libxkbcommon.so.0: cannot open shared object file: No such file or directory
```

which is contained in, you guessed it, the `libxkbcommon` package.
After installing this, the application should load the GUI, but the fonts might be broken.
To fix this, I referenced [this thread](https://www.reddit.com/r/archlinux/comments/rm3kch/which_fonts_do_you_guys_actually_install/), which recommends installing the `noto-fonts` package.
Installing this package fixed all of my font issues, and GUI development in Python is possible through WSL!

#### Profiling

The [snakeviz](https://jiffyclub.github.io/snakeviz/) tool interfaces with the built-in Python [cProfile](https://docs.python.org/3/library/profile.html) tool to show total and cumulative file / function runtimes in a browser window.
To generate a profile for viewing, run:

```bash
python -m cProfile -o main.prof main.py
```

If you're using uv, the `uvx` command can be used to run tools, in this case:

```bash
uvx snakeviz main.prof
```

Another great tool is [pyinstrument](https://github.com/joerick/pyinstrument), which is a statistical profiler that has less overhead than cProfile.
You can also run this with `uvx`, but I've had issues with it recognizing the virtual environment for some reason.
Instead, I just do:

```bash
source .venv/bin/activate
uv pip install pyinstrument
```

To get an HTML output, run:

```bash
pyinstrument -r html main.py
```

### Julia

Unfortunately, the Julia version manager `juliaup` is not available through pacman.
You can either install it from the AUR or use the install script.
The docs say that there are ["drawbacks"](https://github.com/JuliaLang/juliaup?tab=readme-ov-file#software-repositories) to installing from repositories, but I'm not sure what they mean by this.
In either case, you'll need to inspect the installation script before actually installing it.
For the AUR, this amounts to reading the PKGBUILD script (`paru` opens this script for inspection by default).

```bash
paru -S juliaup
```

{{< notice warning >}}

When you're installing something from the AUR, *always* read the PKGBUILD script.
They're typically quite short and tend to pull sources from GitHub, so they're easy to vet.
If you're installing something by `curl`ing a shell script, *never* pipe the script directly into bash.
If you must install software this way, redirect it to a file, read the file, and execute it once you're sure it's okay:

```bash
curl -fsSL https://install.julialang.org > install.sh
nvim install.sh
bash install.sh
```

Typically, these custom bash scripts are much longer and more difficult to understand than their AUR PKGBUILD counterparts since they have to consider a wider range of systems.
Even if the script is "easy to read," it might just download a binary from some unknown source and execute it, preventing you from auditing the actual source itself.
I really don't recommend you install any piece of software this way.
Auditing the script by simply `curl`ing the output to bash, as many people suggest, is [not advised](https://archive.is/xLSDn) because the payload might be changed if the pipe is detected server-side.

{{< /notice >}}

#### Neovim LSP

Attempting to install the julials language server through [Mason](https://github.com/mason-org/mason.nvim) does not work at the time of writing (25/08/15).
There is an [open issue](https://github.com/mason-org/mason-lspconfig.nvim/issues/582) referencing this problem, but until it gets resolved, there is a manual solution.
User `danielwe` on the Julia discourse board posted a [solution](https://discourse.julialang.org/t/neovim-languageserver-jl-crashing-again/130273/3) which I'll reiterate here.
Create a shared Julia environment called `@nvim-lspconfig` and manually install the `LanguageServer.jl` package inside it:

```julia
julia --project=@nvim-lspconfig -e 'using Pkg; Pkg.add("LanguageServer")'
```

Add `vim.lsp.enable("julials")` inside your `nvim-lspconfig.lua` file.
Neovim's LSP will automatically find the language server at the installed location.

#### Julia workflow

Open a Julia REPL in your home directory and add Revise.jl to the global Julia environment:

```julia
(@v1.12) pkg> add Revise
```

Next, create or modify the `~/.julia/config/startup.jl` file to enable Revise.jl if Julia is started through the REPL:

```julia
if isinteractive()
    try
        using Revise
    catch e
        @warn "Error initializing Revise" exception = (e, catch_backtrace())
    end
end
```

Revise allows you to change your package code without restarting the Julia session.
While working on a package, you can minimize the latency caused by the JIT compiler by creating a script file separate from the package itself.
Create a new script directory, open a Julia REPL, activate it, and install the local package that you're working on:

```julia
(@v1.12) pkg> activate .
(@v1.12) pkg> dev ~/path/to/package/
```

Activating an environment means that it is modified when executing package commands in the REPL (instead of the global environment).
To check which packages are installed in a specific environment, open a REPL and type

```julia
(@v1.12) pkg> status
```

To open a REPL using the current project directory as the environment, use

```bash
julia --project
```

Opening a REPL without this flag will use the global environment by default.
Now, you can open a REPL in the script directory with `julia --project` and keep it open.
To run the file, you can use

```julia
julia> include("script.jl")
```

The [iron.nvim](https://github.com/Vigemus/iron.nvim) plugin allows for an interactive REPL over Neovim, which is quite nice.
Once the packages used in your script have been precompiled once, the overhead reduces significantly and everything executes quickly as long as the REPL is kept active.
For more details, see this excellent [video](https://www.youtube.com/watch?v=_3vJSBk0Bls) by Jacobus.

### C and C++

Luckily, C and C++ are easy:

```bash
sudo pacman -S gcc
```

### Fortran

GCC has a Fortran frontend:

```bash
sudo pacman -S gcc-fortran
```

### Rust

Install via pacman:

```bash
sudo pacman -S rustup
```

### Go

Golang itself only provides archives, not a self-updating installer:

```bash
sudo pacman -S go
```

### $\LaTeX$

#### $\TeX$ Live

These instructions are a mix between the [$\TeX$ Live docs](https://tug.org/texlive/quickinstall.html) and a [user-written page](https://wiki.archlinux.org/title/User:Ms/TeX_Live_Manual_Installation) on the Arch wiki.
First, create and append an environment variable for the installation location:

```bash
export SHELL_ENV=$HOME/.bashrc
echo "TEXLIVE=$HOME/texlive/2025" >> $SHELL_ENV
```

Navigate to a temporary directory and install the latest version of $\TeX$ Live:

```bash
cd /tmp
wget https://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz
zcat < install-tl-unx.tar.gz | tar xf -
cd install-tl-2*
```

Before running the installer, make sure to point it to a directory residing in `/home/`:

```bash
export TEXLIVE_INSTALL_PREFIX=~/texlive
perl ./install-tl
```

Once the installer loads, the `TEXDIR` location identified by the installer should read something like `/home/username/texlive/2025`.
The reason we go through these steps is to ensure that $\TeX$ Live does not get installed in the usual location of `/usr/local/`.
Since the $\TeX$ Live package manager resides in the $\TeX$ Live installation directory, it requires root permissions each time it's invoked if $\TeX$ Live is installed in `/usr/local/`, so it's better to avoid this entirely and install in `/home/`.

If you want every package in $\TeX$ Live to be installed, then install `scheme-full`.
Personally, I don't want or need 9 GB of storage space being taken up by random $\TeX$ packages, so `scheme-basic`, which includes $\LaTeX$ and some essential packages, is good enough.
More space can be saved by not installing the macro/font doc and source trees, but this [isn't recommended](https://tug.org/texlive/doc/texlive-en/texlive-en.html).

Finally, the environment variables need to be appended in `.bashrc`:

```bash
TEXLIVE=/home/username/texlive/2025
export PATH=$TEXLIVE/bin/x86_64-linux:$PATH
export MANPATH=$TEXLIVE/texmf-dist/doc/man:$MANPATH
export INFOPATH=$TEXLIVE/texmf-dist/doc/info:$INFOPATH
```

#### Personal packages

To get $\TeX$ Live to recognize and use locally installed packages, first find the directory in which the files are expected:

```bash
kpsewhich -var-value=TEXMFHOME
```

This should output something like `/home/username/texmf`.
That directory probably won't exist yet, so make it along with the subdirectories that $\TeX$ Live will look for `.sty` and `.cls` files on:

```bash
mkdir /home/username/texmf/tex/latex/local
```

Local files can now be added to this directory.
In my case, I have a package on GitHub that I can clone down:

```bash
cd /home/username/texmf/tex/latex/local
git clone https://github.com/nategphillips/natex
```

Finally, run `texhash /home/username/texmf` to ensure that the directory can be found.

#### $\LaTeX$ Workshop in VS Code

If you installed $\TeX$ Live using `scheme-basic`, the first package you'll need to start compiling is `latexmk`:

```bash
tlmgr install latexmk
```

Before trying to compile anything using LaTeX Workshop, close all instances of WSL and type:

```powershell
wsl --shutdown
```

in a Windows terminal to ensure the PATH variables are updated correctly.
Restarting WSL and compiling a simple $\LaTeX$ document without any dependencies should then work.

#### $\LaTeX$ Indent

A useful formatting tool is `latexindent`, which can be used in tandem with $\LaTeX$ Workshop.
Install it:

```bash
tlmgr install latexindent
```

Calling `latexindent main.tex` on any $\TeX$ file will give the output

```bash
Can't locate YAML/Tiny.pm in @INC (you may need to install the YAML::Tiny module)
```

The best place to go to resolve these issues are the Arch [man pages](https://man.archlinux.org/), on which you can type in any string, like `YAML::Tiny`, to figure out which package contains it.
In this case, searching gives us [this](https://man.archlinux.org/man/YAML::Tiny.3pm) man page, which indicates that the `perl-yaml-tiny` package will solve our issue, so install it:

```bash
sudo pacman -S perl-yaml-tiny
```

Running `latexindent main.tex` again gives the error

```bash
Can't locate File/HomeDir.pm in @INC (you may need to install the File::HomeDir module)
```

Using the same process, install

```bash
sudo pacman -S perl-file-homedir
```

Everything should now work!

#### Installing Packages

Attempting to compile a file without having the correct dependencies will produce an error like

```bash
! LaTeX Error: File `derivative.sty' not found.
```

To install packages, just use `tlmgr` like normal:

```bash
tlmgr install derivative
```

Sometimes, you'll get a message like `tlmgr install: package authblk not present in repository.` when attempting to install a package.
The most common cause of this is that the dependency is part of a larger package inside of $\TeX$ Live.
To see which package you need to install, go to [CTAN](https://www.ctan.org) and type in the name of the package you want---it will give you the name of the larger container in which the package is bundled for $\TeX$ Live.

I use `biblatex` with the `ieee` style for citations in my own work, which requires the [`biblatex-ieee`](https://ctan.org/pkg/biblatex-ieee) package.
If you don't have this and try to use `ieee` as the style, even if `biblatex` is present, it will throw some absolutely nonsense errors.

If you use `biber` as the `biblatex` backend, make sure it's installed as well, otherwise the errors thrown there make similar (no) sense.
For `biber` to work, it needs access to the `libcrypt.so.1` file, which is contained in the [`libxcrypt-compat`](https://archlinux.org/packages/core/x86_64/libxcrypt-compat/) package from core.
For general troubleshooting of missing files, the `pacman -F` command is really useful, as it shows the libraries which contain the file in question.

### Terminal Font

I like using [Nerd Fonts](https://www.nerdfonts.com/) in the terminal since they have nice little icons that prompts and other CLI tools sometimes use.
But how does one get both a Nerd Font and emoji working in the terminal at the same time?
First, ensure that your desired Nerd Font is installed (Arch provides a [long list](https://archlinux.org/groups/x86_64/nerd-fonts/)) along with the Noto font emojis:

```bash
sudo pacman -S ttf-firacode-nerd noto-fonts-emoji
```

Next, we'll need to actually configure the font family that the terminal will use.
First, `grep` the actual name of the two font families we've just installed:

```bash
fc-list : family | grep "Fira"
```

This should return a bunch of font families, any of which can be used.
Edit (or create) the `~/.config/fontconfig/fonts.conf` file to include something like the following minimal example:

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "urn:fontconfig:fonts.dtd">
<fontconfig>
    <alias>
        <family>monospace</family>
        <prefer>
            <family>FiraCode Nerd Font Mono</family>
            <family>Noto Color Emoji</family>
        </prefer>
    </alias>
</fontconfig>
```

The Arch wiki has several [font configuration examples](https://wiki.archlinux.org/title/Font_configuration/Examples#Default_fonts) to get ideas for what this could look like in the case of more complex setups.
In my case, I'm using alacritty, so the newly created font family needs to be specified in `~/.config/alacritty/alacritty.toml`:

```toml
[font]
size = 11.0

[font.normal]
family = "monospace"
style = "Regular"
```

Some configs also specify configurations for bold, italic, and bold italic separately, but I've found this can actually break those font types.
To test if everything is working, you can use the following terminal escape codes from this [StackExchange post](https://askubuntu.com/questions/528928/how-to-do-underline-bold-italic-strikethrough-color-background-and-size-i):

```bash
echo -e "\e[1mbold\e[0m"
echo -e "\e[3mitalic\e[0m"
echo -e "\e[3m\e[1mbold italic\e[0m"
echo -e "\e[4munderline\e[0m"
echo -e "\e[9mstrikethrough\e[0m"
```
