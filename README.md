# Fedora Workstation Post-Install

Personal post-installation checklist for a fresh Fedora Workstation installation.

> **Note:** Commands in this document assume a standard Fedora Workstation installation using GNOME and `dnf5`. Some applications are intentionally installed through Flatpak, while development environments are isolated using Toolbox.

---

## 1. Update the system

Perform a complete system update before installing additional software.

```bash
sudo dnf upgrade
```

Reboot if the update includes a new kernel or other major system components:

```bash
systemctl reboot
```

---

## 2. Change the hostname

Set the machine hostname:

```bash
sudo hostnamectl set-hostname <new-name>
```

Verify:

```bash
hostnamectl
```

---

## 3. Review third-party repositories

Open **Settings → Software Repositories** and review the enabled repositories.

Disable repositories that are unnecessary for this installation.

For example:

* COPR repository for PyCharm (`phracek`)
* RPM Fusion Fedora Nonfree NVIDIA Driver repository, if NVIDIA drivers are not required
* RPM Fusion Fedora Nonfree Steam repository, if Steam will be installed through Flatpak

### Why?

Avoiding unnecessary third-party repositories reduces the number of external packages that can participate in dependency resolution and system updates.

> **Note:** Do not disable a repository if you actually depend on packages provided by it.

---

## 4. Enable RPM Fusion

RPM Fusion provides software that Fedora cannot distribute in its official repositories because of licensing, patent, or other legal restrictions.

Install both the Free and Nonfree repositories:

```bash
sudo dnf install \
  https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

Enable Fedora's Cisco OpenH264 repository:

```bash
sudo dnf config-manager setopt fedora-cisco-openh264.enabled=1
```

Verify the repositories:

```bash
dnf repolist
```

---

## 5. Multimedia codecs

Fedora intentionally ships with a restricted set of multimedia codecs. RPM Fusion can provide additional codec support.

Replace Fedora's `ffmpeg-free` package with the RPM Fusion version:

```bash
sudo dnf swap ffmpeg-free ffmpeg --allowerasing
```

Install the multimedia group:

```bash
sudo dnf install \
  @multimedia \
  --setopt="install_weak_deps=False" \
  --exclude=PackageKit-gstreamer-plugin
```

Update the multimedia group:

```bash
sudo dnf group upgrade multimedia
```

### Why?

This provides broader support for audio and video formats in applications such as VLC, Kdenlive, browsers, and other multimedia software.

RPM Fusion currently provides `ffmpeg` and other multimedia packages for Fedora 44.

---

## 6. Intel Media Driver

For Intel integrated graphics, install the Intel VA-API media driver:

```bash
sudo dnf install intel-media-driver
```

Check whether VA-API is working:

```bash
vainfo
```

If `vainfo` is not installed:

```bash
sudo dnf install libva-utils
```

### Why?

The Intel Media Driver provides hardware-accelerated video decoding/encoding for supported Intel GPUs.

---

# 7. Containers and development environments

## 7.1 Install Podman and Toolbox

Install Podman and Toolbox:

```bash
sudo dnf install podman toolbox
```

Check the installations:

```bash
podman --version
toolbox --version
```

### Why Toolbox?

Toolbox provides disposable development environments based on OCI containers while keeping the Fedora host relatively clean.

This is particularly useful for development tools and applications that have dependencies you don't want installed directly on the host.

---

## 7.2 Create a development Toolbox

Create a general-purpose development environment:

```bash
toolbox create -c development
```

Enter it:

```bash
toolbox enter development
```

Inside Toolbox, packages can be installed normally:

```bash
sudo dnf install <package>
```

Exit the container with:

```bash
exit
```

List existing Toolboxes:

```bash
toolbox list
```

---

# 8. Virtualization

Install Fedora's virtualization package group:

```bash
sudo dnf group install --with-optional virtualization
```

Verify that libvirt is running:

```bash
systemctl status libvirtd
```

Enable it if necessary:

```bash
sudo systemctl enable --now libvirtd
```

Add the current user to the appropriate virtualization group:

```bash
sudo usermod -aG libvirt $USER
```

Log out and back in after changing group membership.

### Why?

This provides the infrastructure required to run virtual machines using technologies such as KVM/QEMU and libvirt.

Virtual machines are useful for testing other Linux distributions without changing the host operating system.

---

# 9. Enable Flathub

Add Flathub as a Flatpak repository:

```bash
flatpak remote-add --if-not-exists \
  flathub \
  https://dl.flathub.org/repo/flathub.flatpakrepo
