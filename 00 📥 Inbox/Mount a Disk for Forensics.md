
**Tags:** #disk #forensic

1. **Boot using USB**

2. List the available disks:
    ```bash
    lsblk
    ```

3. Mount the main partition:
    ```bash
    sudo mount /dev/sda3 /mnt
    ```

4. Mount the EFI boot partition:
    ```bash
    sudo mount /dev/sda1 /mnt/boot/efi
    ```

5. Bind necessary directories and chroot into the system:
    ```bash
    for i in /dev /dev/pts /proc /sys /run; do sudo mount -R $i /mnt$i; done
    sudo chroot /mnt
    ```

With this last command, you’ll have root access to your installed system. Once the drive is accessed, maintenance commands can be run. For example, refer to [package manager repair commands](https://support.system76.com/articles/package-manager-pop). You can also access your files via **Files** under "Other Locations" -> "Computer" -> "mnt."

6. **Change Permissions for Home Directory**:
    ```bash
    sudo chown myuser:myuser /home/myuser
    ```
    ```bash
    sudo chmod 755 /home/myuser
    ```


