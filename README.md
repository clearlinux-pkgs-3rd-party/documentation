# Table of contents
1. [Adding the repo](#setup)
2. [Available Packages](#packages)
3. [Important information](#important)
4. [Fixes for some issues still existing with swupd 3rd-party](#fixes)
5. [Extra packages (manual installation)](#extras)
6. [Changelog](#changelog)

# Adding the repo<a name="setup"></a>

In order to use the repository, we first have to enable it with:

```
sudo swupd 3rd-party add greginator https://clear.greginator.xyz/
```

do not change the name greginator to anything else, it is the name that will be used for the repository on your own machine and it will mean that all content of the repository will be placed under /opt/3rd-party/bundles/greginator. This is needed for some .desktop files as they are hardcoded to this path

In order to list all available bundles from 3rd-party repos:

```
sudo swupd 3rd-party bundle-list -a
```

# Available Packages<a name="packages"></a>

* FFmpeg

  ```
  sudo swupd 3rd-party bundle-add ffmpeg
  ```

* Flameshot

  ```
  sudo swupd 3rd-party bundle-add flameshot
  ```

* Google Chrome Stable

  ```
  sudo swupd 3rd-party bundle-add google-chrome-stable
  ```

* Skype

  ```
  sudo swupd 3rd-party bundle-add skypeforlinux
  ```

* Teams

  ```
  sudo swupd 3rd-party bundle-add teams
  ```

* Visual Studio Code

  ```
  sudo swupd 3rd-party bundle-add code
  ```

* Zoom

  ```
  sudo swupd 3rd-party bundle-add zoom
  ```

* Transmission

  ```
  sudo swupd 3rd-party bundle-add transmission
  ```

# Important information<a name="important"></a>

## Updating from a version before 17-04-2020
Since I messed up my installation and had to remove the mixer folder I now have new private keys for the repository, this means that you have to remove and add the repository again. (Note that in the first command you should replace greginator with the name you gave the repository on your device).

```
sudo swupd 3rd-party remove greginator
sudo swupd 3rd-party add greginator https://clear.greginator.xyz/
```

Now reinstall all the software that you want, your config files will still work so you don't have to reconfigure things like VS Code.

## Fixes for some issues still existing with swupd 3rd-party:<a name="fixes"></a>

### .desktop files

Currently swupd 3rd-party doesn't export .desktop files, I have patched them manually to use the /opt/3rd-party path but you still have to copy them to your applications folder.

```
cp -R /opt/3rd-party/bundles/greginator/usr/share/applications/* $HOME/.local/share/applications
```


### VS Code filesystem watches

We need to allow VS Code to watch larger filesystems for changes, the default is too small

```
sudo mkdir -p /etc/sysctl.d
echo fs.inotify.max_user_watches=524288 | sudo tee /etc/sysctl.d/40-max-user-watches.conf && sudo sysctl --system
```

### Codecs

We need to add out codecs to the LD_LIBRARY_PATH, for this a few different things need to be done due to incompatibility between the configs for wayland, x11 and some quirck in firefox.

#### LD

```
sudo tee -a /etc/ld.so.conf << EOF
/opt/3rd-party/bundles/greginator/usr/lib64
/opt/3rd-party/bundles/greginator/usr/lib32
EOF
sudo ldconfig
```

#### x11

```
sudo tee -a /etc/profile << EOF
if [[ \$UID -ge 1000 && -d /opt/3rd-party/bundles/greginator/usr/lib64 && -z \$(echo \$LD_LIBRARY_PATH | grep -o /opt/3rd-party/bundles/greginator/usr/lib64) ]]
then
    export LD_LIBRARY_PATH="/opt/3rd-party/bundles/greginator/usr/lib64:/opt/3rd-party/bundles/greginator/usr/lib32:\${LD_LIBRARY_PATH}"
fi
EOF
```

#### Wayland

```
sudo mkdir -p /etc/environment.d/
echo LD_LIBRARY_PATH=/opt/3rd-party/bundles/greginator/usr/lib64:/opt/3rd-party/bundles/greginator/usr/lib32 | sudo tee /etc/environment.d/10-codecs.conf
```

#### Firefox

```
echo "export LD_LIBRARY_PATH=/opt/3rd-party/bundles/greginator/usr/lib64:/opt/3rd-party/bundles/greginator/usr/lib32" >> ${HOME}/.config/firefox.conf
```

# Extra packages (manual installation)<a name="extras"></a>

#### QTStyleplugins

The QTSyleplugins package provides some files for QT5 in order to be able to theme bettter, most importantly it includes the ability for QT to use the GTK2 theme making it possible to have a uniform look for QT applications in GNOME.

```
sudo swupd bundle-add wget os-clr-on-clr qt5ct
wget https://github.com/clearlinux-pkgs-3rd-party/qtstyleplugins/releases/download/v1.0/qtstyleplugins-lib-1-2.x86_64.rpm -O qtstyleplugins-lib-1-2.x86_64.rpm
rpm2cpio qtstyleplugins-lib-1-2.x86_64.rpm | (cd /; sudo cpio -i -d -u 2> /dev/null);
echo QT_QPA_PLATFORMTHEME=qt5ct | sudo tee /etc/environment.d/20-QT.conf
```

Now open qt5ct and set the style to gtk2, standard dialogs to GTK2, while you're at it you might want to change the icon theme and fonts as well.


# Changelog<a name="changelog"></a>

* 17-04-2020
  * Removed iio-sensor-proxy since it is available in the official repositories
  * Added transmission (gtk version)
  * Updated chrome, vscode, skype, teams and flameshot
  * QTStyleplugins is available for manual installation