```

Verify:

```bash
flatpak remotes
```

### Why Flatpak?

Use Flatpak primarily for desktop applications where it provides a convenient, isolated, and up-to-date distribution channel.

The general rule for this setup is:

* **DNF/RPM** → system components, libraries, CLI tools, drivers
* **Flatpak** → desktop applications
* **Toolbox** → development environments and isolated CLI applications

---

# 10. Applications

## 10.1 Applications from Flathub

Applications I commonly install through Flathub:

* Brave
* Ente
* Epiphany (GNOME Web)
* Extension Manager
* Flatseal
* Gear Lever
* Kdenlive
* Krita
* LibreOffice
* Mission Center
* Okular
* OnlyOffice
* Podman Desktop
* qBittorrent
* Spotify
* Steam
* Thunderbird
* VLC
* Warehouse

Install applications through GNOME Software or directly with:

```bash
flatpak install flathub <application-id>
```

List installed Flatpaks:

```bash
flatpak list
```

Update them:

```bash
flatpak update
```

---

## 10.2 Applications from Fedora/RPM repositories

Install applications that are better suited to native RPM packages:

```bash
sudo dnf install \
  gnome-tweaks \
  gimp \
  inkscape \
  audacity \
  epiphany
```

### Google Chrome

Google Chrome is installed from Google's RPM repository/package rather than Fedora's repositories.

After downloading the RPM:

```bash
sudo dnf install ./google-chrome-*.rpm
```

Using `dnf install` for a local RPM is preferable to `rpm -ivh` because DNF can resolve dependencies.

---

# 11. GNOME Extensions

Install **Extension Manager** from Flathub and use it to manage GNOME Shell extensions.

Extensions I currently use:

* Alphabetical App Grid
* Blur my Shell
* Caffeine
* Clipboard Indicator
* Hot Edge
* Vitals

### Recommendation

Avoid installing large numbers of GNOME extensions.

GNOME Shell extensions can break after major GNOME upgrades, so keeping the list relatively small makes the desktop easier to maintain.

---

# 12. Git and GitHub

## 12.1 Configure Git

Set the global Git identity:

```bash
git config --global user.name "John Doe"
git config --global user.email "johndoe@email.com"
```

Review the configuration:

```bash
git config --global --list
```

---

## 12.2 Generate an SSH key

Generate an Ed25519 key:

```bash
ssh-keygen -t ed25519 -C "johndoe@email.com"
```

Accept the default location:

```text
~/.ssh/id_ed25519
```

> **Important:** The private key (`id_ed25519`) must never be uploaded or shared.

Display the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the output and add it to:

**GitHub → Settings → SSH and GPG keys → New SSH key**

Test the connection:

```bash
ssh -T git@github.com
```

A successful authentication should produce a message indicating that GitHub recognized the account.

---

# 13. LaTeX

Create an isolated Toolbox for LaTeX:

```bash
toolbox create -c latex-box
```

Enter it:

```bash
toolbox enter latex-box
```

Install the LaTeX distribution and required dependencies:

```bash
sudo dnf install \
  texlive-scheme-medium \
  qt6-qtwebengine \
  qt6-qt5compat
```

### Texmaker

Download Texmaker from:

https://www.xm1math.net/texmaker/

Install the downloaded RPM using DNF:

```bash
sudo dnf install ./<texmaker-package>.rpm
```

### TeXstudio

Alternatively:

```bash
sudo dnf install texstudio
```

---

# 14. Password Manager — Proton Pass

Create a dedicated Toolbox:

```bash
toolbox create -c proton-pass-box
```

Enter it:

```bash
toolbox enter proton-pass-box
```

Install the required dependency:

```bash
sudo dnf update
sudo dnf install -y webkit2gtk4.1
```

Download Proton Pass from:

https://proton.me/pass

Install the downloaded RPM:

```bash
sudo dnf install ./<proton-pass-package>.rpm
```

If Proton Pass needs to be launched from the Toolbox, create a desktop entry:

```bash
mkdir -p ~/.local/share/applications
```

Create the desktop file:

```bash
vi ~/.local/share/applications/proton-pass.desktop
```

Content:

```ini
[Desktop Entry]
Type=Application
Name=Proton Pass
Exec=toolbox run -c proton-pass-box proton-pass %f
Icon=/path/to/proton-pass-icon.svg
Terminal=false
Categories=Utility;
```

Update the desktop database:

```bash
update-desktop-database ~/.local/share/applications
```

> **Note:** Replace `/path/to/proton-pass-icon.svg` with the actual location of the icon.

---

# 15. Anki

Create a dedicated Toolbox:

```bash
toolbox create -c anki-box
```

Enter it:

```bash
toolbox enter anki-box
```

Install the dependencies required by Anki:

```bash
sudo dnf install -y \
  zstd \
  qt6-qtwayland \
  libXcomposite \
  libXcursor \
  libXi \
  libXtst \
  libXrandr \
  libxcrypt-compat \
  xdg-utils \
  libatomic \
  libxkbfile \
  mpv
