---
tags: cybersecurity, linux, kali-linux
---
# Kali Linux Notes
## Foundations of Kali Linux
### Commands
- pwd - print name of current/working directory
- ls - list directory contents
- touch - change file timestamps
- chmod - change file mode bits
- plocate - find files by name, quickly
- updatedb - update a database for plocate
- which - locate a command
- find - search for files in a directory hierarchy
- ps - report a snapshot of the current processes
- man - an interface to the system reference manuals
- apropos - search the manual page names and descriptions
- top - display Linux processes
- kill - send a signal to a process
- killall - kill processes by name
- grep, egrep, fgrep, rgrep - print lines that match patterns
- sudo, sudoedit - execute a command as another user
- useradd - create a new user or update default new user information
- adduser, addgroup - add or manipulate users or groups
- groupadd - create a new group
- usermod - modify a user account
- passwd - change user password
- chsh - change login shell
- systemd, init - systemd system and service manager
- systemctl - control the systemd system and service manager

### Directories
- /bin - Commands/binary files that have to be available when the system is booted in single-user mode.
- /boot - Where boot files are stored, including the configuration of the boot loader, the kernel, and any initial ramdisk files needed to boot the kernel.
- /dev - A pseudofilesystem that contains entries for hardware devices for programs to access.
- /etc - Configuration files related to the operating system and system services.
- /home - The directory containing the user's home directories.
- /lib - Library files that contain shared code and functions that any program can use.
- /opt - Where optional, third-party software is located.
- /proc - A pseudiofilesystem that has directories containing files related to running processes, including memory maps, the command line used to run the program, and other essential system information related to the program.
- /root - The home directory of the root user.
- /sbin - System binaries that also need to be available in single-user mode.
- /tmp - Where temporary files are stored.
- /usr - Read-only user data (includes bin, doc, lib, sbin, and share subdirectories).
- /var - Variable data, including state information about running processes, logfiles, runtime data, and other temporary files. All these files are expected to change in size or existence during the running of the system.

## Network Security Testing Basics
### Using hping3 for flooding
```bash
sudo hping3 --flood -S -p 80 192.168.86.1
```

### Using hping3 for a LAND attack
```bash
sudo hping3 -S -p 80 192.168.1.1 -a 192.168.1.1
```
