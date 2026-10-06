
                    Cypress Semiconductor Corporation
                   CyUSB Suite For Linux, version 1.0.5
                 Based on Ho-Ro's Qt5 update, with Qt6 support
                   ====================================

## Build and run locally

This fork supports Qt 5 and Qt 6. The fixed-size GUI uses logical-pixel fonts
to avoid the observed text clipping on Wayland with 150% display scaling.

On Ubuntu, install the Qt 6 build dependencies:

```bash
sudo apt install build-essential libusb-1.0-0-dev qt6-base-dev qt6-base-dev-tools qmake6 qt6-wayland
```

Clone the development branch and build from the repository root:

```bash
git clone --branch codex/qt6-gui https://github.com/electro-logic/cyusb_linux.git
cd cyusb_linux
make lib
make gui QMAKE=qmake6
./bin/cyusb
```

Normal launch enumerates and opens configured USB devices and can detach their
kernel drivers. Close UHD and other applications using the device first.
Do not use firmware programming or reset controls on hardware that is in use.
Local builds load the library from the checkout; no system-wide installation
or `sudo` launch is needed, provided your USB device permissions allow access.

For Qt 5, install `qtbase5-dev` and `qt5-qmake`, then use
`make gui QMAKE=qmake` instead. Re-run qmake when switching Qt versions.

## GUI tests without USB access

The smoke test constructs the GUI without enumerating or opening USB devices,
prints a result, and exits automatically after about 250 ms:

```bash
./bin/cyusb --gui-smoke-test
QT_QPA_PLATFORM=offscreen QT_SCALE_FACTOR=1.5 ./bin/cyusb --gui-smoke-test
./bin/cyusb --gui-smoke-test --gui-screenshot /tmp/cyusb-gui.png
```

The screenshot option also writes one image per main tab beside the requested
image. It requires `--gui-smoke-test`. Qt 5.15.18 and Qt 6.10.2 builds and GUI
smoke tests passed; the Qt 6 GUI was checked on native Wayland at 150% scaling.
USB transfers, firmware programming, and hotplug have not yet been validated
for this port. The GUI retains the upstream fixed layout rather than a
responsive layout.

## Device configuration

The library reads `~/.config/cyusb/cyusb.conf` when present, otherwise
`/etc/cyusb.conf`. These files select the USB vendor and product IDs to open;
the supplied configuration lists Cypress devices, not every board using FX3.
An SDR running its own firmware may therefore be absent from the device list.
Do not add its ID while another application is using it.

### Add a device to the per-user configuration

Close UHD and any other application using the device, then check its current
vendor and product IDs with `lsusb`. For example, a B210 running its firmware
can appear as `2500:0020`, rather than a Cypress bootloader ID.

From the repository root, create a per-user configuration if none exists:

```bash
mkdir -p ~/.config/cyusb
if [ ! -e ~/.config/cyusb/cyusb.conf ]; then
    cp configs/cyusb.conf ~/.config/cyusb/cyusb.conf
fi
nano ~/.config/cyusb/cyusb.conf
```

Add the matching hexadecimal IDs inside the existing `<VPD>` block, before
`</VPD>`, retaining the other entries. For the B210 example:

```text
2500    0020    USRP B210
```

Save the file and restart `./bin/cyusb` in normal mode, without
`--gui-smoke-test`. The per-user file replaces the global configuration for
that user; it does not modify `/etc/cyusb.conf`. If the list remains empty,
check the terminal output for access errors and verify that the current USB
ID still matches the entry. Device permissions remain a separate requirement.

Adding an ID only makes the device eligible to be opened. It does not make
its firmware compatible with Cypress example vendor commands. Avoid reset,
programming, and arbitrary transfers until the device protocol is verified.

## Legacy installation instructions

Pre-requisites:
---------------

 1. libusb-1.0.x is required for compilation and functioning of the API library.

 2. Native gcc/g++ tool-chain and the GNU make utility are required for
    compiling the library and application.

 3. Qt5 or Qt6 development packages are required for building the cyusb GUI application.

 4. If you want to build a Debian package you need also the packages
    'checkinstall' and 'fakeroot'.

 5. The pidof command is used by the cyusb_linux application to handle
    hot-plug of USB devices.


Installation Steps:
-------------------

 1. cd to the main directory where the files were extracted. 
    
 2. 'make lib' compiles the libcyusb library and creates a dynamic library.
    'make gui' compiles the cyusb_linux GUI application.
    'make all' combines both steps above.
    You can test the GUI application before installation: 'bin/cyusb'
    (To use the auto detection of newly installed USB devices you have to install the program.) 
    
 3. 'make install' installs the libcyusb library and the application program 
    in the system directories (/usr/local/lib and /usr/local/bin).
    It also sets up a set of UDEV rules in /etc/udev/rules.d 
    and updates the environment variables under the /etc/profile.d directory.
    It installs the global config file '/etc/cyusb.conf' (origin is directory 'configs').
    For a clean uninstall call 'make uninstall', this command removes the library,
    the application program and the configurations from the system directories.
    
    As these changes require root (super user) permissions,
    'make install' and 'make uninstall' need to be executed from a root login, e.g.:
    'sudo make install'

 4. 'make deb' creates lib and gui and creates a debian package that installs and uninstalls 
    cleanly under debian and ubuntu. The package building can be done as user. 
    You must be root to install the *.deb package, e.g.:
    'sudo apt install ./cyusb_1.0.5-1_amd64.deb'

 5. The GUI application can now be launched using the 'cyusb' command. 
    You should call it always from a command line to make use of the status messages.

 6. Programs using the library libcyusb require a valid config file, this is either
    the global one '/etc/cyusb.conf' or an individual file for each user,
    '~/.config/cyusb/cyusb.conf', which takes precedence over the global config.

EEPROM:
-------

If you only want to store an application file on the *large* EEPROM of the FX2, you can also use
the CLI tool [fx2eeprom](https://github.com/Ho-Ro/fx2eeprom) that requires less preparation effort.
