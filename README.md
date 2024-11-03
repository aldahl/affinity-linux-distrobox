# HOWTO install Affinity Photo/Designer/Publisher under Linux in a distrobox environment

This has been used on Fedora40/nobara40 but it should work very similar on most other modern distributions.

Sources used:
- https://affinity.liz.pet/docs/1-intro.html
- https://forum.affinity.serif.com/index.php?/topic/182758-affinity-suite-v2-on-linux-wine/&do=findComment&comment=1279641
- https://commons.wikimedia.org/wiki/File:Affinity_Publisher_V2_icon.svg
- https://commons.wikimedia.org/wiki/File:Affinity_Designer_V2_icon.svg
- https://commons.wikimedia.org/wiki/File:Affinity_Photo_V2_icon.svg

IMPORTANT: Especially the branch name might evolve over time
which is used for the special Wine build, so check the forum and official docs above. This is more a quick cookbook for myself, as I was missing a complete guide that included distrobox and the icon/desktop file setup in one place.


## Create an arch distrobox (replace dnf with whatever your distro uses as a package manager)
```sh
# if you are on a RedHat family distro like Fedora or Nobara podman is usually the better solution as container runtime,
# for some other distros docker might be better, so check what is recommended for your distribution and what ships in their
# package repositories
sudo dnf install podman
sudo dnf install distrobox
distrobox create -n arch -i archlinux
distrobox enter arch
```

## Now from within the arch linux distrobox:
```sh
sudo pacman -Syu nano
sudo nano /etc/pacman.conf
```

#### under the other repository definitions add:
```ini
[multilib]
Include = /etc/pacman.d/mirrorlist
```

