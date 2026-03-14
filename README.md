# BeaverOS

### How to build
#### Bare-metal
Setup build
```
git clone https://github.com/mbober1/Beaver-OS.git
cd Beaver-OS
repo init -u https://github.com/mbober1/Beaver-OS.git -m default.xml
repo sync -j$(nproc)
./setup-environment build
```

Initialize env and build image
```
source sources/oe-core/oe-init-build-env build
bitbake lab-image-minimal
```

#### Docker
```

```

### Update throught OTA

Build update
```
bitbake lab-image-minimal-update
```

Go to `http://<radxa-ip>:8080/` and upload `lab-image-minimal-update-radxa-zero-3e-custom.rootfs.swu` file.
