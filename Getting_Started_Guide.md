# DX-M1 AI Accelerator Development Kit for Raspberry Pi 5 — Getting Started Guide for AWS IoT Greengrass

This guide walks you through setting up AWS IoT Greengrass V2 on the DEEPX DX-M1 AI Accelerator Development Kit for Raspberry Pi 5 and verifying that the device communicates with AWS IoT services, from assembly to deploying your first Greengrass component.

## Document Information

### Applicable operating systems for this guide

| Role | Operating system |
|---|---|
| Device (DX-M1 Kit / Raspberry Pi 5) | Raspberry Pi OS (64-bit, Debian 13 "Trixie") |
| Host computer | Any OS with a web browser and an SSH client (Windows 10/11, macOS, or Linux) |

### Document revision history

| Revision | Date | Description |
|---|---|---|
| 1.0 | 2026-08-20 | Initial release |

## Overview

The DX-M1 AI Accelerator Development Kit pairs a Raspberry Pi 5 (8 GB) host with the DEEPX DX-M1 AI accelerator (M.2 module) on an M.2 base HAT, delivering up to 25 TOPS of neural inference at 1–5 W of accelerator power. It is intended for developers building and evaluating edge AI vision projects that combine local NPU-accelerated inference with cloud connectivity through AWS IoT Greengrass.

Running Raspberry Pi OS (64-bit, Debian 13 "Trixie", aarch64), the kit hosts the AWS IoT Greengrass Core software (classic nucleus), enabling local execution of Greengrass components, Docker application components, messaging, and secure communication with the AWS Cloud.

## Hardware Description

### Datasheet

- DEEPX DX-M1 product information: https://deepx.ai
- Kit product page (purchase): https://www.eleparts.co.kr/goods/view?no=18574428

### Standard kit contents

- 1x Raspberry Pi 5 (8 GB RAM)
- 1x DEEPX DX-M1 AI accelerator (M.2 2242/2280 module)
- 1x M.2 base HAT (PCIe carrier for the DX-M1 module)
- Assembly hardware
- Reference materials: DX-AllSuite guide (GitHub), assembly guide (PDF), Raspberry Pi 5 user guide (PDF)

### Key specifications

| Item | Specification |
|---|---|
| Host board | Raspberry Pi 5 (Broadcom BCM2712, quad-core Arm Cortex-A76 @ 2.4 GHz, aarch64) |
| AI Accelerator | DEEPX DX-M1 (M.2 module), 25 TOPS INT8, 4 GB LPDDR5, max 5 W |
| Memory | 8 GB LPDDR4X (host) |
| Storage | microSD (user-provided) or NVMe via M.2 (configuration-dependent) |
| Accelerator interface | PCIe Gen3 x1 via M.2 base HAT |
| Networking | 1x GbE LAN (RJ45) |
| USB | 2x USB 3.0, 2x USB 2.0 |
| Display | 2x micro-HDMI |
| OS | Raspberry Pi OS (64-bit, Debian 13 "Trixie") |

### User provided items

- microSD card (32 GB or larger) for Raspberry Pi OS, or an NVMe SSD
- Official Raspberry Pi 5 power supply (27 W USB-C PD recommended)
- Active cooling for the Raspberry Pi 5 (recommended)
- Ethernet cable for internet connectivity
- USB keyboard/mouse and a monitor with micro-HDMI for initial setup (or use SSH over the network)
- A host computer for remote access

### 3rd party purchasable items

None beyond the user-provided items listed above.

## Set Up Your Development Environment

No device-specific IDE is required to run AWS IoT Greengrass. You will need:

- The AWS Command Line Interface (AWS CLI), if you prefer command-line setup. See [Installing the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).
- (Optional) To develop AI-accelerated Greengrass components using the DX-M1 accelerator, install the DEEPX DX-RT runtime and DXNN® SDK. See the DEEPX Developer Portal: https://developer.deepx.ai

## Set Up Device Hardware

### Assemble the kit

1. Attach the M.2 base HAT to the Raspberry Pi 5 40-pin header and PCIe FPC connector, following the assembly guide included with the kit.
2. Install the DEEPX DX-M1 M.2 module into the base HAT's M.2 slot and secure it.
3. Attach active cooling to the Raspberry Pi 5.
4. Insert a microSD card (or NVMe SSD) prepared with Raspberry Pi OS (see next section).
5. Connect an Ethernet cable to the Raspberry Pi 5 LAN port.
6. (Initial setup) Connect a monitor via micro-HDMI and a USB keyboard, or plan to use SSH.
7. Connect the official 27 W USB-C power supply to boot.

### Flash Raspberry Pi OS

Using Raspberry Pi Imager on your host computer, flash **Raspberry Pi OS (64-bit)** to the microSD card or NVMe SSD. Create your user account during imaging or on first boot, and enable SSH if you plan to work remotely.

Verify internet connectivity on the device:

```bash
ping -c 3 amazon.com
```

### Enable PCIe Gen 3

The DX-M1 uses the Raspberry Pi 5 PCIe interface. Enable PCIe Gen 3 for full bandwidth:

```bash
sudo raspi-config
# Advanced Options -> PCIe Speed -> Yes (enable PCIe Gen 3) -> Finish -> reboot
```

After reboot, confirm the accelerator is detected on the PCIe bus:

```bash
lspci | grep -i deepx
# Expected: 0001:01:00.0 Processing accelerators: DEEPX Co., Ltd. DX_M1 (rev 01)
```

### Install the DX-M1 driver and runtime

