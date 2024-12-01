<!-- /docs/emmcFlash.md -->
# Write Image to USB Drive

1. Install the Raspberry Pi Imager and insert a USB Flash Drive into your PC.
2. Select **Choose Device** -> **Raspberry Pi 4**.

   ![Raspberry Pi Imager - Device Selection](/images/rpiImager1.png)

3. Select **Choose OS** -> **Raspberry Pi OS (other)** -> **Raspberry Pi OS Lite (64-bit)**.

   ![Raspberry Pi Imager - OS Selection](/images/rpiImager2.png)

4. Select **Choose Storage** -> your USB flash drive.

   ![Raspberry Pi Imager - Storage Selection](/images/rpiImager3.png)

5. Click **Next** and then **Edit Settings**.
   - Select the **General** tab and configure the hostname, username, password, and locale.

   ![Raspberry Pi Imager - General Settings](/images/rpiImager5.png)

6. Select the **Services** tab, enable **SSH**, and choose **Use password authentication**.
   - Click **Save**.

   ![Raspberry Pi Imager - Services Settings](/images/rpiImager6.png)

7. Click **Yes** when prompted to apply customization settings.

   ![Customization Confirmation](/images/rpiImager7.png)

8. Note the warning message, and if you agree, select **Yes**.

   ![Customization Warning](/images/rpiImager8.png)

---

# Install CM4 and USB Flash Drive

1. Install the CM4 module into **Node 1** and insert the prepared USB flash drive into the Turing Pi's USB-2 port.

   ![CM4 Installation](/images/TPi1.png)

2.a If you have the newer Turing Pi V2.5.x board:
   - Log in to [https://turingpi.local](https://turingpi.local).
   - Select **Node 1 USB-A Compatibility Mode** and click **Change**.

   ![USB Compatibility Mode](/images/USBComp.png)

2.b If you have the older Turing Pi V2.4.x board:
   - Log in to [https://turingpi.local](https://turingpi.local).
   - Select USB Mode: Host / Connected node: Node 1

     Note: A little quirk if using newer usb3 thumb drives, they don't sometimes work. eg: on a SanDisk Cruzer Blade thumbdrive I got xHC-CMD error:4 type: 11 errors and no boot, using an older USB2 drive worked fine, I used a USB-A (USB2) to microSD card reader without issue 

   ![USB Host Mode](/images/USBComp2.png)
---

# Obtain the Raspberry Pi Download URL

1. Browse to [https://www.raspberrypi.com/software/operating-systems/](https://www.raspberrypi.com/software/operating-systems/) on your PC.
2. Right-click the **Download** button and select **Copy Link** for the OS version you wish to install.
3. Paste the link into Notepad for later reference, e.g.,

   ```plaintext
   https://downloads.raspberrypi.com/raspios_armhf/images/raspios_armhf-2024-11-19/2024-11-19-raspios-bookworm-armhf.img.xz
   ```

   ![OS Download URL](/images/osURL.png)

---

# Boot from USB and Flash to eMMC

1. SSH into the Turing Pi using a terminal or terminal emulator (e.g., PuTTY):

   ```bash
   ssh root@turingpi.local
   ```
   Password: **turing**

2. Power on Node 1:

   ```bash
   tpi power on -n1
   ```

   ![SSH Command](/images/ssh7.png)

3. Wait for the system to start and expand the OS partition. Then, SSH into the Raspberry Pi using the hostname, username, and password you configured earlier:

   ```bash
   ssh pi@raspberrypi.local
   ```

   **Note:** Do not remove the USB flash drive as the CM4 has booted from it.

4. Download the Raspberry Pi OS using `wget` and the previously copied URL:

   ```bash
   wget https://downloads.raspberrypi.com/raspios_armhf/images/raspios_armhf-2024-11-19/2024-11-19-raspios-bookworm-armhf.img.xz
   ```

   ![Download Image](/images/ssh1.png)

5. Unarchive the downloaded image using `unxz` (this may take a while):

   ```bash
   unxz 2024-11-19-raspios-bookworm-armhf.img.xz
   ```

   ![Unarchive Image](/images/ssh2.png)

6. Check storage disks and partitions. `sda` represents the USB flash drive, and `mmcblk0` is the eMMC storage:

   ```bash
   lsblk
   ```

   ![Storage Disks](/images/ssh3.png)

7. Write the image to the CM4's eMMC using `dd` (grab another coffee):

   ```bash
   sudo dd if=2024-11-19-raspios-bookworm-armhf.img of=/dev/mmcblk0 bs=10MB
   ```

   ![Flash Image](/images/ssh4.png)

---

# Enable SSH and the Default Pi User Account

1. The latest Raspberry Pi OS versions do not enable SSH or create a default user by default.
   - Verify the new partitions: `boot` (mmcblk0p1) and `os` (mmcblk0p2):

     ```bash
     lsblk
     ```

     ![Partitions](/images/ssh5.png)

2. Create and mount the boot drive:

   ```bash
   sudo mkdir /media/rootdrive/
   sudo mount /dev/mmcblk0p1 /media/rootdrive/
   ```

3. Create the `ssh` and `userconf` files:

   ```bash
   sudo touch /media/rootdrive/ssh
   sudo touch /media/rootdrive/userconf
   ```

4. Edit the `userconf` file to create the default user (`pi:raspberry`):

   ```bash
   sudo nano /media/rootdrive/userconf
   ```

   Add the following line and save:

   ```plaintext
   pi:$6$c70VpvPsVNCG0YR5$l5vWWLsLko9Kj65gcQ8qvMkuOoRkEagI90qi3F/Y7rm8eNYZHW8CY6BOIKwMH7a3YYzZYL90zf304cAHLFaZE0
   ```

   Press `Ctrl+X`, then `Y`, and `Enter` to save.

5. Shutdown the CM4:

   ```bash
   sudo shutdown now
   ```

6. Power off Node 1 via Turing Pi SSH or the BMC UI:

   ```bash
   tpi power off -n1
   ```

   ![Shutdown Node](/images/ssh6.png)

7. Remove the USB flash drive, then power on Node 1:

   ```bash
   tpi power on -n1
   ```

   ![Power On Node](/images/ssh7.png)

---

# Logging In

1. SSH into the CM4:

   ```bash
   ssh pi@raspberrypi.local
   ```
   Password: **raspberry**

2. If you encounter an SSH key error, remove the old key from the `known_hosts` file:

   ```bash
   ssh-keygen -f ~/.ssh/known_hosts -R raspberrypi.local
   ```

   ![SSH Key Error](/images/ssh10.png)

3. Once logged in, confirm eMMC storage (`mmcblk0`) and its partitions (`mmcblk0p1`, `mmcblk0p2`):

   ```bash
   lsblk
   ```

   ![eMMC Partitions](/images/ssh11.png)
