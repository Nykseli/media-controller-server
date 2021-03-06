# Mediacontroller Server

Server part of the remote media controller see the client [here](https://github.com/Nykseli/android-media-controller)

## How to run

```
git clone https://github.com/Nykseli/media-controller-server
cd media-controller-server
python3 src/controller.py
```

## Config

You can put the [config.json](https://github.com/Nykseli/media-controller-server/blob/master/config.json.example) file to the project root or in ~/.MediaControllerClient/config.json

### Config structure

| Key | Type | Description |
|-----| -----| ------      |
| crypto | Object | Contains config of the security settings|
| crypto.secretKey | String | Ascii string with length of 16 that is used to encrypt and decrypt messages |
| general | Object | Contains config of the general program settings|
| general.display | String | DISPLAY enviroment variable user by X11. Needs to be set if program is used as a daemon |
| vlc |Object| Contains vlc configs|
| vlc.allowedFilePaths | String[] | Contains list of allowed paths that can browsed by client ( currently Android client supports only one path) |
| vlc.allowedFileTypes | String[] | Contains list of allowed filetypes that can be browsed by client|
| vlc.commandlineArguments | String[] | Contains list of extra arguments that you want to use. E.g. --fullscreen|
| websocket | Object | Contains web socket settings |
| websocket.allowedOrigins | string[] | Define what origins are allowed to connect |
| websocket.port | int | Define what port websocket listens |


## Security

There is option for using 128-bit AES (CFB) encryption. Enable this by setting secret key in [config](#Config).

### Receiving data from server
Encrypted messages are encoded in Base64 and send as byte array.
<br />
After Base64 decode the first 16 bytes contains the IV.
<br />
After decrypting the message, you find that the padding is done with '0'. character

### Sending data do server
When sending data to server send it as byte data since plain text is interpreted as unencrypted.
<br />
Message needs to be Base64 encoded. Base64 decoded messages first 16 bytes needs to contain the IV.
<br />
Decrypted message padding needs to be done using '0' character


## Websocket API

Controlling divice with client works by sending text in json format to server.
Basic structure is:
```
{
    "interface": <Interface that command uses e.g. vlc>,
    "command": <Command string>,
    "additionalInfo": {
        <Object that contains additional info that the command can use>
    }
}
```

### Interfaces

Interfaces are as follows:

* audio
  * Used to control system audio
* config
  * Used to get configuration from server
* general
  * Used to control system settings e.g. volume, mouse and keyboard input, etc
* keyboard
  * Used to simulate keyboard inputs. Supports utf-8
* mouse
  * Used to simulate mouse functionality. e.g. movement and clicks

* vlc
  * Used to control Vlc mediaplayer

### Api doc

Currently following commands are implemented

|Interface|Command|Additional Info| Description|
| ------- | ----- | ----------- | ---------- |
| audio | decreaseMasterVolume | - | Decrease system volume |
| audio | increaseMasterVolume | - | Increase system volume |
| audio | muteMasterVolume | - | Mute system volume |
| config  | getConfig | - | Get config.json contents. <br /> Response format: ```{"config": <object from config.json>```
| general | getFilesAndFolders | "absolutePath": string | Get files and folders in ***absolutePath***. <br /> Response format: ```{"files": string[], "folders": string[], "currentPath": absolutePath}```|
| keyboard | inputString | "input": string | Input string where the cursor currently is. String can contain utf-8 characters |
| keyboard | pressEnter | - | Simulate enter keypress |
| keyboard | pressTab | - | Simulate tab keypress |
| keyboard | pressBackSpace | - | Simulate BackSpace keypress |
| keyboard | pressArrowUp | - | Simulate Arrow Up keypress |
| keyboard | pressArrowRight | - | Simulate Arrow Right keypress |
| keyboard | pressArrowDown | - | Simulate Arrow Down keypress |
| keyboard | pressArrowLeft | - | Simulate Arrow Left keypress |
| mouse | moveMouseX | "amount": int | Moves mouse on x axis by x amount that is defined by ***amount***. ***amount*** can be negative. |
| mouse | moveMouseY | "amount": int | Moves mouse on y axis by x amount that is defined by ***amount***. ***amount*** can be negative. |
| mouse | leftMouseClick | - | Click with left mouse button |
| mouse | setMousePosition | "x": int,  <br />"y": int | Set mouse position to ***x***,***y*** coordinate |
| vlc | pauseFile | - | Toggle vlc pause on/off |
| vlc | playFile | "absolutePath": string | Play file defined by ***absolutePath***|
| vlc | playFiles | "absolutePaths": string[] | Add files in ***absolutePaths*** list to vlc medialist and play them|
| vlc | playNextMedia | - | Play next media in vlc medialist|
| vlc | playPreviousMedia | - | Play previous media in vlc medialist|
| vlc | stopMedia | - | Stop current medialist |
| vlc | increaseVolume | - | Increace Vlc volume by one step (5%) |
| vlc | decreaseVolume | - | Decreace Vlc volume by one step (5%) |
| vlc | muteVolume | - | Toggle Vlc mute on/off |
| vlc | fastForward | - | Fast forward 10 seconds |
| vlc | rewind | - | Rewind 10 secods |
| vlc | cycleAudioTrack | - | Cycle through audio tracks |
| vlc | cycleSubtitleTrack | - | Cycle through subtitle tracks |
| vlc | getCurrentlyPlaying | - | Get title of currently playing file. <br/> Response format: ```{"interface": "vlc", "messageData": {"currentlyPlaying": string }}```. <br /> ***currentlyPlaying*** is empty if medialist is empty. |
