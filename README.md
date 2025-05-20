# Rooting-Bigme-B751C


Rooting Bigme B751C


A work in progress guide documenting the process of successfully rooting the Bigme B751C e-reader device, including all necessary steps, tools, and cautions.


## Table of Contents

- [Important Warning](#important-warning)
- [Introduction](#introduction)
- [Required Tools](#required-tools)
- [Important Preparation](#important-preparation)
- [Setting Up Your Device](#setting-up-your-device)
- [Creating a Backup](#creating-a-backup)
- [Patching Files](#patching-files)
    - [Patching the Boot Image](#patching-the-boot-image)
    - [Patching the Vbmeta](#patching-the-vbmeta)
- [Rooting Process](#rooting-process)
- [Recovery Option](#recovery-option)


## Important Warning

> [!WARNING]
> **PLEASE READ BEFORE PROCEEDING**
>
> By following this guide, you acknowledge and accept the following risks and conditions:
>
> - **WARRANTY VOID**: Rooting your device will likely void your manufacturer's warranty.
> - **DATA LOSS**: There is always a risk of complete data loss. BACKUP ALL IMPORTANT DATA before proceeding.
> - **BRICKING**: Following these steps incorrectly may permanently damage ("brick") your device.
> - **SECURITY RISKS**: Rooted devices are more vulnerable to malware and security threats.
> - **OTA UPDATES**: You may lose the ability to receive over-the-air updates from your manufacturer.
>
> I assume NO LIABILITY for any damage to your device, loss of data, or any other negative consequences that may result from following this guide. You are proceeding entirely AT YOUR OWN RISK.
>
> This guide is specific to the Bigme B751C model. While it might be somewhat similar for other Bigme devices, proceed at your own risk.
>
> If you are not comfortable with these risks or lack technical experience, I strongly recommend NOT proceeding with rooting your device.

## Introduction

This is not exactly a guide but rather a documentation of how I successfully rooted this device. I am not an Android developer. Please note that you will need to wipe your device in order to gain root access.


## Required Tools

| Tool | Purpose | Repository |
|------|---------|------------|
| [MTK Client](https://github.com/bkerler/mtkclient) | Backup & Flashing | <https://github.com/bkerler/mtkclient> |
| [vbmeta-disable-verification](https://github.com/WessellUrdata/vbmeta-disable-verification) | Vbmeta Patching | <https://github.com/WessellUrdata/vbmeta-disable-verification> |
| [Magisk](https://github.com/topjohnwu/Magisk) | Root Solution | <https://github.com/topjohnwu/Magisk> |


## Important Preparation

**BEFORE PROCEEDING, ALWAYS MAKE A FULL BACKUP OF THE DEVICE**

Not having a backup will cause problems as Bigme does not provide a way to unbrick the device (as far as I know) like a fastboot ROM.

I performed this process on system version B751CE_R_6.3.0. It should work with newer software versions as well.

## Setting Up Your Device

1. **Enable USB debugging**:
    - From the default launcher find: App Management → Application Information → Settings → Open
    - Go to About Tablet and tap Build Number several times until you see "You are now a developer!"
    - Go back to System → Advanced → Developer Options
    - Enable USB Debugging and OEM Unlocking

> **Note**: Bigme might have disabled ADB and fastboot functionality. Also, the device doesn't have a recovery mode, which makes things more challenging.

## Creating a Backup

Open a terminal in the folder where you have MTK Client installed:

```bash
python mtk.py rl --skip userdata c:/projects/bigme
```

You can change `c:/projects/bigme` to a folder of your choice. Make sure you have enough free space (approximately 10GB).

To perform the backup:

1. Power off the B751C
2. Hold both page buttons while connecting the device to your PC with a USB cable
3. Don't release the keys until the device is recognized in the terminal
4. The backup process will start and take some time

After completion, verify there are no errors and the files have proper sizes (files with 0KB are suspicious). If unsure, repeat the process. This is the most critical part of the procedure.

## Patching Files

1. Copy `boot_a.bin` and `vbmeta_a.bin` to another folder
2. Rename them to `original_boot_a.bin` and `original_vbmeta_a.bin` to keep the originals intact

### Patching the Boot Image

I had patched the `boot_a.bin` with Magisk on my B751C and the size was considerably smaller than the original, so I used my phone but still failed. I ended up using this site: [MagiskPatcher](https://circlecashteam.github.io/MagiskPatcher/)

Source: [GitHub - affggh/Magisk_patcher](https://github.com/affggh/Magisk_patcher)

I used a debug version of Magisk since I thought there was a problem patching this device's boot bin file. In the end, the Magisk patcher website and the latest debug version of Magisk worked fine.

**How to use the site:**

1. Download the latest Magisk from: [GitHub - topjohnwu/Magisk/releases](https://github.com/topjohnwu/Magisk/releases) or let the website download and load the latest version
2. Upload your `original_boot_a.bin`
3. Set the following options:
    - Keep Verity: **OFF**
    - Keep Force Encrypt: **ON**
    - Recovery Mode: **OFF**
    - Patch Vbmeta Flag: **ON**
    - Legacy SAR Device: **OFF**

> **Note**: I am not 100% sure if I used these exact settings, but it should work. If your device gets stuck in a bootloop, try setting Keep Force Encrypt to OFF and try again.

4. Patch and look for the success messages:

```
Success: Worker terminated.
Success: Download starting...
```

5. Rename the file to `patched_boot.img` and move it to a folder of your choice

### Patching the Vbmeta

1. Place `patch-vbmeta.py` from the vbmeta-disable-verification repository in the same folder as your `original_vbmeta_a.bin`
2. Run the command:

```bash
python patch-vbmeta.py original_vbmeta_a.bin
```

3. Rename the patched file to `patched_vbmeta.bin`

## Rooting Process

> [!WARNING]
> **THIS STEP WILL ERASE YOUR DEVICE!**

1. **Wipe the device**:

```bash
python mtk.py e metadata,userdata,md_udc
```

Always make sure there are no errors after the commands
2. **OEM Unlock command**:

```bash
python mtk.py da seccfg unlock
```

3. **Bootloader unlock**:

```bash
python mtk.py da vbmeta 3
```

4. **Flash the patched boot images**:

```bash
python mtk.py w boot_a patched_boot.img
python mtk.py w boot_b patched_boot.img
```

5. **Flash patched vbmeta**:

```bash
python mtk.py w vbmeta_a patched_vbmeta.bin
python mtk.py w vbmeta_b patched_vbmeta.bin
```

6. **Finally, use this command to reboot**:

```bash
python mtk.py reset
```


Your device will boot as if it's the first time. Complete the initial setup process, install the Magisk APK, and verify root access works with a root checker app.

## Recovery Option

If something goes wrong, you can restore your backup with commands like:

```bash
python mtk.py w boot_a c:/projects/bigme/boot_a.bin
```


(Repeat for other partitions as needed)

---

**Note**: This guide is provided for educational purposes only. Always proceed with caution when modifying system files on your device.

<div style="text-align: center">⁂</div>

[^1_1]: https://github.com/abhisheknaiidu/awesome-github-profile-readme

[^1_2]: https://www.youtube.com/watch?v=qunj4_cNbt8

[^1_3]: https://www.hatica.io/blog/best-practices-for-github-readme/

[^1_4]: https://help.corsizio.com/article/65-markdown-formatting

[^1_5]: https://grrr.tech/posts/2024/format-a-github-readme/

[^1_6]: https://docs.github.com/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax

[^1_7]: https://www.freecodecamp.org/news/how-to-create-notice-blocks-in-markdown/

[^1_8]: https://www.codecademy.com/resources/docs/markdown/code-blocks

[^1_9]: https://docs.github.com/articles/about-readmes

[^1_10]: https://www.youtube.com/watch?v=rCt9DatF63I

[^1_11]: https://github.com/matiassingers/awesome-readme

[^1_12]: https://github.blog/changelog/2021-04-13-table-of-contents-support-in-markdown-files/

[^1_13]: https://www.reddit.com/r/github/comments/uulygm/what_are_some_really_nice_github_profile_readmes/

[^1_14]: https://stackoverflow.com/questions/18244417/how-do-i-create-some-kind-of-table-of-content-in-github-wiki

[^1_15]: https://github.com/othneildrew/Best-README-Template

[^1_16]: https://www.setcorrect.com/portfolio/work11/

[^1_17]: https://www.docstomarkdown.pro/create-a-table-of-contents-in-markdown/

[^1_18]: https://dev.to/github/10-standout-github-profile-readmes-h2o

[^1_19]: https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/organizing-information-with-tables