Install the DEEPX NPU driver (DKMS) and the DX-RT runtime following the DX-AllSuite guide provided with the kit and the DEEPX Developer Portal (https://developer.deepx.ai). After installation, verify:

```bash
lsmod | grep -E 'dxrt|dx_'        # expect dxrt_driver and dx_dma loaded
ls /dev/dxrt0                     # expect the device node to exist
dxrt-cli --version                # expect DX-RT runtime version to print
```

> The DX-M1 driver and runtime are required to run AI inference on the accelerator. They are independent of AWS IoT Greengrass qualification, which validates the Greengrass Core software path; both are installed here so that AI-accelerated Greengrass components can use the NPU.

### Install Greengrass prerequisites

AWS IoT Greengrass Core software requires a Java Runtime Environment (Java 8 or greater):

```bash
sudo apt update
sudo apt install -y default-jre curl unzip
java -version
```

(Optional) To deploy Docker application components, install Docker Engine and confirm the classic image store:

```bash
docker --version
docker info | grep -i 'storage driver'   # expect: overlay2
```

## About AWS IoT Greengrass

To learn more about AWS IoT Greengrass, see [How AWS IoT Greengrass works](https://docs.aws.amazon.com/greengrass/v2/developerguide/how-it-works.html) and [What's new in AWS IoT Greengrass Version 2](https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-v2-whats-new.html).

## Greengrass Prerequisites

Follow the instructions in the following sections of [Tutorial: Getting started with AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/getting-started.html):

- [Step 1: Set up an AWS account](https://docs.aws.amazon.com/greengrass/v2/developerguide/getting-started-set-up-aws-account.html)
- [Step 2: Set up your environment](https://docs.aws.amazon.com/greengrass/v2/developerguide/getting-started-prerequisites.html)

## Install AWS IoT Greengrass

Follow the online guide to [Install the AWS IoT Greengrass Core software with automatic provisioning](https://docs.aws.amazon.com/greengrass/v2/developerguide/quick-installation.html). The steps below summarize the process on this device.

1. Provide AWS credentials to the device (development environments can use long-term IAM user credentials):

```bash
export AWS_ACCESS_KEY_ID=<the access key id for your user>
export AWS_SECRET_ACCESS_KEY=<the secret access key for your user>
```

2. Download the AWS IoT Greengrass Core software:

```bash
cd ~
curl -s https://d2s8p88vqu9w66.cloudfront.net/releases/greengrass-nucleus-latest.zip -o greengrass-nucleus-latest.zip
unzip greengrass-nucleus-latest.zip -d GreengrassInstaller
```

3. Install the AWS IoT Greengrass Core software (replace region, thing name, and group name as needed):

```bash
sudo -E java -Droot="/greengrass/v2" -Dlog.store=FILE \
  -jar ./GreengrassInstaller/lib/Greengrass.jar \
  --aws-region ap-northeast-2 \
  --thing-name DX-M1-RPI5-Core \
  --thing-group-name DXGreengrassGroup \
  --thing-policy-name GreengrassV2IoTThingPolicy \
  --tes-role-name GreengrassV2TokenExchangeRole \
  --tes-role-alias-name GreengrassCoreTokenExchangeRoleAlias \
  --component-default-user ggc_user:ggc_group \
  --provision true \
  --setup-system-service true
```

4. Verify that the Greengrass Core software is running:

```bash
sudo systemctl status greengrass.service
```

You can also verify in the AWS IoT console under **Greengrass devices > Core devices** that the device status is **Healthy**.

## Create a "Hello World" Component

### Create the component on your edge device

Follow [Develop and test a component on your device](https://docs.aws.amazon.com/greengrass/v2/developerguide/getting-started.html#create-first-component) to create a simple Hello World component locally and verify its output:

```bash
sudo tail -f /greengrass/v2/logs/com.example.HelloWorld.log
```

### Upload the "Hello World" component

Follow [Create your component in the AWS IoT Greengrass service](https://docs.aws.amazon.com/greengrass/v2/developerguide/getting-started.html#upload-first-component) to upload your component to the cloud.

### Deploy your component

Follow [Deploy your component](https://docs.aws.amazon.com/greengrass/v2/developerguide/getting-started.html#deploy-first-component) to deploy the component from the AWS IoT console and verify that it is running on the device.

## Debugging

- Greengrass Core logs: `/greengrass/v2/logs/greengrass.log`; component logs: `/greengrass/v2/logs/<component-name>.log`.
- To change the nucleus log level, see [Log levels](https://docs.aws.amazon.com/greengrass/v2/developerguide/monitor-logs.html).
- System service messages: `journalctl -u greengrass.service -f`.

## Troubleshooting

| Symptom | Resolution |
|---|---|
| Installer fails with `java: command not found` | Install Java: `sudo apt install -y default-jre`, then re-run the installer |
| `lspci` does not show the DX_M1 | Ensure PCIe Gen 3 is enabled in `raspi-config`, the M.2 module is fully seated, and the base HAT FPC cable is connected; reboot |
| `dxrt-cli` not found | Install the DX-M1 driver and DX-RT runtime per the DX-AllSuite guide; confirm `dxrt_driver` is loaded with `lsmod` |
| Device does not appear in AWS IoT console | Check internet connectivity and that outbound TCP 443 to AWS endpoints is allowed; verify the AWS region matches the console region |
| `greengrass.service` fails to start | Check `/greengrass/v2/logs/greengrass.log`; ensure the installation used `sudo` |
| Docker component deployment fails | Confirm Docker Engine is installed and `docker info` reports `overlay2` storage driver |

For more information, see [Troubleshooting AWS IoT Greengrass V2](https://docs.aws.amazon.com/greengrass/v2/developerguide/troubleshooting.html).

For device-specific support, contact DEEPX at tech_support@deepx.ai or visit https://deepx.ai/contact-us/technical-support/
