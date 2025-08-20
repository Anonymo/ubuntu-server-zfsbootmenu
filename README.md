# Ubuntu ZFS with zectl install script

This script creates an Ubuntu installation using the ZFS filesystem with zectl boot environment management and systemd-boot. The installation has integrated snapshot management using pyznap. Boot environments can be managed using zectl for easy rollback and system recovery.

Boot environments and snapshots allow you to rollback your system to a previous state if there is a problem. The system automatically creates snapshots on a timer and also when the system is updated with apt. Boot environments can be created and managed using zectl. Snapshots are pruned over time to keep fewer older snapshots.

<details>
<summary><strong>Supported Features</strong></summary>

- Ubuntu 22.04 LTS, 24.04 LTS, 24.10, 25.04, and Linux Mint 22.2 (Zara).
- Root filesystem on ZFS.
- Choose from: Ubuntu Server, Ubuntu Desktop, Kubuntu, Xubuntu, Budgie, Ubuntu MATE, and Linux Mint Cinnamon.
- Single, mirror, raid0, raidz1, raidz2, and raidz3 topologies.
- LUKS and native ZFS encryption.
- systemd-boot bootloader with clean, simple interface.
- zectl boot environment management for easy system rollback.
- Automated system snapshots taken on a timer and also on system updates.
- Creation of a separate encrypted data pool (single/mirror/raidz).

</details>

## Usage
Boot the system with an Ubuntu live desktop iso. Use an Ubuntu iso to boot from even if installing a different Ubuntu flavour such as Kubuntu or Linux Mint. **For Linux Mint installations, use Ubuntu 24.04 LTS media** - the script will install the Ubuntu base system and then add Linux Mint repositories and packages. Start the terminal (Ctrl+Alt+T) and enter the following.

