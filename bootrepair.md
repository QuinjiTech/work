If updating GRUB didn't reveal the Windows boot option, there are a few other methods you can try from within Linux Mint to resolve this issue. It's important to proceed with caution and ensure that you have backups of your important data.

### 1. Re-Check GRUB Configuration

First, double-check your GRUB configuration:

- Open Terminal.
- Edit the GRUB configuration file:
  ```bash
  sudo nano /etc/default/grub
  ```
- Ensure that the line `GRUB_DISABLE_OS_PROBER=false` is present and not commented out (no `#` at the beginning). If it's not there, add it.
- Save and exit (Ctrl+O, Enter, Ctrl+X in nano).
- Update GRUB again:
  ```bash
  sudo update-grub
  ```

### 2. Manually Add Windows to GRUB

If the OS prober doesn't find Windows, you can manually add it to the GRUB menu:

- First, find out where Windows is installed. You can use `lsblk`, `fdisk -l`, or GParted to identify the Windows partition.
- Then, create a custom menu entry for Windows. Edit the custom GRUB file:
  ```bash
  sudo nano /etc/grub.d/40_custom
  ```
- Add the following lines, replacing `hd0,gpt1` with the correct identifiers for your Windows EFI partition (usually the EFI partition is the first partition on the first drive, hence `hd0,gpt1`):
  ```
  menuentry "Windows 10" {
      insmod part_gpt
      insmod fat
      insmod chain
      set root='(hd0,gpt1)'
      chainloader /EFI/Microsoft/Boot/bootmgfw.efi
  }
  ```
- Save and exit.
- Update GRUB:
  ```bash
  sudo update-grub
  ```

### 3. Check EFI Partitions

- Ensure that your EFI partition is healthy and properly configured. The EFI partition is typically a FAT32 partition with a size around 100-500 MB.
- Make sure it's mounted correctly and contains the Windows bootloader files. You can mount and explore it using the file manager or command line.

### 4. Use Boot-Repair

- If manual methods don't work, consider using the Boot-Repair tool. It's a simple tool to repair frequent boot issues.
- Install Boot-Repair in Mint:
  ```bash
  sudo add-apt-repository ppa:yannubuntu/boot-repair
  sudo apt-get update
  sudo apt-get install -y boot-repair
  boot-repair
  ```
- Run Boot-Repair and use the Recommended repair option. This can often resolve dual-boot issues automatically.

- After using Boot-Repar I came to know that my windowsis running on Leagacy mode whereas I was using Linux using UEFI. So make windows operation these are steps which I performed.

If your Windows is installed in legacy mode and you have deleted the Linux partitions including the EFI partition created for the Linux installation, you should restore the Windows bootloader for a legacy boot. Here's how to proceed with a Windows installation media:

1. **Boot from the Windows Installation Media**:
   - Insert your Windows installation media into the computer.
   - Boot from it by selecting it in the BIOS boot menu or by setting it as the primary boot device in the BIOS settings.

2. **Navigate to the Command Prompt**:
   - After you've booted from the media, select the correct language and keyboard input method, then click "Next."
   - Choose "Repair your computer" at the bottom of the window.
   - On the "Choose an option" screen, select "Troubleshoot."
   - Then select "Advanced options" and choose "Command Prompt."

3. **Use the Command Prompt to Repair the Bootloader**:
   - Once in the Command Prompt, type the following commands:
     ```
     bootrec /fixmbr
     bootrec /fixboot
     bootrec /scanos
     bootrec /rebuildbcd
     ```
   - These commands will attempt to fix the Master Boot Record (MBR) and the boot configuration data.

   **Again got error here at last step then performed this 4 and 5**

4. **Use `diskpart` to identify and assign a letter to the EFI partition** (if not already done):
   ```
   diskpart
   list disk
   select disk # (where # is the number of the disk with the EFI partition)
   list partition
   select partition # (the EFI partition number)
   assign letter=S
   exit
   ```

5. **Recreate the UEFI boot files**:
   ```
   bcdboot C:\Windows /s S: /f UEFI
   ```
   - Replace `C:` with the letter of the Windows installed partition if different, and `S:` with the letter you assigned to the EFI partition.
   - Here in my case it picked it up with F: and it fixed the issue

After these steps, reboot the system and check if Windows boots up. If you are not comfortable with these operations or if the problem persists, seek professional assistance.

4. **Reboot the Computer**:
   - After executing the commands, type `exit` to close the Command Prompt.
   - Restart your computer by typing `shutdown /r /t 0` or by using the power button.

5. **Check if Windows Boots**:
   - Remove the installation media and let the computer boot from the hard drive.
   - Check to see if Windows boots up normally.

If you encounter an "Access is denied" error when running `bootrec /fixboot`, you can try the following workaround:

1. **Boot back into the Windows Installation Media** and open the Command Prompt as before.
2. Type the following commands:
   ```
   diskpart
   list disk
   select disk # (where # is the number of your Windows disk)
   list volume
   ```
3. Find the volume number for your System Reserved partition (or the primary Windows partition if you don't have a System Reserved partition).

### Did this first
4. Then type:
   ```
   select volume # (where # is the number of the System Reserved or Windows partition)
   active
   exit
   ```
5. Now, try the `bootrec` commands again.

If Windows still doesn't boot, you might have to consider restoring from a system image backup if you have one, or performing a repair installation of Windows. If these options are not viable or the issue persists, seeking professional technical support may be necessary.