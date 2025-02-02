# Setting Up and Using TorizonCore Builder

## 1. Install WSL2 or Access a Linux Environment
- **Windows users:** Install **WSL2** (Windows Subsystem for Linux).
- **Linux users:** Open a bash shell (e.g., on Ubuntu, Debian, or any Linux-based OS of your choice).

## 2. Create a Working Directory
Create a directory for your TorizonCore Builder files. For example:

```bash
mkdir -p ~/tcbdir/ && cd ~/tcbdir/
```

## 3. Download TorizonCore Builder Setup Script
Download the setup script from the GitHub repository:

```bash
wget https://raw.githubusercontent.com/toradex/tcb-env-setup/master/tcb-env-setup.sh 
```

## 4. Set Up TorizonCore Builder
Navigate to the `tcbdir` directory and source the setup script:

```bash
source tcb-env-setup.sh
```

**Note:** This command must be run each time a new bash session is started to make the TorizonCore Builder available in the terminal.

To confirm the tool is set up correctly, check the available commands:

```bash
torizoncore-builder --help
```

## Building with TorizonCore Builder

### 1. Create a Configuration File
Create a template configuration file using:

```bash
torizoncore-builder build --create-template
```

This generates a `tcbuild.yaml` file. Edit this file to customize your OS image.

#### Key Sections in `tcbuild.yaml`:
- **Input:** Specifies the source image.
- **Customization:** Defines changes (e.g., device tree, splash screen).
- **Output:** Specifies the customized OS image.

### 2. Specify the Base Image
Download a Toradex Easy Installer image for your device. For example, to download Torizon 7 OS for Verdin AM62 (without pre-provisioned containers):
[Download Torizon 7 OS for Verdin AM62](https://tezi.toradex.com/artifactory/torizoncore-oe-prod-frankfurt/scarthgap-7.x.y/release/4/verdin-am62/torizon/torizon-docker/oedeploy/torizon-docker-verdin-am62-Tezi_7.1.0+build.4.tar)


- Create an `images` directory:
  
  ```bash
  mkdir images
  ```
  
- Move the downloaded image to this folder.
- Update the `tcbuild.yaml` file under the **input** section:
  
  ```yaml
  input:
    easy-installer:
      local: images/torizon-docker-verdin-am62-Tezi_7.1.0+build.4.tar
  ```

### 3. Configure Output Directory
Specify the output directory in `tcbuild.yaml` for the customized image:

```yaml
output:
  easy-installer:
    local: output_directory/torizon-docker-verdin-am62-Tezi_7.1.0.CUSTOM
```

### 4. Add a Custom Splash Screen
Add a custom splash screen in the **customization** section:

```yaml
customization:
  splash-screen: freaked_out_cat.png
```

### 5. Edit the Device Tree
To modify the device tree:

1. Unpack the base image:
   ```bash
   torizoncore-builder images unpack images/torizon-docker-verdin-am62-Tezi_7.1.0+build.4.tar
   ```
2. Clone necessary repositories:
   ```bash
   git clone -b toradex_ti-linux-6.6.y git://git.toradex.com/linux-toradex.git linux
   git clone -b toradex_ti-linux-6.6.y git://git.toradex.com/device-tree-overlays.git device-trees
   ```
3. Find compatible device trees:
   ```bash
   find linux -name "*am62*-verdin*.dts"
   ```
4. Add overlays using:
   ```bash
   torizoncore-builder dto list --device-tree ./linux/arch/arm/boot/dts/ti/<device-tree>
   ```
5. Update `tcbuild.yaml`:
   
   ```yaml
   customization:
     device-tree:
       include-dirs:
         - linux/include
       custom: linux/arch/arm/boot/dts/ti/<device-tree>
       overlays:
         add:
           - device-trees/overlays/<overlay>
   ```

### 6. Add Kernel Modules
To add kernel modules (e.g., a Hello World module):

1. Clone the repository:
   ```bash
   git clone https://github.com/toradex/hello-mod
   ```
2. Update `tcbuild.yaml`:
   
   ```yaml
   customization:
     kernel:
       modules:
         - source-dir: hello-mod/
           autoload: no
   ```

### 7. Update Kernel Parameters
Update kernel parameters in `tcbuild.yaml`:

```yaml
customization:
  kernel:
    arguments:
      - key1=val1
      - key2=val2
```

### 8. Capture Target Device Changes
To capture changes made directly on the target device:

1. Use the `isolate` command:
   ```bash
   torizoncore-builder isolate --remote-host <IP> --remote-username torizon --remote-password torizon --changes-directory changes1
   ```
2. Add the changes to `tcbuild.yaml`:
   
   ```yaml
   customization:
     filesystem:
       - changes1/
   ```

### 9. Pre-Provision Application Containers
Add application containers to the custom image:

- Directly specify the Docker Compose file:
  
  ```yaml
  output:
    easy-installer:
      bundle:
        compose-file: custom/docker-compose.yml
  ```

- Alternatively, create a tarball from the Compose file:
  
  ```bash
  torizoncore-builder bundle <compose-file-path> --bundle-directory bundle --platform linux/arm64
  ```
  
  Update `tcbuild.yaml`:
  
  ```yaml
  output:
    easy-installer:
      bundle:
        dir: bundle/
  ```

### 10. Build and Deploy the Custom OS

1. Build the OS:
   ```bash
   torizoncore-builder build
   ```
2. Deploy to the board:
   ```bash
   torizoncore-builder images unpack <output-image>
   torizoncore-builder deploy --remote-host <IP> --remote-username torizon --remote-password torizon --reboot
   ```

