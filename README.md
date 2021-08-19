# hearthstone-linux

Craft your own linux native Hearthstone client

*Tested with client version 21.0.3.91040.88998*

Hearthstone is based on the Unity engine, that allows to deploy to multiple platforms, including linux. The platform specific engine files are mostly generic, so let's take the game files and run them with Unity's linux binaries. Taking the windows version does not work, since it was exported only with Direct3D renderer enabled, but the MacOS version uses the OpenGLCore renderer, that we can perfectly use on linux!

Even though we don't have to modify any of the game internals, please note that this is unofficial and you might risk a ban when using this method.

None of the proprietary files are distributed here, you can retrieve them from the official locations for free.

Hearthstone is ©2014 Blizzard Entertainment, Inc. All rights reserved. Heroes of Warcraft is a trademark, and Hearthstone is a registered trademark of Blizzard Entertainment, Inc. in the U.S. and/or other countries.

## Installation

### 1) Clone the repository

`$ git clone --recursive https://github.com/0xf4b1/hearthstone-linux.git && cd hearthstone-linux`

### 2) Hearthstone installation

Just execute the crafting script. You may need to install some required build files.

```
$ ./craft.sh
```

If you have an up-to-date Hearthstone installation folder from your Mac `/Applications/Hearthstone` somewhere in place, you can specify the path as the first argument and skip the download. If you have Unity files not in `~/Unity`, you can specify the path as second argument.

```
$ ./craft.sh [<path of the MacOS installation>] [<Unity path>]
```

<details>
  <summary>Download Hearthstone via keg manually</summary>

To download the required game files, [keg](https://github.com/HearthSim/keg) from the HearthSim project can be used. It's an implementation of Blizzard's NGDP protocol and allows to mirror the contents of the CDN. A slightly modified version is linked into this repository, that allows to download only the needed files for the installation. Use the `ngdp` command from `keg/bin/ngdp`.

Initialize the repository

```
$ ngdp init
```

Add the Hearthstone remote

```
$ ngdp remote add hsb
```

Update the metadata

```
$ ngdp fetch hsb --metadata-only
```

List available versions

```
$ ngdp inspect hsb
```

Fetch the game files

```
$ ngdp fetch hsb --tags OSX --tags enUS --tags Production
```

Install the current version

```
$ ngdp install hsb 18.6.0.63543 --tags OSX --tags enUS --tags Production
```
</details>

<details>
  <summary>Download Unity Engine files manually</summary>

* Download Unity Hub from [here](https://public-cdn.cloud.unity3d.com/hub/prod/UnityHub.AppImage)

`$ curl -O https://public-cdn.cloud.unity3d.com/hub/prod/UnityHub.AppImage`

* Make the file executable

`$ chmod +x ./UnityHub.AppImage`

* Start Unity Hub with the following url

`$ ./UnityHub.AppImage unityhub://2018.4.10f1/a0470569e97b`

* You can ignore the licensing stuff that may show up, wait until the installation window appears. You don't need to install any of the additional modules.

By default, it should download the files into your home directory in `~/Unity`.
</details>

### 3) Login

Use the `login` app to retrieve the authentication token for your account.

<details>
  <summary>Alternatively retrieve it manually</summary>

Visit the website https://eu.battle.net/login/en/?app=wtcg, enter your account credentials and you will get the authentication token in the browser's address bar via redirection, similarly to this:

```
http://localhost:0/?ST=XX-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX-XXXXXXXXX
```

Store the token:

```
$ mono token.exe XX-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX-XXXXXXXXX
```
</details>

### 4) Launch the game!

Launch the game via the desktop entry or directly via the executable.

```
$ Bin/Hearthstone.x86_64
```

The game runs perfectly besides some features, like the in-game shop, due to missing libraries.

<details>
  <summary>If you are interested what's going on behind the scenes, you can continue reading.</summary>

The `craft.sh` script copies and rearranges the needed files for your linux client and additionally does the following tasks:

A file named `client.config` is used by the client for configuration. It will be added with some predefined values, including the option `Aurora.ClientCheck=false` to be able to run the client without the Launcher.

Since we use the MacOS version, it has some platform specific dependencies we don't have on linux, that prevent the game from launching. The script builds some very simple stubs for the missing libraries, that are `CoreFoundation` and `OSXWindowManagement`.

When starting the game client without the Launcher, the game is stuck on the title screen and does not offer something like a login. This happens even on MacOS with the normal installation. But since it tries to read an existing login token from the MacOS registry, the `CoreFoundation` stub is used to provide our manually requested token.

The game tries to read the authentication token from the registry AES encrypted with some static parameters. Based on that logic, a small token tool will be built that encrypts a provided webtoken and stores it in a file named `token`, where the `CoreFoundation` stub reads it from.
</details>
