# Install CollabJam

A tool that enables real-time collaboration when deisgning vibrotactile experiences.

## Frontend
<!-- 
### How to Install the Binary
1. Download the latest binary from the [Releases](https://github.com/TactileVision/TactileCollab/releases) page
2. Doubleclick the executable to install the software -->

**NOTE:** For Windows you need a bluetooth-dongle, which use the WinUsb Driver (for flashing Bluetooth Adapter use zadig). Before you start the frontend, make sure that the bluetooth-dongle is connected to your computer with the correct driver and that it's currently not used.

If you downloaded the executable binary you simply need to double click that file to start the frontend.

You can find a tutorial about the features and usage of this software in the [wiki](https://github.com/TactileVision/TactileCollab/wiki/Frontend-Tutorial).

### How to Build from Sources

This software was tested with NodeJS **v14.18.2**. Make sure to install the same version on your computer. You can easliy install or change the active Node version with a version manager like [NVM](https://github.com/nvm-sh/nvm) or [N](https://www.npmjs.com/package/n). Please read the dedicated documentation how to do that.

Clone the repository on your computer, e.g. `git clone https://github.com/TactileVision/CollabJam-Client`

1. Execute `npm install`
    - If an error related to *electron modules* occures, you need to remove all folders containing "electron" in the name inside of the `node_modules` folder.
    - Execute `npm install` once more
2. **Only for Windows users:** check the `node_modules/abandonware/noble` for other requiered packages

### How to Run from Sources

**NOTE:** If the server doesn't run on the same computer you need to set the server IP address in `src\core\WebSocketManager\index.ts`

Inside the repository, execute `npm run electron:serve`

***

## Backend

This software was tested with NodeJS **v14.18.2**. Make sure to install the same version on your computer. You can easliy install or change the active Node version with a version manager like [NVM](https://github.com/nvm-sh/nvm) or [N](https://www.npmjs.com/package/n). Please read the dedicated documentation how to do that.

Clone the repository on your computer, e.g. `git clone https://github.com/TactileVision/CollabJam-Server`

### How to Install

Inside the repository, execute `npm install`

### How to Run

Inside the repository, execute `npm run serve`
