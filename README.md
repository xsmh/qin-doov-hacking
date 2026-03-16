# Qin/Doov Hacking
Documentation for hacking Qin (Duoqin) / Doov brand phones.

# Support Me
If this saved you time and effort, I’d appreciate your support on Ko-fi.

<a href="https://ko-fi.com/sheep1">
  <img src="https://cdn.prod.website-files.com/5c14e387dab576fe667689cf/670f5a0171bfb928b21a7e00_support_me_on_kofi_beige.png" alt="Ko-fi" width="400">
</a>

# ToC
- [Overview](#overview)
   * [Tested devices ](#tested-devices)
- [Device button combinations ](#device-button-combinations)
- [Warning  ⚠️ ⚠️ ⚠️](#warning--%EF%B8%8F-%EF%B8%8F-%EF%B8%8F)
- [Prerequisites](#prerequisites)
- [Install the flashing tools ](#install-the-flashing-tools)
   * [Create bootable USB stick](#create-bootable-usb-stick)
   * [Boot from USB stick ](#boot-from-usb-stick)
   * [General info about the Linux ISO](#general-info-about-the-linux-iso)
- [Make a backup](#make-a-backup)
    * [Option 1 (Recommended): store backup on 2nd USB stick](#option-1-recommended-store-backup-on-2nd-usb-stick)
    * [Option 2: store backup on the live Linux environment](#option-2-store-backup-on-the-live-linux-environment)
- [Unlock the bootloader](#unlock-the-bootloader)
   * [For most models](#for-most-models)
   * [For F21 Pro and similar models where “press volume up” doesn’t work](#for-f21-pro-and-similar-models-where-press-volume-up-doesnt-work)
- [Flash new ROM](#flash-new-rom)
- [Enter fastboot](#enter-fastboot)
- [Recover from backup](#recover-from-backup)
    * [Recover stock ROM without affecting American bands or other partitions](#recover-stock-rom-without-affecting-american-bands-or-other-partitions)
    * [Full recovery](#full-recovery)
- [Remove TWRP from F21 Pro](#remove-twrp-from-f21-pro)
   * [Solution](#solution)
- [Flash American bands on F21 Pro](#flash-american-bands-on-f21-pro)
   * [Flash](#flash)
- [Common errors](#common-errors)
   * [Error: write_sparse_skip_chunk: don't care size XXXXXXXXX is not a multiple of the block size XXXX](#error-write_sparse_skip_chunk-dont-care-size-xxxxxxxxx-is-not-a-multiple-of-the-block-size-xxxx)
   * [FAILED (remote: 'This partition doesn't exist')](#failed-remote-this-partition-doesnt-exist)
   * [FAILED (remote: 'Not enough space to resize partition')](#failed-remote-not-enough-space-to-resize-partition)
      + [Solution](#solution-1)
         - [Option 1: Delete product partition (Experimental)](#option-1-delete-product-partition-experimental)
         - [Option 2: Delete COW partitions](#option-2-delete-cow-partitions)
            * [For `cow` partitions that are in slot `a` (.e.g `system_a-cow`)](#for-cow-partitions-that-are-in-slot-a-eg-system_a-cow)
            * [For `cow` partitions that are in slot `b` (.e.g `system_b-cow`)](#for-cow-partitions-that-are-in-slot-b-eg-system_b-cow)
            * [Finally](#finally)
   * [Dm-verity corruption](#dm-verity-corruption)
      + [Solution](#solution-2)
   * [Orange state warning ](#orange-state-warning)
   * [Preloader - \[LIB\]: Status: Handshake failed](#preloader---lib-status-handshake-failed)
- [Special Thanks](#special-thanks)


# Overview
This documentation has been written to walk the owners of Qin/Doov devices through flashing DumberOS (formerly Dumbdroid) or any other compatible ROM of their choice.

The guide assumes that you are using Windows 10/11. If you are using Linux, I trust your ability to figure out the OS specific parts on your own. If you are using macOS, good luck.

If you encounter any problems while following this guide, refer to the [common errors](#common-errors) section. If your issue isn’t listed there, please [open a new issue](https://github.com/xsmh/duoqin-doov-hacking/issues/new) in this repository and include a description of the problem along with the relevant logs.

## Tested devices 
The guide has been tested on the following devices but it should work on most other models as well:

**Qin:**
- F21 Pro
- F22 Pro
- F22[^F22] (see footnote for DumberOS)

**Doov:**
- R77 Pro (R77c)
- R77
- R17 (Z17) Pro (3.5 inch screen)

# Device button combinations 
We won't be using any of the button combos in this guide, but they are useful to know sometimes.  

- For BROM mode you will need to run mtkclient first, and then hold the button combo and plug in the cable while still holding the buttons.  
- Recovery mode is built into the phone and no cable or program is required.  

| Model | BROM (bootrom) | Recovery |
| :---  | :--- | :--- |
| **F21 Pro** | `menu` + `back` (top two buttons) | `menu` + `power` + `*`, wait until android logo appears and then hold `power` + `up` |
| **F22 Pro** | `menu` + `back` (top two buttons) | - |
| **F22** | `menu` + `back` (top two buttons) | `power` + `up`|
| **R77 Pro** | `call` + `power`| `#` + `power` for 10 seconds (disabled on some units)|
| **R77** | `call` + `power`| - |
| **R17 Pro** | `call` + `power`| - |

# Warning  ⚠️ ⚠️ ⚠️
- Do **NOT** use AI chatbots for this unless you want a bricked device.
- Do **NOT** skip making a backup. I cannot help you if you brick your device and do not have a backup.
- Do **NOT** delete or flash the preloader. Recovering from a broken preloader is extremely difficult, if not impossible. Especially without a backup.

Flashing your device can **brick your phone** if done incorrectly.  
By following this guide, you **agree to proceed at your own risk**. I'm **not responsible** for any damage, data loss, or other issues that may occur.  

# Prerequisites
1. A Qin or Doov brand phone.
2. A computer with at least 8GB[^RAM] of RAM and three USB-A ports[^USB] for running the flashing tools.[^Apple] (see footnote for Apple)
3. Two USB flash drives.[^Drive] Each having a capacity of 12GB or more.[^Capacity] Alternatively, you could use only one flash drive if you have +16GB of RAM, check the note in [Make a backup](#make-a-backup) to learn more.
4. A data transfer USB A-C cable. Strictly A-C, **not** C-C. Make sure the cable you use is capable of transferring data, not just power.[^Cable] No adapters should be used with this cable, it needs to be connected directly to a USB-A port.


# Install the flashing tools 
This has been by far the most difficult part of the process for most users.
To simplify it I have created a customized Linux ISO that comes with the tools you need pre-installed. The OS you are using on your machine is irrelevant as it will not be affected.  
The Linux ISO does not include SN Writer, which you will only need if you are flashing the American bands. You will have to use Windows for that part if you need it.

## Create bootable USB stick
1. Download the [Linux ISO](https://drive.google.com/file/d/1Et7JjyKfpadQd9hh9fi7D-ECnHhC0tQT) that comes pre-installed with the tools.
2. Download and install [Rufus](https://rufus.ie/en/#download).
3. Launch Rufus and insert a USB stick. Your USB drive should show up in the `Device` field.
4. Click `SELECT`. Choose the Linux ISO and click `Open` to confirm.
5. If you’ve already tried Rufus and the USB stick failed to boot on your system, change `Partitioning scheme` to `GPT` and check that `Target system` is set to `UEFI (non CSM)`. This works on modern systems that disable legacy compatibility.
6. Click `START`. Click `OK` or `Yes` on any prompts and popups.
7. The image will now be written to the drive. Once it is done, the status bar will say `READY`. You can then click `CLOSE` to finish the process.

## Boot from USB stick 
There are two ways you could go about this.

**Option 1:** Connect your USB stick if it is not already. Hold the `Shift` key while pressing the `Restart` button and wait until Windows prompts you to choose an option. Click `Use a device` and then click `Removable Device`. Windows should now reboot into the USB drive.

**Option 2:** Connect your USB stick if it is not already. Reboot your computer and go into the BIOS. Disable `Secure Boot` and change the boot order to make the USB drive the first option. Save and reboot. These are some general instructions. This method will depend on your computer model, so you will have to look it up if you don't know how to do it. 

**Finally:** When the computer reboots, you will be greeted with a few options. Press enter on the first option `Start Linux Mint` (If you have already done that before and ran into issues, then try picking `Compatibility Mode` instead). Once you have booted into Linux, you will be shown a login screen. Insert the password `user` and hit enter.

## General info about the Linux ISO

- Password for logging in to the live Linux environment is `user`.
- There is no persistence. Meaning that any data you store on the Linux ISO itself will be lost after a reboot.
- Wi-Fi may not work on some computer models due to unavailable proprietary drivers. In which case you will have to either use an Ethernet cable or transfer data via an external drive.
- Includes empty vbmeta file, American bands partitions, python script to force fastboot mode, and an `F21 Pro` boot image without TWRP installed.
- There are 4 pre-installed programs that you can run with the following commands from the terminal: 
 1. `adb`
 2. `fastboot`
 3. `ghex`
 4. `mtk` for CLI mode of mtkclient & `mtk_gui` for the graphical interface


 To open the terminal, simply click the black square icon in the task bar at the bottom.  
 Going forward, whenever I mention **Run**, it means type the command that follows in the terminal and press enter.  
 You should always run commands from home directory. If you are having an issue with a command, you should run `cd` first to go back to the home directory if you aren't in it already.  
  

# Make a backup

> [!IMPORTANT] 
> This is the most important step in the guide. It is crucial that you do not skip it.  
> Do note that this will only backup the firmware, it will not backup personal user data if you have any stored on your device.

> [!NOTE]
If your computer has +16GB of RAM, you could skip using the 2nd drive and store the backup directly on the Linux image and upload it to a cloud storage service (like Google Drive) once it's done (keep in mind the Linux ISO would lose all data after a reboot). I do not recommend this method as it uses RAM as storage and the live image can crash if you run out of it. But it should be safe if you have +32GB RAM. Follow **Option 2** if you want to go this route.

> [!CAUTION]
> If you have more than one device and you have already made a backup for one, you should change `stock_rom` in the commands with a different folder name (e.g. `stock_rom2`) so that you do not overwrite the already existing backup.

## Option 1 (Recommended): store backup on 2nd USB stick

1. While booted into the live Linux image, connect your 2nd USB stick and wait for a notification in the top right corner of the screen that says `Volume mounted`. This 2nd USB stick should previously be formatted to exFAT (**not** FAT32), we will use this one for storing the backup. Do **not** unplug the 1st USB stick that has the Linux image on it.
2. Open the terminal in the Linux ISO by clicking the black square icon in the taskbar.
3. Type `lsblk` and hit enter. Under `MOUNTPOINTS` you will see an entry similar to  
`/media/user/exampleName`. In your case `exampleName` will be whatever your USB drive name is. Take note of this path as we will use it in the next step.  **Note:** Sometimes you might see more than one mountpoint that looks similar but with a different name like `/media/user/differentName`, we want the one that has the flash drive's name and not something else like your computer's internal drive.
4. Run `mkdir "/media/user/exampleName/stock_rom"` but replace `exampleName` in the path with whatever your drive name was from the previous step. This command creates the folder we will be using to store our backup in.
5. To make the backup, run `mtk rl --skip userdata "/media/user/exampleName/stock_rom"` but don't forget to replace `exampleName`. Connect the cable to your phone while it is **turned off** and wait for the command to finish running. This will take roughly 10 minutes and will show this message once it is done `DaHandler - All Dumped partitions success`. If the command ran into any errors at any point, you probably don't have enough storage on your 2nd USB drive (possibly due to it being formatted as FAT32) and you should not proceed until you resolve the issue, even if you see the success message at the end. You can double check to see if the files were actually made inside the `stock_rom` folder of the USB drive using the file explorer, but keep in mind that this does not mean they were made correctly if you did run into any errors.
6. To backup the preloader, run `mtk r preloader "/media/user/exampleName/stock_rom/preloader.bin" --parttype=boot1`. Don't forget to replace `exampleName` here too. After this has finished, you should now be able to see a bunch of files with .bin extension inside the stock_rom folder of your USB drive.

## Option 2: store backup on the live Linux environment

1. Open the terminal in the Linux ISO by clicking the black square icon in the taskbar.
2. Run `mkdir stock_rom` to create the folder we will be using to store our backup in.
3. To make the backup, run `mtk rl --skip userdata stock_rom`. Connect the cable to your phone while it is **turned off** and wait for the command to finish running. This will take roughly 10 minutes and will show this message once it is done `DaHandler - All Dumped partitions success`. If the command ran into any errors at any point you should not proceed until you resolve the issue, even if you see the success message at the end. You can double check to see if the files were actually made inside the `stock_rom` folder which you can find inside the home folder using the file explorer, but keep in mind that this does not mean they were made correctly if you did run into any errors.
4. To backup the preloader, run `mtk r preloader stock_rom/preloader.bin --parttype=boot1`. After this has finished, you should now be able to see a bunch of files with .bin extension inside the stock_rom folder of your USB drive.
5. Move the stock_rom folder to an external drive or upload it to a cloud storage solution like Google Drive. **Note:** rebooting the Linux ISO will reset the live image and you will lose your backup if you haven't moved it somewhere else.

# Unlock the bootloader
> [!WARNING]
> This will factory reset your phone and you will lose your data!

You need to unlock the bootloader in order to flash the new ROM.   

## For most models
1. [Enter fastboot](#enter-fastboot).
2. Run `fastboot flashing unlock`.
3. Run `fastboot --disable-verity --disable-verification flash vbmeta vbmeta_a.bin`.  

## For F21 Pro and similar models where “press volume up” doesn’t work
1. Turn off the phone.
2. Run `mtk da seccfg unlock`. Connect the cable and wait for the command to finish.
3. [Enter fastboot](#enter-fastboot).
4. Run `fastboot --disable-verity --disable-verification flash vbmeta vbmeta_a.bin`.



# Flash new ROM

>[!NOTE] 
> Before flashing a new ROM, if you have the `F21 Pro`[^Bands] and live in US/Canada and want to flash the American bands, jump to [Flash American bands on F21 Pro](#flash-american-bands-on-f21-pro)

>[!NOTE]
> Some F21 Pro users might have previously installed TWRP. You will need to [remove TWRP](#remove-twrp-from-f21-pro) in order to flash DumberOS.


There are a few LineageOS ROMs available that you can try. I'm going to flash DumberOS as it's currently the best option for these keypad phones.


1. [Enter fastboot](#enter-fastboot) mode if you aren't in it already.
2. Run `fastboot reboot fastboot` and wait for the device to reboot into fastboot**D** (colored text on black background).
3. Erase user data if you are upgrading from the stock ROM. Updating DumberOS doesn’t require this step. Run the following commands.  
`fastboot erase userdata`  
`fastboot erase metadata`
4. Download the appropriate *.img.gz from the [latest build of DumberOS](https://github.com/miki151/dumbdroid_build/releases/latest) onto the Linux ISO or the 2nd USB drive. Choose between G-apps and Vanilla (Micro-g). For the F21 pro use version **30**. For all other phones use **31**. 
5. After the download has finished, extract (unzip) the file by right clicking on it and then clicking `Extract here`. Do **NOT** simply rename it to .img from .img.gz.
6. Run this command from fastboot**D**  
`fastboot flash system "Downloads/???.img"` but replace `???` with the actual filename and wait for it to finish. **Note:** The `"Downloads/???.img"` path assumes you extracted the DumberOS image inside the Downloads folder of the live Linux image.
8. Run `fastboot reboot` and wait for the device to reboot. If Orange State warning appears, press the power button to proceed and wait 5-10 minutes for the new OS to boot.


# Enter fastboot
If you need to enter fastboot: 
1. Turn off the phone if it is not already.
2. Run `python3 mtkfastboot.py`.
3. Conncect the cable and wait until the command forces the device to reboot into fastboot. You should see a text that says "fastboot" at the bottom left of the screen.

> [!TIP]
> Alternatively you could try `mtk payload --metamode FASTBOOT` in [BROM](#device-button-combinations) mode.

# Recover from backup

> [!WARNING]
> This will factory reset your phone and you will lose your data!

## Recover stock ROM without affecting American bands or other partitions
1. Turn off the phone.
2. Assuming your backup is stored on your USB drive, Run `mtk w super "/user/media/exampleName/stock_rom/super.bin"` but replace `exampleName` with the name of your drive as mentioned in [Make a backup](#make-a-backup).
3. Connect your cable and wait for the command to finish running. Once it's done unplug the cable and turn on the phone.

## Full recovery
Follow the three steps in the previous subsection but replace the command in the 2nd step with `mtk wl "/media/user/exampleName/stock_rom"`. This should take about 10 minutes to flash.

> [!NOTE] 
> If you encounter the following error message, ignore it: `Error: couldn't detect partition: partitionName, skipping`.

# Remove TWRP from F21 Pro

If you come from that one infamous guide on XDA where they guide you to install TWRP without making a backup. You have probably been stuck trying to flash DumberOS. That's because fastboot**D** is broken on that particular installation of TWRP.
## Solution
Because there are different hardware revisions of the F21 Pro, I cannot guarantee that this solution will work. That's why it's essential to make a backup first. If it does not work for you then you will need to find a boot image that's compatible with your device and does not have TWRP installed.

1. Make sure that you have [made a backup](#make-a-backup).
2. Turn off the phone.
3. Run `mtk w boot_a TWRPless_F21_Boot/boot_a.bin`.
4. Connect the cable and wait for the command to finish.

TWRP should now be gone.

# Flash American bands on F21 Pro

> [!Note]
> Skip this section if you do not live in US/Canada.

In this section we will go through the process of flashing American bands on the F21 Pro for users who need them.  

- This part should be followed after [unlocking the bootloader](#unlock-the-bootloader) and **before** [flashing a new ROM](#flash-new-rom) because SN Write Tool does not work with LineageOS/DumberOS.
- Make sure that you have [made a backup](#make-a-backup).
- Covered LTE Bands: 2, 4, 12, 13, 17, 66, 71  
This covers most T-Mobile users, in addition to some AT&T support depending on region. 
- **Verizon** will **not** work on DumberOS. As for getting it to work on the stock ROM you will need to flash the 1.1.1 firmware which I won't be covering in this guide. You can look up other guides on how to do that.

## Flash

> [!CAUTION] 
> If you skip SN Write Tool, you’ll get dummy identifiers that may conflict with other devices.

1. **Backup Identifiers**
    1. Go to Settings > About Phone.
    2. Write down these fields: **IMEI**, **WiFi MAC**, **Bluetooth MAC**.

2. **Flash LTE Bands**
    1. Run `mtk wl F30_Modem_Files` and wait for it to finish.

3. **Prepare SN Write Tool (Windows)**
    1. Switch to Windows, download and unzip [SN Write Tool](https://drive.google.com/file/d/1mmiI9kMxqQdlhN6pV-Z44qI3lXHt9ChA).

4. **Restore Identifiers**
    1. Open SN Write Tool
    2. Set:  
        ComPort: USBVCOM
        Target Type: Smart Phone
    3. Click System Config. Under Write Option, check **IMEI**, **BT Address**, and **WiFi Address**.
    4. Under Database File
        1. Check **Load AP DB from DUT** and **Load Modem DB from DUT**.
        2. Click MD1_DB: Select `MDDB_InfoCustomAppSrcP_MT6761S00...EDB` (inside AP DB Base/MT6761).
        3. Click AP_DB: Select `APDB_MT6761_S01__W1947...` (inside AP DB Base/MT6761).
        4. Click Save and return to the main window.
    5. Click Start and input your saved identifiers (no spaces in IMEI, no colons in BT/WiFi).
    6. Hold your phone's Back button, plug it in via USB, and click OK. Wait for the green PASS. If a second window pops up, close it if you already saw PASS.

5. **Verify**
    1. Turn the phone on and check Settings > About Phone to confirm your identifiers are restored.
    2. Dial **\*#\*#3646633#\*#\*** to open Engineer Mode. Go to Band Mode, scroll down and confirm bands `2, 4, 12, 13, 17, 66, 71` are active.
    3. Test calls/texting and internet data on the stock ROM to make sure whatever issue you might face after installing DumberOS isn't related to your carrier.

# Common errors

## Error: write_sparse_skip_chunk: don't care size XXXXXXXXX is not a multiple of the block size XXXX
You probably didn't unzip the ROM file you are trying to flash. Unzip and try again with the unzipped file.

## FAILED(remote: 'Erase is not allowed on locked devices')
You have not unlocked the bootloader because you probably missed a step in [Unlock the bootloader](#unlock-the-bootloader). Go back and redo the steps in that section.

## FAILED (remote: 'This partition doesn't exist')
You are probably trying to flash the `system` partition from fastboot instead of fastboot**D**.  
Run `fastboot reboot fastboot` and wait for the device to reboot into fastboot**D** (colored text on black background).

## FAILED (remote: 'Not enough space to resize partition')
On some devices like the F21 Pro 3GB model, you might run into this error when you try to flash the system partition with DumberOS.

### Solution

You can pick one of the following options to fix it. 

#### Option 1: Delete product partition (Experimental)

> [!CAUTION] 
> Deleting the product partition has not been tested extensively. The side effects on the newly installed ROM are unknown. Usually it is recommended to flash a smaller product image instead.
> But this is the simpler solution, and it would be great if more people could test it. Make sure to have a backup first.  

1. [Enter fastboot](#enter-fastboot)
2. Run `fastboot reboot fastboot` and wait for the device to reboot into fastboot**D**,.
3. Run `fastboot getvar current-slot` to check which slot is currently active (`a` or `b`). Take note of the active slot as we will be using it in the next step.
4. Run `fastboot delete-logical-partition product_a` if your active slot was `a` in the previous step, otherwise replace `product_a` with `product_b` in the command.

You can now repeat steps 6-7 from [Flash the new ROM](#flash-new-rom) section.

#### Option 2: Delete COW partitions
1. [Enter fastboot](#enter-fastboot) mode if you aren't in it already.
2. Run `fastboot getvar all` and check if you have any partitions with the name ending with `cow`. Example: `system_a-cow`. If you have them proceed to the next step, otherwise ignore this option and use [Option 1](#option-1-delete-product-partition-experimental) instead.
3. Run `fastboot getvar current-slot` to check which slot is currently active (`a` or `b`). Take note of the active slot as we will be using it later.

##### For `cow` partitions that are in slot `a` (.e.g `system_a-cow`)

1. Run `fastboot set_active a` to set the active slot to `a`.
2. Run `fastboot reboot fastboot` and wait for the device to reboot into fastboot**D**.
3. Use `fastboot delete-logical-partition examplePartition` to delete the desired `cow` partition. Replace `examplePartition` with the name of the `cow` partition you want to delete (e.g. `system_a-cow`).
4. Repeat the previous step for each `cow` partition in the `a` slot.


##### For `cow` partitions that are in slot `b` (.e.g `system_b-cow`)
1. Run `fastboot set_active b` to set the active slot to `b`.
2. Run `fastboot reboot fastboot` and wait for the device to reboot into fastboot**D**.
3. Use `fastboot delete-logical-partition examplePartition` to delete the desired `cow` partition. Replace `examplePartition` with the name of the `cow` partition you want to delete (e.g. `system_b-cow`).
4. Repeat the previous step for each `cow` partition in the `b` slot.

##### Finally
1. Switch back to your initial active slot with the `fastboot set_active exampleSlot` command, replace `exampleSlot` with `a` or `b` depending on which one was active before deleting the cow partition.
2. Repeat steps 6-7 from [Flash the new ROM](#flash-new-rom) section.


## Dm-verity corruption
A common issue that many run into is the following message appearing on boot and not letting them go past the bootloader after unlocking the device.

```
dm-verity corruption
Your device is corrupt.
It can't be trusted and may not work properly
Press power button to continue.
Or, device will power off in 5s
```

### Solution
Follow step 3-4 from [this section](#for-f21-pro-and-similar-models-where-press-volume-up-doesnt-work). If that doesn't work, you can try this:

1. Turn off the phone.
2. Run `mtk w vbmeta vbmeta_a.bin`.
3. Connect the cable and wait for the command to finish. Then unplug and reboot the phone to see if the message is gone.

> [!TIP]
> Alternatively you could try `mtk da vbmeta 3`.

## Orange state warning 
Your device may show this message on boot. This is normal as long as your device boots after you press the power button and wait 5 seconds.
You don't need to remove it but you can if you wish to, although it may require some effort. Follow the [AlikornSause guide](https://github.com/AlikornSause/Notes-on-QIN-F21-PRO?tab=readme-ov-file#removing-the-orange-state-warning-text) if you are interested.

```
Orange State
Your device has been unlocked and can't be trusted
Your device will boot in 5 seconds
```

## Preloader - [LIB]: Status: Handshake failed
Assuming you are using the Linux ISO in this guide and not some other OS:
1. Make sure your cable matches the description in [Prerequisites](#prerequisites).
2. Press CTRL+C to kill mtkclient.
3. Unplug the cable.
4. Rerun the command.
5. Replug the cable.

If it still doesn't work try again in [BROM](#device-button-combinations) mode. Repeat step 2-4. On step 5 hold the button combo and plug the cable in while still holding the buttons.
Repeat this a couple of times if it still doesn't work.

## An error occured while extracting files. Command exited abnormally.
If you see this error inside the Linux ISO you are probably running out of RAM and the system is crashing because the live image uses RAM for storage. Most likely because your computer has less RAM than what's stated in [Prerequisites](#prerequisites).

# Special Thanks

This guide would not have been possible without the amazing contributions from:

[AlikornSause](https://ko-fi.com/alikornsause)  
[Michal Brzozowski](https://ko-fi.com/dumbdroid)  
[Deathmist](https://github.com/JamiKettunen)  
[CatStoleTheCrown](https://ko-fi.com/storymode)

[^F22]: DumberOS does not work with the F22 non-pro, it uses a 32-bit system and you will have to find a compatible ROM on your own.
[^RAM]: 4GB of RAM is also possible but not recommended because the Linux ISO will crash if you download and extract the DumberOS image on it. You will have to download and extract the DumberOS image from your main operating system on your computer. You would then put it on an external drive, reboot into the Linux ISO and flsah the image with the correct path provided.
[^USB]: At least one USB-A port is required for connecting the phone without adapters because you will most likely run into connection issues if you use an adapter. But you could use an adapter for the USB sticks if you don't have enough USB-A ports on your computer.
[^Apple]: No Apple junk. Unless it has an Intel CPU, the Linux ISO should work fine then. 
[^Drive]: Any other type of external storage device works.
[^Capacity]: 8 + 12 GB is also fine.
[^Cable]: The one included in the box should normally work fine.
[^Bands]: Not applicable to other models.
