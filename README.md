# WiseOS - /anki folder

**This is where the personality code for WiseOS exists.**

For the entire OS, check out the [wise-os](https://github.com/Phu1237/vector-wise-pod) repo. This repo, `wise-os-victor`, is a submodule of that, and just builds the /anki folder which goes into that OS. However, `wise-os-victor` can still be built standalone and deployed to a robot which is running a good base OTA. This is recommended for developers.

Most changes happen in this repo. If one wants to, for instance, add a new feature; this is where they'd do it.

If you want to add a program to the OS, do that in [wise-os](https://github.com/Phu1237/vector-wise-pod).

Check [vector-docs](https://os-vector.github.io/vector-docs) for more information about the source code leak, what we can do with this, and general Vector info.

## What is WiseOS?

WiseOS serves as a stable, up-to-date, easily-buildable base for CFW, and is a continuation of Anki's work.

Any feature added here should be somewhat objectively applicable to other CFW. Feel free to make a PR. Wise encourages PRs which add things like 3rd-party library upgrades, new behaviors which Anki might have planned to add, code documentation, and optimizations. Wise discourages PRs which significantly alter the experience, like transforming him into a (soulless) GPT box or making him into Cozmo. If you want to do something like that, make your own CFW (instructions in [vector-docs](https://os-vector.github.io/vector-docs)).

## Building

`wise-os-victor` can be built standalone on most Linux distros (arm64 or amd64), and on macOS (arm64 only, for now).

Docker is recommended for now (especially if you have a weird or old Linux distro installed), though bare metal works nicely too.

Note that if you have built in Docker before and want to build on bare metal now (or vice-versa), you should do a [clean](#cleaning) build.

Click an option below for instructions.

<details>
<summary><strong>Docker: x86_64 or arm64 Linux</strong></summary>
<br />

- Prerequisites: Make sure you have `docker` and `git` installed.

1. Clone the repo and `cd` into it:

```
cd ~
git clone --recurse-submodules https://github.com/os-vector/wise-os-victor
cd wise-os-victor
```

2. Make sure you can run Docker as a normal user. This will probably involve:

```
sudo groupadd docker
sudo gpasswd -a $USER docker
newgrp docker
sudo chown root:docker /var/run/docker.sock
sudo chmod 660 /var/run/docker.sock
```

3. Run the build script:
```
cd ~/wise-os-victor
./build/build-v.sh
```

</details>

<details>
<summary><strong>Bare Metal: x86_64 or arm64 Linux</strong></summary>
<br \>

- Prerequisites:
  - glibc 2.27 or above - this means anything Ubuntu 18.04 and newer will work.
  - The following packages need to be installed: `git wget curl openssl ninja g++ gcc pkg-config ccache`
```
# Arch Linux:
sudo pacman -S git wget curl openssl ninja gcc pkgconf ccache
# Ubuntu/Debian:
sudo apt-get update && sudo apt-get install -y git wget curl openssl ninja-build gcc g++ pkg-config ccache
# Fedora
sudo dnf install -y git wget curl openssl ninja-build gcc gcc-c++ pkgconf-pkg-config ccache
```

1. Clone the repo and `cd` into it:

```
cd ~
git clone --recurse-submodules https://github.com/os-vector/wise-os-victor
cd wise-os-victor
```

2. Source `setenv.sh`:
```
source setenv.sh
```

3. (OPTIONAL) Run this so you don't have to perform step 2 every time:
```
echo "source \"$(pwd)/setenv.sh\"" >> $HOME/.bashrc
```

4. Build:
```
vbuild
```

</details>

<details>

<summary><strong>macOS (M-series only)</strong></summary>
<br />

- Prereqs: Make sure you have [brew](https://brew.sh/) installed.
  -  Then: `brew install ccache wget upx ninja`

1. Clone the repo and cd into it:

```
cd ~
git clone --recurse-submodules https://github.com/os-vector/wise-os-victor
cd victor
```

2. Run the build script:
```
cd ~/wise-os-victor
./build/build-v.sh
```

</details>

## Deploying

1. Install WiseOS on your robot.
2. Get your robot's IP through CCIS:
  - 1. Place your robot on the charger
  - 2. Double click the button
  - 3. Lift the lift up then down
  - 4. Write down the IP address somewhere
  - 5. Lift the lift up then down again to exit CCIS
3. One of the following:

<details>
<summary><strong>(Docker: x86_64 or arm64 Linux) or (macOS M-series)</strong></summary>
<br \>

- Run:

```
./build/deploy-v.sh
```
</details>

<details>
<summary><strong>Bare Metal: x86_64 or arm64 Linux</strong></summary>
<br \>

- Run:

```
vdeploy
```
</details>

## Cleaning

99% of the time, if you're working on a behavior or something, you don't need to clean any build directories. The CMakeLists are correctly setup to properly rebuild the code which needs to be rebuilt upon a file change.

If you do want to clean anyway:

<details>
<summary><strong>(Docker: x86_64 or arm64 Linux) or (macOS M-series)</strong></summary>
<br \>

- Run:

```
./build/clean.sh
```
</details>

<details>
<summary><strong>Bare Metal: x86_64 or arm64 Linux</strong></summary>
<br \>

- Run:

```
vclean
```
</details>
