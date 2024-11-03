### Install Windows 10/11 in BIOS Legacy Mode on Pre-2012 MacBook (Without an Internal Optical Drive)

> **Note**: You may need to disable System Integrity Protection (SIP) by running `csrutil disable` in macOS recovery mode.

### Step 1: Create a Boot Camp Volume Using macOS

- On MacBooks released before 2012, the original Boot Camp Assistant may require an internal optical drive to create a Boot Camp partition. To bypass this, you can modify the Boot Camp Assistant app:

1. Copy the Boot Camp Assistant app to the desktop.
2. Right-click the copied app, select "Show Package Contents," and locate `Info.plist`.
3. Edit the file by removing "Pre" from two lines:
   ```xml
   <key>PreUSBBootSupportedModels</key> → <key>USBBootSupportedModels</key>
   <key>PreWindows10OnlyModes</key> → <key>Windows10OnlyModes</key>
   ```
4. Save the changes and open the modified Boot Camp Assistant. Plug in a USB drive if needed, then uncheck the options to create a USB installer and download support drivers. You should now be able to create a Boot Camp partition without an internal optical drive.

> **Note**: This modified Boot Camp Assistant will create a Boot Camp partition with a Hybrid MBR, which is essential for BIOS mode booting.

### Step 2: Boot into a WinPE Environment (Using USB) or Another Tool with Windows Command-Line Access

1. Create a bootable USB drive with WinPE or a similar tool that provides access to Windows command-line utilities (e.g., Anhdvboot: https://anhdvboot.com/en/download/).
2. Boot the MacBook from the USB. You can boot the USB in UEFI mode.

### Step 3: Format the Boot Camp Partition for BIOS Legacy Mode

1. Open **Command Prompt** (as Administrator, if needed).
2. Run the following `diskpart` commands to prepare the Boot Camp partition:

   ```cmd
   diskpart
   list disk
   select disk X      # Replace X with the number for your Boot Camp drive
   list partition
   select partition Y  # Replace Y with the partition number for Boot Camp
   format fs=ntfs quick   # Format as NTFS
   active                 # Marks the partition as active for BIOS boot
   assign                 # Assigns a drive letter (e.g., E)
   exit
   ```

### Step 4: Apply the Windows Image to the Boot Camp Partition

```bash
This is a manual step to pre-install Windows on the Boot Camp partition. You can use the app in the release package to complete this step and download the Boot Camp drivers.
```

1. Insert your Windows installation media or mount the ISO, noting its drive letter (e.g., D).
2. Use the `DISM` tool to apply the Windows image (install.wim) to the Boot Camp partition:

   ```cmd
   dism /apply-image /imagefile:D:\sources\install.wim /index:6 /applydir:E:\
   ```

   - Replace `D:\sources\install.wim` with the path to `install.wim` on your installation media.
   - `E:\` is the Boot Camp partition.
   - `index:6` is usually for Windows Pro version from `install.wim` file. You can choose other version you like.
   - To show windows version, run command: `dism /get-wiminfo /wimfile:D:\sources\install.wim`

### Step 5: Configure the Bootloader

After applying the Windows image, set up the bootloader on the Boot Camp partition:

```cmd
bcdboot E:\Windows /s E: /f BIOS
bootsect /nt60 E: /mbr
```

- `E:` is the Boot Camp partition.

This will make the Boot Camp partition bootable in BIOS mode.

### Step 6: Reboot and Test the Installation

1. Reboot your MacBook and hold the **Option** key to access the boot picker.
2. If the "Windows" option appears in the boot picker, congratulations—your installation was successful!
3. If the "Windows" option does not appear, review the steps, especially Step 5, to ensure the bootloader was correctly configured.