First install git (Ubuntu live CD doesn't include it by default):

	sudo apt install -y git

Then clone the repository:

	git clone -b zectl https://github.com/Anonymo/ubuntu-server-zfsbootmenu.git ~/ubuntu-server-zfsbootmenu
    cd ~/ubuntu-server-zfsbootmenu
    chmod +x ubuntu_server_encrypted_root_zfs.sh
	
Edit the variables in the ubuntu_server_encrypted_root_zfs.sh file to your preferences.

	nano ubuntu_server_encrypted_root_zfs.sh

Or use this one-liner to install git, clone, setup, and open the editor in one command:

	sudo apt install -y git && git clone -b zectl https://github.com/Anonymo/ubuntu-server-zfsbootmenu.git ~/ubuntu-server-zfsbootmenu && cd ~/ubuntu-server-zfsbootmenu && chmod +x ubuntu_server_encrypted_root_zfs.sh && gnome-text-editor ubuntu_server_encrypted_root_zfs.sh
	
Run the "initial" option of the script with sudo privileges.

	sudo ./ubuntu_server_encrypted_root_zfs.sh initial

Reboot after the initial installation completes and login to the new install. Username and password is as set in the script variables. Then run the second part of the script.

	sudo ./ubuntu_server_encrypted_root_zfs.sh postreboot

## Installation Recovery

<details>
<summary><strong>Resume interrupted installations</strong></summary>

If your installation is interrupted (network disconnection, system crash, etc.), you can resume where you left off:

	sudo ./ubuntu_server_encrypted_root_zfs.sh status    # Check installation progress
	sudo ./ubuntu_server_encrypted_root_zfs.sh resume    # Resume from last checkpoint

The script automatically detects previous installations and offers to resume when you run the `initial` command.

</details>

<details>
<summary><strong>Boot Environment Management</strong></summary>

This installation uses zectl for boot environment management with systemd-boot. Boot environments allow you to create snapshots of your entire system that can be booted independently.

Common zectl commands after installation:
- `sudo zectl create new-environment` - Create a new boot environment
- `sudo zectl list` - List all boot environments  
- `sudo zectl activate new-environment` - Set boot environment as default
- `sudo zectl destroy old-environment` - Remove unused boot environment

Note: Boot environment selection is done at the systemd-boot menu during startup. Remote SSH access during boot is not available with this setup.

</details>

<details>
<summary><strong>Optional: Create a zfs data pool</strong></summary>

The script includes an optional feature to create an encrypted zfs data pool on a non-root drive. The data pool will be unlocked automatically after the root drive password is entered at boot.

	sudo ./ubuntu_server_encrypted_root_zfs.sh datapool

</details>

<details>
<summary><strong>FAQ</strong></summary>

Additional guidance and notes can be found in the script.

<details>
<summary>1. How do I manage boot environments with zectl?</summary>

You can manage boot environments and rollback to previous states using zectl commands. This is useful if an upgrade doesn't work and you wish to revert to a previous state. I recommend testing any changes out in a virtual machine first before rolling them out in a production environment.

Common zectl operations:
- `sudo zectl list` - Show all available boot environments
- `sudo zectl create backup-before-upgrade` - Create a new boot environment before making changes
- `sudo zectl activate backup-before-upgrade` - Set a different boot environment as default
- Boot environments are selectable at the systemd-boot menu during startup

To rollback after a failed upgrade:
1. Reboot and select the previous boot environment from the systemd-boot menu
2. Once booted into the working environment, run `sudo zectl activate current-environment-name` to make it the default
3. Optionally, remove the failed boot environment with `sudo zectl destroy failed-environment-name`

</details>

<details>
<summary>2. How do I delete a boot environment I no longer need?</summary>

You can delete a boot environment you no longer need using zectl commands. This can be done from any running boot environment.

- Delete a boot environment using zectl (recommended):
  - List boot environments: `sudo zectl list`
  - Delete the unwanted environment: `sudo zectl destroy environment-name`
  - Note: You cannot destroy the currently active boot environment

- Delete a boot environment manually using zfs commands:
  - List ZFS datasets: `zfs list`
  - Find the dataset for the boot environment (e.g., "rpool/ROOT/ubuntu.2022.10.01")
  - Delete it: `sudo zfs destroy -r rpool/ROOT/ubuntu.2022.10.01`

</details>

<details>
<summary>3. Can I upgrade the system normally using do-release-upgrade?</summary>

- systemd-boot and zectl

  Ubuntu release upgrades should work normally with this setup. The systemd-boot bootloader and zectl boot environment manager are maintained as part of Ubuntu and should continue to work with newer ZFS versions. However, it's always recommended to create a test system in a virtual machine first to duplicate your setup and test the upgrade process.
- Pyznap

  Pyznap is not included as a package in the ubuntu repos at present. It may need to be re-compiled and re-installed. You can reference the install script for the relevant code to re-compile and re-install. 

</details>

<details>
<summary>4. How do I change the password on a natively encrypted zfs root pool?</summary>

You can change the password of your encrypted root as follows. Change "rpool" to the name of your root pool.
   - Update root pool password file.

     `nano /etc/zfs/rpool.key`
   - Update root pool key.

     `zfs change-key -o keylocation=file:///etc/zfs/rpool.key -o keyformat=passphrase rpool`
   - Optional: If you have an encrypted data pool that unlocks at boot using the root pool password, then update its key too. Change "datapool" to the name of your data pool.

     `zfs change-key -o keylocation=file:///etc/zfs/rpool.key -o keyformat=passphrase datapool`
   - Update initramfs.

     `update-initramfs -u -k all`

</details>

</details>

## Discussion threads
Please use the discussions section. \
https://github.com/Sithuk/ubuntu-server-zfsbootmenu/discussions

For historical reference, the initial discussion thread can be found on reddit.
https://www.reddit.com/r/zfs/comments/mj4nfa/ubuntu_server_2104_native_encrypted_root_on_zfs/

## Credits
ahesford E39M5S62/zdykstra (https://github.com/zbm-dev/zfsbootmenu)

zectl (https://github.com/johnramsden/zectl)

cythoning (https://github.com/yboetz/pyznap)

rlaager (https://openzfs.github.io/openzfs-docs/Getting%20Started/Ubuntu/Ubuntu%2022.04%20Root%20on%20ZFS.html)
