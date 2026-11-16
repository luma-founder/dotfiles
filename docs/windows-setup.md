# Windows Setup

## Install 

Press Start, type **PowerShell**, right-click **Windows PowerShell**, and choose **Run as administrator**. Then run:

    wsl --install -d Ubuntu

When it finishes, restart Windows.

## Create your Linux user

After the reboot, Ubuntu usually opens a window by itself and finishes installing. If it doesn’t, open the Start menu and launch Ubuntu. 
It will ask for a username and password. These are just for Linux: they don’t need to match your Windows login, and the password is what sudo will ask for.

## Update Ubuntu and install the basics

In the Ubuntu window:

    sudo apt update && sudo apt upgrade -y
    sudo apt install -y git curl build-essential