```

```bash
sudo dnf install -y \
  google-noto-sans-cjk-fonts \
  google-noto-color-emoji-fonts
```

Create a desktop entry:

```bash
mkdir -p ~/.local/share/applications
```

```bash
vi ~/.local/share/applications/anki.desktop
```

Content:

```ini
[Desktop Entry]
Type=Application
Name=Anki
Exec=toolbox run -c anki-box anki %f
Icon=/path/to/anki-icon.svg
Terminal=false
Categories=Utility;
```

Update the desktop database:

```bash
update-desktop-database ~/.local/share/applications
```

---

## 16. Install Cockpit

Install Cockpit:

```bash
sudo dnf install cockpit
```

Enable the Cockpit socket:

```bash
sudo systemctl enable --now cockpit.socket
```

Verify:

```bash
systemctl status cockpit.socket
```

Cockpit is normally accessed through:

```text
https://localhost:9090
```

---

# 17. Useful system checks

After completing the installation, these commands are useful for verifying the system.

### Fedora version

```bash
cat /etc/fedora-release
```

### Kernel

```bash
uname -r
```

### Enabled repositories

```bash
dnf repolist
```

### Installed RPM packages

```bash
dnf list --installed
```

### Flatpak applications

```bash
flatpak list
```

### Toolbox environments

```bash
toolbox list
```

### Podman

```bash
podman info
```

### Virtualization

```bash
virsh list --all
```

---

# 18. Maintenance

## Update RPM packages

```bash
sudo dnf upgrade
```

## Update Flatpaks

```bash
flatpak update
```

## Update Toolbox containers

Enter the Toolbox and update it normally:

```bash
toolbox enter development
sudo dnf upgrade
```

Repeat for other Toolboxes when necessary.

---

# 19. Simple Btrfs Maintenance

Fedora Workstation uses Btrfs by default. **No regular manual Btrfs maintenance is required for normal operation.**

Do not routinely run `btrfs balance`, `btrfs check`, or filesystem defragmentation simply as a maintenance task.

The following commands are useful for occasional health checks.

## Check filesystem usage

Check how much space Btrfs has allocated and is currently using:

```bash
sudo btrfs filesystem usage /
```

For a more concise view:

```bash
sudo btrfs filesystem df /
```

This is especially useful when using Btrfs snapshots, since snapshots can gradually consume additional disk space.

---

## Check for filesystem errors

Check the Btrfs device error counters:

```bash
sudo btrfs device stats /
```

Ideally, the following counters should remain at `0`:

```text
read_io_errs
write_io_errs
flush_io_errs
corruption_errs
generation_errs
```

Do **not** routinely reset these counters. They are useful for detecting previous storage or filesystem problems.

---

## Check SSD TRIM

Fedora normally handles periodic TRIM automatically.

Check whether the systemd timer is active:

```bash
systemctl status fstrim.timer
```

If it is enabled and active, no manual TRIM is necessary.

---

## Periodic Btrfs scrub

A scrub verifies the checksums of Btrfs data and metadata and can repair corrupted data when a valid redundant copy is available.

A scrub does not need to be performed every week. For a typical workstation, running one **every month or every few months** is sufficient.

Start a scrub:

```bash
sudo btrfs scrub start -Bd /
```

Check the result:

```bash
sudo btrfs scrub status /
```

Pay particular attention to errors such as:

```text
Read errors
Checksum errors
Verify errors
Uncorrectable errors
```

A healthy scrub should report zero errors.

---

## Recommended routine

There is no need for a strict weekly Btrfs maintenance routine.

### Occasionally

```bash
# Check filesystem usage
sudo btrfs filesystem usage /

# Check device errors
sudo btrfs device stats /

# Check Snapper snapshots, if installed
sudo snapper list
```

### Every 1–3 months

```bash
# Verify filesystem integrity
sudo btrfs scrub start -Bd /
```

Then:

```bash
sudo btrfs scrub status /
```

### Avoid running routinely

```bash
btrfs balance
btrfs check
btrfs filesystem defragment
```

These commands have specific purposes and should be used when there is a reason to do so rather than as routine maintenance.

> **Optional automation:** If you don't want to remember to perform periodic scrubs, consider using `btrfsmaintenance` or Fedora's `btrfsd` to automate Btrfs maintenance.

---

# References

* [Fedora Quick Docs](https://docs.fedoraproject.org/en-US/quick-docs/)
* [NeuroFedora](https://docs.fedoraproject.org/en-US/neurofedora/overview/)
* [RPM Fusion](https://rpmfusion.org/)
* [Flathub](https://flathub.org/)
* [Toolbox documentation](https://docs.fedoraproject.org/en-US/fedora-silverblue/toolbox/)