## Save and continue in the distrobox terminal:
```sh
# install all dependencies
sudo pacman -Syu alsa-lib alsa-plugins autoconf bison cups desktop-file-utils flex fontconfig freetype2 gcc-libs gettext gnutls gst-plugins-bad gst-plugins-base gst-plugins-base-libs gst-plugins-good gst-plugins-ugly libcups libgphoto2 libpcap libpulse libunwind libxcomposite libxcursor libxi libxinerama libxkbcommon libxrandr libxxf86vm mesa mesa-libgl mingw-w64-gcc opencl-headers opencl-icd-loader pcsclite perl samba sane sdl2 unixodbc v4l-utils vulkan-headers vulkan-icd-loader wayland wine-gecko wine-mono
sudo pacman -Syu git winetricks gcc pkgconfig make

# now setup rum and the special wine build
cd ~/Documents/
git clone https://gitlab.com/xkero/rum $HOME/Documents/rum
# make sure ~/.local/bin is in your users $PATH variable, some distros do that by default in the .bashrc, some don't
cp "$HOME/Documents/rum/rum" "$HOME/.local/bin/rum"
git clone https://gitlab.winehq.org/ElementalWarrior/wine.git ElementalWarrior-wine
cd ElementalWarrior-wine/
git switch affinity-photo3-wine9.13-part3
mkdir winewow64-build/ wine-install/
cd winewow64-build/
../configure --prefix="$HOME/Documents/ElementalWarrior-wine/wine-install" --enable-archs=i386,x86_64
make -j 16
make install -j16
sudo mkdir -p /opt/wines
sudo chown $USER:$USER /opt/wines
cp --recursive "$HOME/Documents/ElementalWarrior-wine/wine-install" "/opt/wines/affinity-photo3-wine9.13-part3"
ln -s /opt/wines/affinity-photo3-wine9.13-part3/bin/wine /opt/wines/affinity-photo3-wine9.13-part3/bin/wine64
rum affinity-photo3-wine9.13-part3 $HOME/.wineAffinity wineboot --init
rum affinity-photo3-wine9.13-part3 $HOME/.wineAffinity winetricks --unattended dotnet48 corefonts
rum affinity-photo3-wine9.13-part3 $HOME/.wineAffinity wine winecfg -v win11
rum affinity-photo3-wine9.13-part3 $HOME/.wineAffinity winetricks renderer=vulkan
# now set 144dpi in GUI in case of high resolution screen, otherwise you can skip this winecfg step
rum affinity-photo3-wine9.13-part3 $HOME/.wineAffinity wine winecfg

# ensure WinMetadata files are present
cd $HOME/.wineAffinity/drive_c/windows/system32
mkdir WinMetadata
cd WinMetadata/
# these Metadata files need to be extracted from a Windows .iso or real install from C:\Windows\system32\WinMetadata
# they are not part of this repo for legal reasons
unzip ~/Downloads/WinMetadata.zip 

# now run the installers, replace with whatever version you have downloaded (use the msi-exe installers)
rum affinity-photo3-wine9.13-part3 $HOME/.wineAffinity wine ~/Downloads/affinity-designer-msi-2.5.5.exe 
rum affinity-photo3-wine9.13-part3 $HOME/.wineAffinity wine ~/Downloads/affinity-photo-msi-2.5.5.exe 
rum affinity-photo3-wine9.13-part3 $HOME/.wineAffinity wine ~/Downloads/affinity-publisher-msi-2.5.5.exe
# now check that the apps are running properly when called via rum, also sign in to activate your license seat
rum affinity-photo3-wine9.13-part3 $HOME/.wineAffinity wine "$HOME/.wineAffinity/drive_c/Program Files/Affinity/Designer 2/Designer.exe"
rum affinity-photo3-wine9.13-part3 $HOME/.wineAffinity wine "$HOME/.wineAffinity/drive_c/Program Files/Affinity/Publisher 2/Publisher.exe"
rum affinity-photo3-wine9.13-part3 $HOME/.wineAffinity wine "$HOME/.wineAffinity/drive_c/Program Files/Affinity/Photo 2/Photo.exe"

# download icons
mkdir -p ~/.local/share/icons/
wget https://upload.wikimedia.org/wikipedia/commons/thumb/8/86/Affinity_Designer_V2_icon.svg/512px-Affinity_Designer_V2_icon.svg.png -O ~/.local/share/icons/Designer2.png
wget https://upload.wikimedia.org/wikipedia/commons/thumb/f/f5/Affinity_Photo_V2_icon.svg/512px-Affinity_Photo_V2_icon.svg.png -O ~/.local/share/icons/Photo2.png
wget https://upload.wikimedia.org/wikipedia/commons/thumb/9/9c/Affinity_Publisher_V2_icon.svg/512px-Affinity_Publisher_V2_icon.svg.png -O ~/.local/share/icons/Publisher2.png

# exit the distrobox container
exit
```

## Now back outside the distrobox in your host system:
```sh
mkdir -p ~/.local/share/applications/
# copy the desktop files from this repository (adapt the source path if needed)
cp ~/Downloads/Photo.desktop ~/.local/share/applications/Photo.desktop
cp ~/Downloads/Publisher.desktop ~/.local/share/applications/Publisher.desktop
cp ~/Downloads/Designer.desktop ~/.local/share/applications/Designer.desktop
# now refresh the Desktop app and icon cache
# under Plasma 5 run
kbuildsycoca5
# under Plasma 6 run
kbuildsycoca6
```
On other desktop managers check how to refresh or simply do a reboot, that should always work.

The apps should now show up normally in your start menu and you can pin them to the taskbar, desktop etc. .

## Later: To update the apps
```sh
# download the msi-exe versions from https://store.serif.com/de/account/licences/
# enter the box environment
distrobox enter arch
# run the installers like on first install, just with the new version
rum affinity-photo3-wine9.13-part3 $HOME/.wineAffinity wine ~/Downloads/affinity-designer-msi-2.5.5.exe 
rum affinity-photo3-wine9.13-part3 $HOME/.wineAffinity wine ~/Downloads/affinity-photo-msi-2.5.5.exe 
rum affinity-photo3-wine9.13-part3 $HOME/.wineAffinity wine ~/Downloads/affinity-publisher-msi-2.5.5.exe
```