# Windows Setup

## Install wsl

Press Start, type **PowerShell**, right-click **Windows PowerShell**, and choose **Run as administrator**. Then run:

    wsl --install -d Ubuntu

When it finishes, restart Windows.

## Create your Linux user

After the reboot, Ubuntu usually opens a window by itself and finishes installing. If it doesn’t, open the Start menu and launch Ubuntu. 
It will ask for a username and password. These are just for Linux: they don’t need to match your Windows login, and the password is what sudo will ask for.

## Install dotfiles

    sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply $GITHUB_USERNAME

# About WSL

Ubuntu’s whole filesystem is stored in one virtual disk file on your Windows drive:

    C:\Users\<USERNAME>\AppData\Local\wsl\<UUID>\ext4.vhdx

It’s an ext4 disk image, so don’t open or edit that file directly. You can browse it in two ways instead:

- From Windows: Explorer at `\\wsl.localhost\Ubuntu\home\USERNAME`. “Linux” also appears in Explorer’s sidebar. Or run `explorer.exe .` from any Ubuntu folder.
- From Linux: your Windows drives appear as `/mnt/c`, `/mnt/d` and so on. `C:\Users\USERNAME` is `/mnt/c/Users/USERNAME`.

