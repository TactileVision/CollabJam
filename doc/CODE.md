
## Server (TODO)

The Collabjam-Server is a NodeJS application that communicates with Collabjam-Clients through websocket connections. It has three responsibilities: (1) Propagating the available rooms a client can join, (2) responding to the actions a client executes when jamming  by broadcasting the appropriate answer to all clients in the same room and (3) recording and sharing tactons based on the previously mentioned jamming. The most important  components of Collabjam-Server are [express.js](https://expressjs.com) and the [*ws* websocket implementation](https://github.com/websockets/ws?tab=readme-ov-file).  

- Server to multiple clients communication

### Connecting

Upon receiving a http request from a client, server and client upgrade to a websocket connection[^2].

The server shares the available rooms based on requests from a client.

```mermaid
sequenceDiagram
    participant s as Server
    participant c1 as Client 1
    participant c2 as Client n
    actor u1 as User
    u1->>c1: Request available Rooms
    c1->>s: GET_AVAILABLE_ROOMS_SERV
    s->>c1: GET_AVAILABLE_ROOMS_CLI
    u1->>c1: User chooses room
    c1->>s: ENTER_ROOM_SERV
    Note left of s: Server creates tacton recording<br>if the user is the first in the room
    s->>s: enterUserInRoom()
    s->>c1: ENTER_ROOM_CLI
    par Broadcast updated room information to all clients
      s->>c1: UPDATE_ROOM_CLI
    and 
      s->>c2: UPDATE_ROOM_CLI
    end

```

### Jamming

It is also responsible for broadcasting jamming inputs and control commands from one client to all in the room.
Additionally, the server tracks the recording time of a room and ends it after a specific amount of time, that can be configured in the json definition of the room.

```mermaid
sequenceDiagram
    participant s as Server
    participant c1 as Client 1
    participant c2 as Client n

    Note left of s: Processing on client side<br>omitted in this<br>sequence diagram
    actor u1 as User 1
    actor u2 as User 2
    u1->>c1: Presses button on gamepad
    c1->>s: SEND_INSTRUCTION_SERV
    s->>s: processInstructionsFromClient()
    par Broadcast
      s->>c1: SEND_INSTRUCTION_CLI
      c1->>u1: Vibration output
    and 
      s->>c2: SEND_INSTRUCTION_CLI
      c2->>u2: Vibration output
    end
```





[^2]:Learn how websocket connection are established: <https://sookocheff.com/post/networking/how-do-websockets-work/>>

️#### Playing back recorded tactons

### Data Structures

#### Room Configuration

- Rooms are preconfigured at the moment, to add or remove a room, edit `src/util/DefaultRooms.ts` file, where an array of rooms is stored.
- After that the server has to be built again

```typescript
{
  currentRecordingTime: 0,
  participants: [],
  description: "",
  id: "9cbb9a45-b03c-4693-bf05-9cd47806ebca", // EDIT: Insert a new uuid v4 here
  maxDurationRecord: 20000, //EDIT:  The maximal length a of tacton for this room in milliseconds 
  mode: InteractionMode.Jamming,
  name: "Amsterdam", //EDIT: The name that will be used in the UI 
  recordingNamePrefix: "amsterdam" //EDIT: Prefix for the tactons stored as json files
}
```

#### Tacton JSON
⚠️
The recordings created are stored on a per room basis inside folders as json files under `userData/<uuid>/<recordingNamePrefix.json>`. It includes metadata and the tacton data.
The tacton-format is a set of `setParameter` and `wait`instructions. [^3]
A `setParameter` instruction is instantly executed and changes the amplitude (later the frequency as well ) of one or more actuators.
The `wait` instructions specifies an amount of time of milliseconds before the next instruction will be executed.
This structure leads to a chain of `setParameter` and `wait` instructions.

```json
{
	"uuid":"183e6707-21ee-44e6-af19-ab3fd836a714",
	"metadata":{
		"name":"eindhoven-3",
		"favorite":false,
		"recordDate":"2023-06-22T12:30:25.131Z"},
    "instructions":[
		{"setParameter":{"channelIds":[0],"intensity":0.6}},
		{"wait":{"miliseconds":80}},
		{"setParameter":{"channelIds":[1],"intensity":0.6}},
		{"wait":{"miliseconds":84}},
		{"setParameter":{"channelIds":[2],"intensity":0.6}},
		{"wait":{"miliseconds":55}},
		{"setParameter":{"channelIds":[0],"intensity":0}},
		{"wait":{"miliseconds":20}},
		{"setParameter":{"channelIds":[1],"intensity":0}},
		{"wait":{"miliseconds":101}},
		{"setParameter":{"channelIds":[0,1],"intensity":0.6}},
		{"setParameter":{"channelIds":[2],"intensity":0}},
		{"wait":{"miliseconds":100}},
		{"setParameter":{"channelIds":[2],"intensity":0.6}},
		{"wait":{"miliseconds":29}},
		{"setParameter":{"channelIds":[0],"intensity":0}},
		{"wait":{"miliseconds":16}},
		{"setParameter":{"channelIds":[1],"intensity":0}},
		{"wait":{"miliseconds":54}},
		{"setParameter":{"channelIds":[0,1],"intensity":0.6}},
		{"wait":{"miliseconds":141}},
		{"setParameter":{"channelIds":[0,2,1],"intensity":0}},
		{"setParameter":{"intensity":0,"channelIds":[0,1,2]}}
	]
}
```


[^3]: The format is basically a subset of the vtproto protocol buffer implementation devleoped by lhinderberger. See `doc/vtproto.proto`


## Client (TODO)

<!-- ### Application Information -->
The Collabjam-Client is an Electron application and development requires basic knowledge about:

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

The client is split into frontend(renderer process) and backend(main process). The frontend contains components that, outside of Electron, would run in the browser, the backend includes the parts that would run on the webserver.
In addition to being an Electron app, Collabjam-Client is build with [Vue.js](https://vuejs.org). [Vue CLI Plugin Electron Builder](https://nklayman.github.io/vue-cli-plugin-electron-builder/) is used to build the app, instead of the vanilla [Electron builder](https://www.electron.build).

Relevant Files for working with the Vue+Electron structure are:
| File          | Purpose                                                                                                    |
| ------------- | ---------------------------------------------------------------------------------------------------------- |
| main.js       | Entry point for the renderer part of the electron app. Initialization of Vue, Vue store and input handling |
| App.vue       | Contains the basic layout of the application. All other views are inserted into this View                  |
| background.ts | The file where the application window gets configured(resolution, title, ...) and instantiated             |
| preload.js    | Defines which functions are exposed between renderer and main process (see IPC, further reading)           |

#### IPC

To communicate between frontend/renderer and backend/os level components, Electron uses inter-process communication. [^1]

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

- Strings define the different message types in IPC. The messages for the Collabjam-Client are defined in the `[core/IPC/IpcChannels.ts]` file
- (⚠️WIP) The messages are structured in the `function.target.message` scheme
  - e.g. `bluetooth.main.writeAmplitudeBuffer` is a message coming from the renderer process targeting to the bluetooth functionality of the main process
  - e.g. `bluetooth.renderer.connectedToDevice` is a message from the main process, informing the renderer process about a newly connected device

### Folder Structure

- The folder structure of collabjam-client groups the code into features and groups them into three categories:  
  - **App** contains the infrastructure components of the application:
    - Routing
    - Vue Store
    - The fundamental Vue Views defining the always available UI  
      - **Root View** All other views are injected into the root view
      - Sidebar
  - **Core** is made up of the base functionality of the CollabJam client application
    - IPC
    - Managing BLE Connections
    - A basic API for tactile displays
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

On top of the `core/Ble` stack, `core/TactileDisplays/main/TactileDisplayCharacteristicWriter` exposes  functions to interface with the tactile dislay on a application level. Currently, the amplitude and the frequency of a tactile display can be changed.

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
