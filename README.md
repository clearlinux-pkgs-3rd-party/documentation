# Available Packages

* FFmpeg

* Flameshot

* Google Chrome Stable

* iio-sensor-proxy

* Skype

* Teams

* VSCode

* Zoom

# Installation

In order to use the repository, we first have to enable it with:

```

sudo swupd 3rd-party add <repo-name> https://clear.greginator.xyz/

```

where \<repo-name> can be anything, it is the name that will be used for the repository on your own machine and it will mean that all content of the repository will be placed under /opt/3rd-party/bundles/\<repo-name>

In order to list all available bundles from 3rd-party repos:

```

sudo swupd 3rd-party bundle-list -a

```

To update the 3rd-party repos:

```

sudo swupd 3rd-party update

```

In order to install the actual packages (note this will install all the packages, you can choose to only select a few)

```

sudo swupd 3rd-party bundle-add ffmpeg flameshot google-chrome-stable iio-sensor-proxy skype teams vscode zoom

```

## Some fixes:

Making the software available through a desktop environment:

```

cp -R /opt/3rd-party/bundles/<repo-name>/usr/share/applications $HOME/.local/share/applications

```

Make iio-sensor-proxy autostart

```

sudo cp -R /opt/3rd-party/bundles/<repo-name>/usr/lib/udev /etc

sudo cp -R /opt/3rd-party/bundles/<repo-name>/usr/lib/systemd /etc

```

Allow for large directory tracking in VS Code

```

sudo mkdir -p /etc/sysctl.d

echo fs.inotify.max_user_watches=524288 | sudo tee /etc/sysctl.d/40-max-user-watches.conf && sudo sysctl --system

```

Adding the codecs to the path (note this doesn't seem to be enough for firefox and vlc)

```

sudo sh -c 'echo /opt/3rd-party/bundles/<repo-name>/usr/lib >>/etc/ld.so.conf'

sudo ldconfig

```

To fix firefox and vlc (only for bash shell not for zsh, still working on a fix for that) add to /etc/environment

```

if [[ $UID -ge 1000 && -d /opt/3rd-party/bundles/<repo-name>/usr/lib && -z $(echo $LD_LIBRARY_PATH | grep -o /opt/3rd-party/bundles/<repo-name>/usr/lib) ]]

then

export LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:/opt/3rd-party/bundles/<repo-name>/usr/lib"

fi

```

The current workaround for zsh is a bit cumbersome but the following is needed for firefox:

```

echo "export LD_LIBRARY_PATH=/opt/3rd-party/bundles/<repo-name>/usr/lib" >> ${HOME}/.config/firefox.conf

```

and for vlc just run

```

LD_LIBRARY_PATH=/opt/3rd-party/bundles/<repo-name>/usr/lib vlc

```
