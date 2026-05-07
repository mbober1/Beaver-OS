# BeaverOS

## Supported targets
- Radxa Rock 5B+
- Radxa Zero 3E
- Luckfox Lyra Plus
- STM32MP157C-DK2 

### How to build
Clone repo
```
git clone https://github.com/mbober1/Beaver-OS.git
cd Beaver-OS
```

Reopen devcontainer and sync manifest
```
repo init -u https://github.com/mbober1/Beaver-OS.git -m default.xml
repo sync -j$(nproc)
```

Initialize env and build image
```
./setup-environment build
source sources/oe-core/oe-init-build-env build
bitbake lab-image-minimal
```

### Update throught OTA

Build update
```
bitbake lab-image-minimal-update
```

Go to `http://<target-ip>:8080/` and upload `lab-image-minimal-update.swu` file.
