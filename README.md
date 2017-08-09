UNMAINTAINED
============
This project is not maintained anymore.
If you want to take over contact us at tech@cargomedia.ch.

janus-gateway-audioroom
=======================
[janus-gateway](https://github.com/meetecho/janus-gateway) custom plugin.

[![Build Status](https://travis-ci.org/cargomedia/janus-gateway-audioroom.svg)](https://travis-ci.org/cargomedia/janus-gateway-audioroom)

Overview
--------
This plugin is based on native `janus/audiobridge` plugin and adds additional functionality. 

Main extensions:
- renames plugin with `janus.plugin.cm.audioroom`
- renames identification field called `room` with `id`
- changes type of room `id` from `integer` to `string`
- automatically records the audio `room` into configurable archives
- creates job files and store events like new `archive-finished`
- drops support for `create` endpoint
- creates room transparently for `join` and `changeroom` endpoint
- removes `room` if last participant leaves (for session destroy and `changeroom`)
- allows to disable mixer pre-buffering for audio broadcast scenario

Configuration
-------------
```
[general]
; NOTE: all paths should exist beforehead

; Path for job JSONs
; job_path = /tmp/jobs

; prinf pattern for job filenames (.json is auto)
; Short usage, the following gets substituted:
; #{time}     is timestamp (guint64)
; #{rand}     is random integer (guin32)
; #{md5}      is md5 of (timestamp + plugin name + random integer)
; #{plugin}   is plugin name ("janus.plugin.cm.rtpbroadcast")
; job_pattern = job-#{md5}

; Path for recording and thumbnailing
; archive_path = /tmp/recordings

; printf pattern for recordings filenames
; Short usage, the following gets substituted:
; #{id}       is streamChannelKey (string)
; #{time}     is timestamp (guint64)
; #{type}     is type ("audio", "video" or "thumb" string)
; recording_pattern = rec-#{id}-#{time}-#{type}

; The mixer pre-buffering allows to define the time window of audio
; RTP source to be queued before it is mixed with another RTP sources.
; By default is set to 6 packets what gives 240ms of time tolerance for
; incoming packets. If set to 0 then pre-buffering is disabled, as well as
; dropping of outdated packets is disabled.
; mixer_prebuffering = 6
```

Synchronous actions
-------------------
It supports `destroy`, `list`, `exists`, `listparticipants`, `resetdecoder` actions like native `janus/audiobridge` plugins with 
change that the `id` is of type `string`.

#### `create`
It drops support for `create` and introduce creation of the room with asynchronous action `join` and `changeroom`. It records all `rooms` by default. 
The `room` is created with `sampling` of `48000` by default.

#### `list`
It responses with list of current rooms.

**Response**:
```json
[
    {
        "id": "<string>",
        "uid": "<string>",
        "sampling_rate": "<int>",
        "record": "<boolean>",
        "num_participants": "<int>",
        "description": "<string>"
    }
]
```

Asychronous actions
-------------------
It supports `join`, `configure`, `changeroom`, `leave` actions like native `janus/audiobridge` plugins with change that the `id` is of type `string`.

#### `join`
It creates `room` if does not exist. The `room` gets default values with `sampling` of `48000`. 

**Request**:
```json
{
    "request": "join",
    "id": "<string>"
}
```

**Response**:
```json
{
    "audioroom": "joined",
    "id": "<string>",
    "uid": "<string>",
    "userid": "<int>",
    "participants": []
}
```

#### `changeroom`
It creates `room` if does not exist. The `room` gets default values with `sampling` of `48000`. The `oldroom` is automatically removed if gets 
empty (no more participants) after change.

**Request**:
```json
{
    "request": "changeroom",
    "id": "<string>"
}
```

**Response**:
```json
{
    "audioroom": "roomchanged",
    "id": "<string>",
    "uid": "<string>",
    "userid": "<int>",
    "participants": []
}
```

Job files
---------
It creates configurable `job-files` with plugin events. It support currently for `archive-finished` event.

##### `archive-finished` 
```json
{
    "data": {
        "id": "<string>",
        "uid": "<string>",
        "audio": "<archive_path/recording_pattern>.wav"
    },
    "plugin": "janus.plugin.cm.audioroom",
    "event": "archive-finished"
}
```

The content of dumped file is of type `WAV`.

Testing
-------
There is a simple testing script placed in the `test/tester.py` which allow for triggering basic actions on the plugin. Please find the 
[test/README](test/README.md) for more details.

Building
--------
If you got janus-gateway-audioroom from the git repository, you will first need to run the included `autogen.sh` script 
to generate the `configure` script.

```
./autogen.sh
./configure  --prefix=/opt/janus
make
make install
```
