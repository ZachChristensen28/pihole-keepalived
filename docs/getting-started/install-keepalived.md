# Install Keepalived

!!! tip
    It is recommended to use **[snap](https://snapcraft.io/docs/installing-snapd)** to install and maintain keepalived.

## Ubuntu

Use snap to install the latest version of keepalived and apt to install libipset13.

```shell
sudo apt update
sudo apt install libipset13
sudo snap install keepalived --classic
```

<small>\*_see [https://snapcraft.io/install/keepalived/ubuntu](https://snapcraft.io/install/keepalived/ubuntu) for more information_</small>

## Debian

### Install using apt

```shell
sudo apt update
sudo apt install keepalived lilbipset13
```

-- OR --

### Install using snap

``` shell
sudo apt update
sudo apt install snapd libipset13
sudo reboot
```

``` shell
sudo snap install core
sudo snap install keepalived --classic
```

<small>\*_see [https://snapcraft.io/install/keepalived/debian](https://snapcraft.io/install/keepalived/debian) for more information_</small>

## Raspberry Pi OS

Use snap to install the latest version of keepalived and apt to install libipset13. Snapd may not be installed by default. If this is the case, a reboot may be required.

<small>\*_see [https://snapcraft.io/install/keepalived/raspbian](https://snapcraft.io/install/keepalived/raspbian) for more information_</small>

```shell
sudo apt update
sudo apt install snapd libipset13
sudo reboot
```

```shell
sudo snap install core
sudo snap install keepalived --classic
```

--8<-- "includes/abbreviations.md"
