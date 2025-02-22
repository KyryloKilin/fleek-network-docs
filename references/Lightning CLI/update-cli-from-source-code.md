---
title: Update CLI from source code
slug: update-cli-from-source-code
hide_title: true
tags:
- references
- help
- update
- upgrade
- fix
---

import Author from '@site/src/components/Author';
import SetUserPathInConfigToml from '../../guides/partials/_set-user-path-config-toml.mdx';
import NoteExecStartFlagCConfigPath from '../../guides/partials/_note_execstart-flag-c-config-path.mdx';
import FindAndReplaceConfigWithUserPaths from '../../guides/partials/_find-and-replace-config-with-user-paths.mdx';

:::tip
Although the step-by-step instructions described here are simple to follow, this process is available as an automated script. To use it execute the following command in your server terminal and follow the instructions:

```sh
curl -sS https://get.fleek.network/update | bash
```
:::

## Switch to the installation user

Switch to the username you've used throughout the installation process.

```sh
su <USERNAME>
```

For example, if you used the username `lgtn` it'll look like the following command:

```sh
su lgtn
```

## Change directory to the source code

If you have installed it via the recommended process or instructions, then the default location where the [source code](https://github.com/fleek-network/lightning) is stored is `~/fleek-network/lightning`.

```sh
cd ~/fleek-network/lightning
```

:::tip
Notice that we use `~`, which refers to `$HOME`. You must use the username used for the installation process. For example, on [Ubuntu](/docs/node/requirements#server) if you use the username `lgtn`, the pathname for `$HOME` is `/home/lgtn`.
:::

## Checkout to branch

Make sure that you are checked in to the correct branch. For the current testnet phase that'd be `testnet-alpha-0`. If you use any other branch name, your node will not function correctly. Use the branch name `testnet-alpha-0`.

```sh
git checkout testnet-alpha-0
```

:::tip
We try to update our documentation promptly but sometimes are a bit behind on any changes we might make in real-time. If you find any typos, such as the wrong branch name, help us by letting us know! Find us in Fleek Network section of our [Discord](https://discord.gg/fleekxyz).
:::

## Pull the latest changes

Before make sure that you stash or clear any changes you may have in the working directory, as otherwise, `git` will let you know about local changes–if you'd like to learn more about it read the [git stash document](https://git-scm.com/docs/git-stash).

A quick way to clean is to `stash` the changes, for example:

```sh
git stash 
```


To pull the latest changes use the `git pull` command, as follows:

```sh
git pull origin testnet-alpha-0
```

Alternatively, to have to stash and pull, you can reset the repository to the origin.

```sh
git fetch origin testnet-alpha-0
git reset --hard origin/testnet-alpha-0
git clean -f
```

:::tip
We are using the branch named `testnet-alpha-0`, which is specific to the early testnet launch. Change to the correct branch name according to needs. For example, in the future the mainnet version will go on branch name `main`.
:::

## Build binary from the source

To build the binary from the source code, we execute the cargo build command:

```sh
cargo +stable build --release
```

## Update the symlink

Start by removing the existing one:

```sh
sudo rm -f "/usr/local/bin/lgtn"
```

Create a new symlink that links the new build binary to `/usr/local/bin/lgtn`, as follows:

```sh
sudo ln -s ~/fleek-network/lightning/target/release/lightning-node /usr/local/bin/lgtn
```

## Set user path in config.toml

<SetUserPathInConfigToml />

## Update the systemd service unit

Open and edit the `/etc/systemd/system/lightning.service` file.


1) Replace `<YOUR-USERNAME>` with the username. For example, in the [documentation](/docs/node/install#create-a-user) we use the username `lgtn`.

2) Make sure that the `ExecStart` is set correctly

```sh
[Unit]
Description=Fleek Network Node lightning service

[Service]
User=<YOUR-USERNAME>
Type=simple
MemoryHigh=32G
RestartSec=15s
Restart=always
ExecStart=lgtn -c /home/lgtn/.lightning/config.toml run
StandardOutput=append:/var/log/lightning/output.log
StandardError=append:/var/log/lightning/diagnostic.log
Environment=TMPDIR=/var/tmp

[Install]
WantedBy=multi-user.target
```

<NoteExecStartFlagCConfigPath />

When complete make sure the file is saved. Followed by a systemctl daemon reload:

```sh
sudo systemctl daemon-reload
```

## Clear the data

```sh
rm -rf ~/.lightning/data
```

Depending on how you control the system, this might need **sudo**.

```sh
sudo rm -rf ~/.lightning/data
```

## Update the `config.toml` with user home path

<FindAndReplaceConfigWithUserPaths />

## Restart the service

Once the cargo build process is completed, you have to restart the service. We're assuming you are using non-root user as [recommended](/docs/node/install#create-a-user), you won't use **sudo** to start the service. The command will look as follows:

```sh
sudo systemctl restart lightning
```

:::tip
If you have installed the Fleek Network lightning manually, the [installation instructions](/docs/node/install#systemd-service-setup) recommended setting up a systemd service for the Fleek Network process. If you haven't, you're advised to check the instructions provided.
:::

## Health checkup

Do a quick health check by running:

```sh
curl -w "\p" localhost:4069/health
```

If successful, you should get the response `OK`, as follows:

```sh
OK
```

<Author
    name="Helder Oliveira"
    image="https://github.com/heldrida.png"
    title="Software Developer + DX"
    url="https://github.com/heldrida"
/>
