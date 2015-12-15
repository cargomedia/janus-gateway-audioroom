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
```

Synchronous actions
-------------------
It supports `destroy`, `list`, `exists`, `listparticipants`, `resetdecoder` actions like native `janus/audiobridge` plugins with 
change that the `id` is of type `string`.

#### `create`
It additionally drops support for `record` and `record_file` flag in the favour of recording all `rooms` by default.

**Request**:
```json
{
  "id": "<string>",
  "description": "<string>",
  "sampling": "<int>",
}
```

**Response**:
It responses with default status. 

```json
{
	"audioroom" : "created",
	"id": "<string>"
}
```

#### `list`
It responses with list of current rooms.

**Response**:
```
[
    {
        "id": "<string>",
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

Job files
---------
It creates configurable `job-files` with plugin events. It support currently for `archive-finished` event.

##### `archive-finished` 
```json
{
    "data": {
        "id": "<string>",
        "audio": "<archive_path/recording_pattern>"
    },
    "plugin": "janus.plugin.cm.audioroom",
    "event": "archive-finished"
}
```

The content of dumped file is of type `WAV`.

Testing
-------
There is simple testing script placed in `test/tester.py` which allow for triggering basic actions on the plugin. Please find the 
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
