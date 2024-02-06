
## Server (TODO)

- NodeJS application
- Responsibilities
  - Sharing rooms with each client
  - Receiving and broadcasting tactile messages on a per room basis

### Data Structures

- Room Configuration
- Tacton JSON

## Client (TODO)

<!-- ### Application Information -->
The Collabjam-Client is an Electron application and requires basic knowledge about:

- Typescript
- Electron
- Vue.js
  - VueX Store
  - Vuetify Library
- Node.js
- Bluetooth Low Energy


```mermaid
%% Flowchart describing the basic information flow structure of CollabJam
flowchart LR
 SRV[Collabjam Server] <--->|Websocket| BE
 TDI[Tactile Displays] <--->|BLE| BE
 subgraph "Collabjam Client (Electron)"
    direction LR
    BE[Backend] <-->|IPC| FE[Frontend]
 end
 FE<--->|Web Gamepad API|GP[Gamepad]
```

### Electron

The client is split in two parts: frontend and backend. The frontend contains components that, outside of Electron, would run in a browser, the backend includes the parts that would run on a remote server.
On top of being an Electron app, Collabjam-Client is build with [Vue.js](https://vuejs.org). Therefore, for building the electron app [Vue CLI Plugin Electron Builder](https://nklayman.github.io/vue-cli-plugin-electron-builder/) is used, instead of the vanilla [Electron builder](https://www.electron.build).

Relevant Files for working with the Vue+Electron structure are:
| File          | Purpose                                                                                                    |
| ------------- | ---------------------------------------------------------------------------------------------------------- |
| main.js       | Entry point for the renderer part of the electron app. Initialization of Vue, Vue store and input handling |
| App.vue       | Contains the basic layout of the application. All other views are inserted into this View                  |
| background.ts | The file where the application window gets configured(resolution, title, ...) and instantiated             |
| preload.js    | Defines which functions are exposed between renderer and main process (see IPC, further reading)           |

#### IPC 

To communicate between frontend/renderer and backend/os level components,elctron offers inter-process communication[^1]

[^1]:<https://www.electronjs.org/docs/latest/tutorial/ipc>)

- `ipcMain` (Communicate asynchronously from the main process to renderer processes.)
- `ipcRenderer` (Communicate asynchronously from a renderer process to the main process.)

```mermaid
flowchart LR
M[Main Process] --->|"sendMessageToRenderer() -- window.api.receive()"| R[Renderer Process]
R ---> |"ipcMain.on() -- window.api.send()"| M
```

- The renderer process uses `window.api.send()` and `window.api.receive()` to communicate with the main process
  - The sending functions are spread across the codebase
  - The receiving functions of the renderer process are all clustered in `core/IPC/IpcRendererListener.ts`
- The main process uses the `sendMessageToRenderer()` to send data to the frontend. The `ipcMain.on` function allows the main process to receive data from the renderer.
  - Both the `sendMessageToRenderer()` and all receiving functions of the main process are defined in  `core/IPC/IpcMainController.ts`

##### Messages

- Strings define the different message types in IPC. The messages for the Collabjam-Client are define in the `[core/IPC/IpcChannels.ts]` file
- (⚠️WIP) The messages are structured in the `function.target.message` scheme
  - e.g. `bluetooth.main.writeAmplitudeBuffer` is a message coming from the renderer process targeting to the bluetooth functionality of the main process
  - e.g. `bluetooth.renderer.connectedToDevice` is a message from the main process, informing the renderer process about a newly connetec device

### Folder Structure

- Multiple categroies, ordered by level of fundamentality
  - App: Infrastructure of the application like Routing, Sidebar, Root View
  - Core: Base functionality of the CollabJam application like IPC, BLE Connection, Basic Tactile Display API
  - ⚠️ Features: Views and functions based on top of the core features

- Each (sub)category is a folder
- If there is both frontend and backend code for the feature, split in `main` and `render`
- Use a `types` folder for definition of interfaces the feature provides

### Frontend / Renderer Process

- Vue.js
- Vuetify

### Backend/ Main Process

The Collabjam-Client backend communicates with the Collabjam-Server via websocket and handles the BLE connection with tactile displays running the Collabjam-Firmware.

#### Bluetooth

The [@abandonware/noble](https://github.com/abandonware/noble) package handles working with BLE peripherals. There are some drawbacks using the package.
  
  1. The package is community maintained (hence the term *abandonware*)
  2. Extra work needed to run on  Windows and Linux bases systems (TODO: Add Link to package readme).
     1. ⚠️ At the moment we're not able to actually build a Windows version, because of errors occurring on launch. Serving the application is the only option for Windows based machines right now
     2. ⚠️ Getting the package to run on Linux requires work in the Terminal (Permissions) TODO:(Add link to Richards docs )
  3. ⚠️ No knowledge about already connected devices. When reloading the app, while being connected to a device, the package does not inform about the connection.

In Collabjam-Client basic bluetooth functionality is implemented in `core/Ble/main`.

- `BleController` interfaces with the `@abandonware/noble` package
- `BlePeripheralConnectionManager` stores discovered and connected peripherals and offers an interface to change the connection status of peripherals
- `BleServices` defines the custom BLE-service used to interface with Collabjam-Firmware. The service is define by one service uuid and many characteristics that also have an uuid and an optional callback method that will be called when the characteristic is discovered.

#### Tactile Display API

On top of the `core/Ble` stack, `core/TactileDisplays/main/TactileDisplayCharacteristicWriter` exposes  functions to interface with the tactile dislay on a application level. Currently, the amplitude and the frequency of an tactile display can be changed.

```Typescript
// TactileDisplayCharacteristicWriter.ts
// Peripheral originates from @abandonware/noble
export const writeAmplitudeBuffer = (device: Peripheral, taskList: TactileTask[]) => {
    const service = device.services.find((x) => x.uuid === tactileDisplayService.service.uuid)
    if (service !== undefined) {
        const characteristic = service.characteristics.find((characteristic) => characteristic.uuid === tactileDisplayService.characteristics!.amplitudeValues.uuid);
        if (characteristic !== undefined) {

            const buf = getBufferFromAmplitudeTasks(taskList)
            writeAmplitudeBufferCharacteristic(buf, characteristic);
        }
    }
}

//shared/types/tactonTypes.ts
export interface TactileTask {
 channelIds: number[],
 intensity: number,
}
```

To use the API from the rendering process of electron, use IPC calls

```Typescript
   window.api.send(IPC_CHANNELS.bluetooth.main.writeAmplitudeBuffer, {
    deviceId: this.actuators[0].deviceUuid,
    taskList: [
     {
      channelIds: [this.actuators[0].actuator],
      intensity: leftIntensity,
     },
    ]
   });
```

## Firmware (TODO)

## Inter Component Communication (TODO)

### Websocket Communication between Server and Client (Shared Library)

### Tactile Display  (Client - Firmware)

#### BLE Protocol

#### MIDI Implementation
