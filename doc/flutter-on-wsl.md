# Running ClojureDart and Flutter on Windows Subsystem for Linux

 > This guide is written for WSL 2 using an Ubuntu distribution.

> If you need to install or upgrade from WSL 1, here is the [documentation](https://docs.microsoft.com/en-us/windows/wsl/install).

The setup is as follow:

- On WSL: Flutter, Dart, Clojure and ClojureDart
- On Windows `adb` and `Android Studio` (for the android device emulator)

This way you will be able to build and run `Android` and `Linux` app on your Windows operating system.

## 1. Setup on WSL

> Some tools needs to be manually installed. I've choose to install them in `~/.local/opt/`. This is somewhat arbitrary and you may choose the path of your choice. Just update the following commands accordingly.

### 1.1. Java JDK

Java Open JDK is the first dependency, if not already installed, run:

```bash
sudo apt update && sudo apt install default-jdk
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin
```

### 1.2. Clojure

Install Clojure, if not already installed:

```bash
sudo apt-get update && apt-get install clojure
```

### 1.3. Android SDK

We need to install all the tools required for Android development: `sdkmanager`, `platform-tools` & `adb`.

#### 1.3.1. Command Line Tools

```bash
mkdir -p ~/.local/opt/android/cmdline-tools;
mv ~/.local/opt/android/cmdline-tools;
wget https://dl.google.com/android/repository/commandlinetools-linux-8512546_latest.zip -O latest.zip;
unzip latest.zip;
rm -rf latest.zip;
export ANDROID_HOME=$HOME/.local/opt/android;
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/bin;
```

> The URL of the last version of `cmdline-tools` can be found [here](https://developer.android.com/studio#command-tools).

If everything went well you should be able to run:
`sdkmanager --version`

> If it's not working, it is probably a `$PATH` issue.

#### 1.3.2. Platform Tools

```bash
sdkmanager --install "platform-tools";
export PATH=$PATH:$ANDROID_HOME/platform-tools;
```

If everything went well you should be able to run `adb`:

```bash
% adb
Android Debug Bridge version 1.0.41
Version 33.0.2-8557947
Installed as /home/USER/.local/opt/android/platform-tools/adb
```

#### 1.3.3. Android images

Install Android images and build tools

```bash
sdkmanager --install "system-images;android-29;google_apis;x86" "platforms;android-29" "build-tools;29.0.3";
sdkmanager --licenses;
```
<!-- @todo check android version for android 12 -->

### 1.4. Flutter SDK

```bash
mkdir -p ~/.local/opt/flutter;
mv ~/.local/opt/flutter;
wget https://storage.googleapis.com/flutter_infra/releases/stable/linux/flutter_linux_1.22.5-stable.tar.xz -O flutter_latest.tar.xz;
tar xf flutter_latest.tar.xz;
export FLUTTER_ROOT=$HOME/.local/opt/flutter;
export PATH=$FLUTTER_ROOT/bin:$PATH;
```

> The URL of the latest version of Flutter can be found [here](https://docs.flutter.dev/get-started/install/linux#install-flutter-manually).

You should be able to run `flutter --version`:

```bash
% flutter --version
Flutter 3.0.4 • channel stable • https://github.com/flutter/flutter.git
Framework • revision 85684f9300 (5 days ago) • 2022-06-30 13:22:47 -0700
Engine • revision 6ba2af10bb
Tools • Dart 2.17.5 • DevTools 2.12.2
```

### 1.5 Persisting env

The exports we made needs to be persisted if you want to be able to reuse them.
You can do this by adding previous exports in you shell profile:`.profile` for `Bash`, `.zprofile` for `zsh`, etc...
Here is an exemple for `Bash`

```bash
echo '# add Java to $PATH
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin

# add Android cmdline-tools and platform-rool to $PATH
export ANDROID_HOME=$HOME/.local/opt/android;
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/bin:$ANDROID_HOME/platform-tools;

# add Flutter binaries to $PATH
export FLUTTER_ROOT=$HOME/.local/opt/flutter;
export PATH=$FLUTTER_ROOT/bin:$PATH;' >> .profile
```

> Here Java, Android and Flutter tools are put a the **end** of the path. Be careful if you also have one of those tools installed on Windows and exported in the Windows `Path` as it may take precedence over your Unix version as Windows envs are exported to Linux.
> If so, you may need to put theme at the beginning of your path.

## 2. Setup on Windows

### 2.1 Platform tools

We need to install platform tools on Windows to install `adb` and "connect" android devices to Flutter on WSL.

You can download it [here](https://developer.android.com/studio/releases/platform-tools).
> It must be the **same** version on both WSL and windows

- unzip it and put it at the path of your choice.
- add the folder's path to your Windows `Path`: Search for `Advanced system parameters` then -> `Environment variables`. Create or append the platform-tools's folder to the user variable `Path`

> On windows, the `Path` separator is `;`

Then launch a command prompt and run `adb --version`:

```bash
C:\Users\XXX>adb --version
Android Debug Bridge version 1.0.41
Version 33.02-8557947
Installed as C:\Users\XXX\src\platform-tools\adb.exe
```

### 2.2 Android Studio (optional)

> This step is needed if you which to use an Android Emulator. You **dont** need it to use your own Android device.

- Download and install [Android Studio](https://developer.android.com/studio).
Once done, launch it and launch the device manager.

> You **don't** need to launch a project. On the startup window the device manager can be found in the triple dot menu on the right ;)

- Create and new device and launch it. Corresponding documentation can be found [here](https://developer.android.com/studio/run/managing-avds).

> Be careful on the Android version you install on the device. It must match one of the version installed on Linux with `sdkmanager` at step 1.
3.3.

## 3. Running a project

On Linux:

- Create or checkout a ClojureDart project: `clj -Tcljd init`
- Launch the ClojureDart compiler: `cljd -Tcljs watch`

On Windows:

- Launch and command line with Admin privileges
- run `adb devices` and check you physical or virtual device is listed
- run `adb kill-server && adb -aP 5037 nodaemon`
- allow network connections if prompted

Then on Linux:
- run `adb connect -H $(grep /etc/resolv.conf) -P 5037`
<!-- @todo fix grep -->
- run `adb devices` and you should see you device with it's corresponding id
- finally run: `flutter run -d DEVICE_ID`

**Et voila ! You should now have a running setup with hot reloading.** :)

If you use `VSCode` as IDE on Windows with remote connection to WSL, the Flutter extension should work out of the box as it use the $FLUTTER_HOME env variable exported in step 1.5. ;)