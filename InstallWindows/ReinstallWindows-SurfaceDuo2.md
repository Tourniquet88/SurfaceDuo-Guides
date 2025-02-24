# Reinstall Windows on Surface Duo 2

## Files/Tools Needed 📃

- TWRP image: [surfaceduo2-twrp.img](https://github.com/WOA-Project/SurfaceDuo-Guides/raw/main/InstallWindows/Files/surfaceduo2-twrp.img)
- Mass Storage Shell Script: [msc.tar](https://github.com/WOA-Project/SurfaceDuo-Guides/raw/main/InstallWindows/Files/msc.tar)
- Windows UEFI: [SM8350.UEFI.Surface.Duo.2.zip/uefi.img](https://github.com/WOA-Project/SurfaceDuoPkg/releases/)
- [Platform Tools from Google (ADB and Fastboot)](https://developer.android.com/studio/releases/platform-tools)
- An ARM64 Windows build of your choice that meets the minimum system requirements (specifically the install.wim file). You can use [UUPMediaCreator](https://github.com/gus33000/UUPMediaCreator) for this. [Here's a guide on how to use it.](https://github.com/WOA-Project/SurfaceDuo-Guides/blob/main/CreateWindowsISO.md)
- The driver set: [SurfaceDuo-Drivers-Full.zip](https://github.com/WOA-Project/SurfaceDuo-Drivers/releases/)
- DriverUpdater, to install the driver set: [DriverUpdater](https://github.com/WOA-Project/DriverUpdater/releases/)
- A Windows PC to build the Windows ISO, apply it onto the phone from mass storage, add drivers to the installation, configure ESP

> [!WARNING]
> - Don't create partitions from Mass Storage Mode on Windows (because ABL will break with blank/spaces in names), your phone may be irrecoverable otherwise
> - If you see a warning and/or error during the process, it is not normal. Contact us on telegram if you see anything odd, but do not continue or proceed on your own, you will break things further.
> - Don't rerun the commands if you interrupt the process. You may break your partition table.
> - Do not run all commands at once.
> - Do not commit *any* typo with *any* commands.
> - Be familiar with command line interfaces.
> - When using TWRP, it is normal and expected for the phone to be detected as an Asus phone and for touch to not work.

> [!IMPORTANT]
> **THIS WILL WIPE ALL YOUR WINDOWS DATA**
>
> We don't take any responsibility for any damage done to your phone. By following this guide, you agree to take full responsibility of your actions. We have done some testing,
>
> but this is **STILL IN PREVIEW** and things can go wrong.

**PLEASE READ AND BE SURE TO UNDERSTAND THE ENTIRE GUIDE BEFORE STARTING**

# Steps 🛠️

## Getting to Mass Storage Mode

- Reboot into the Bootloader mode by running this command while inside Android™:

```batch
adb reboot bootloader
```

![Surface Duo in Bootloader mode](https://github.com/WOA-Project/SurfaceDuo-Guides/assets/3755345/eb19d500-4849-4ded-bd0c-894e4ac56486)
_Image of what you should see right now: Surface Duo in Bootloader mode_

- Now let's boot TWRP:

```batch
fastboot boot surfaceduo2-twrp.img
```

Once inside TWRP, touch will not be working and the device will say it is locked. This is completely normal.
- Let's load the mass storage shell script in order to boot into Mass Storage from TWRP

```batch
adb shell "setenforce 0"
adb push <path to downloaded msc.tar> /sdcard/
adb shell "tar -xf /sdcard/msc.tar -C /sdcard --no-same-owner"
adb shell "sh /sdcard/msc.sh"
```

Surface Duo should now be in USB 3 SuperSpeed (or what the USB-IF currently calls it) Mass Storage Mode.

## Formatting the existing Windows partition

- Mount the partitions you made when you first installed Windows and assign them some letters:

```batch
⚠️ THESE ARE NOT ALL COMMANDS. DISKPART COMMANDS VARY A LOT, SO THESE ARE SOME ROUGH INSTRUCTIONS.
ACTUAL COMMANDS START WITH AN HASHTAG (which you will need to remove)
YOU DO NOT HAVE TO USE Y or X, THEY ONLY NEED TO BE FREE LETTERS. IF LETTERS DON'T ASSIGN FINE, USE ANOTHER ONE.
IF ONE PARTITION IS ALREADY ASSIGNED, YOU ALSO DO NOT NEED TO ASSIGN IT AGAIN IF YOU DONT WANT TO.

# list disk
Find the Surface Duo Disk, and take note of the number.
# select disk <number>
# list partition
You will be able to recognize the partitions you made the first time you installed Windows by their size.
Take note of the ESP and WIN partition numbers.
# select partition <esp-partition-number>
# assign letter=Y:
# select partition <win-partition-number>
# assign letter=X:
```

- You will have two partitions loaded, one is the ESP partition, and the other is the Win partition. Take note of the letters you've used.

> [!WARNING]
> From now on we will assume X: is the Win partition and that Y: is the ESP partition for all the commands. Replace them correctly with what you previously picked or you will lose data on your PC.

- Now open the file explorer, find the just-mounted Windows drive. BE CAREFUL! If you choose the wrong device, you WILL LOSE IMPORTANT DATA ON YOUR PC! Check that the letter is the same one you've assigned before. In our case, it's X:\.
- Once you have made sure you have found the right partition, right click on it and select "Format".
- Make sure the selected file system is NTFS, that "Quick Format" is checked, and leave the rest as it is, and press "Start".

Now the Windows Partition on your Surface Duo should be empty. Let's go ahead and reinstall everything.

## Installing Windows

- We will need our install.wim file now. If you haven't it already, you can [use this guide](https://github.com/WOA-Project/SurfaceDuo-Guides/blob/main/CreateWindowsISO.md). When you are ready, run these commands:

```batch
dism /apply-image /ImageFile:"<path to install.wim>" /index:1 /ApplyDir:X:\
```

This will take a bit of time. Go make some coffee ☕ or some tea 🍵.

- Once that is done:

```batch
rmdir /Q /S Y:\EFI
bcdboot X:\Windows /s Y: /f UEFI
```

Windows is now installed but has no drivers.

## Installing the drivers

- Extract the drivers, Extract driver updater, and from the command prompt in the DriverUpdater.exe directory:

```batch
DriverUpdater.exe -d "<path to extracted drivers>\definitions\Desktop\ARM64\Internal\zeta.txt" -r "<path to extracted drivers>" -p X:\
```

## Boot Windows 🚀

We are ready to boot!

You'll have two methods of booting Windows.

- Manual booting with a PC
    - Pros: You can freely update Android™
    - Cons: You will need a PC to boot to Windows

- Enabling Dual Boot
    - Pros: You'll be able to boot Windows directly from the device
    - Cons: Every time you update Android™, you'll have to follow [this guide](https://github.com/WOA-Project/SurfaceDuo-Guides/blob/main/InstallWindows/DualBoot-SurfaceDuo.md)

In case you want the dual boot option, then follow [this guide](https://github.com/WOA-Project/SurfaceDuo-Guides/blob/main/InstallWindows/DualBoot-SurfaceDuo.md)

---
<details>
  <summary>In case you want to manually boot each time: (Click to expand)</summary>
  <p>

Reboot your device to the Bootloader mode, using adb or from the recovery.

![Surface Duo in Bootloader mode](https://github.com/WOA-Project/SurfaceDuo-Guides/assets/3755345/eb19d500-4849-4ded-bd0c-894e4ac56486)
_Image of what you should see right now: Surface Duo in Bootloader mode_

Let's boot the UEFI, from a command prompt:

```batch
fastboot boot surfaceduo2-uefi.img
```

This step above will be needed every time you will want to boot Windows and needs to be done from the Bootloader mode.

If you did everything right, Windows will now boot! Enjoy!

**Note:** If the Touch keyboard won't show up in OOBE, touch somewhere else (to let the text box loose focus) and then touch into the text box again. As an alternative, you can use the On-Screen Keyboard.
  </p>
</details>
