# AmberScript Transcription API
You will receive an apiKey from us, which you will have to provide as query parameter `apiKey` for all endpoints below. After uploading the file using endpoint 1), our transcribers or our automatic speech recognition servers (depending on the jobType) will get to work and create a transcription. `direct` transcriptions are usually ready within 1h, `perfect` ones within 5 business days. You can query the `status` of the job using endpoint 2). When `status` changed to `DONE` the job is finished and you can download the file in json or xml format using endpoint 3).

Possible `status` values: "OPEN", "ERROR", "DONE".

## Table Of Contents
- [Uploading a file](#uploading-a-file)
- [Getting the status of a transcription](#getting-the-status-of-a-transcription)
- [Exporting Files](#exporting-files)
  - [Exporting a finished file (DEPRECATED)](#exporting-a-finished-file)
  - [Export to STL](#export-to-stl)
  - [Export to SRT](#export-to-srt)
  - [Export to VTT](#export-to-vtt)
- [Additional Information](#additional-information)
- [Support](#support)

## Uploading a file
`POST /jobs/upload-media`

Supported parameters:
- `language`: [`nl`, `en`, `de`, `fr`, `da`, `sv`, `fi`, `no`, `es`] 
- `transcriptionType`: [`transcription`, `translation`]
- `jobType`: [`perfect`, `direct`]
- `numberOfSpeakers`: [`1`, `2`, `3`, `4`, `5`]

File requirements:
- max 4GB
- audio or video file
- supported formats: wav, mp3, m4a, aac, wma, mov, m4v, mp4, opus, ogg, flac

_If you need support for a different file format, please get in touch with us: info (at) amberscript (dot) com_

Returns:
```json
{
  "jobStatus": {    
    "jobId": "{{JOB_ID}}",
    "created": 1553871202831,
    "language": "nl",
    "status": "OPEN",
    "jobType": "direct",
    "nrAudioSeconds": 0,
    "transcriptionType": "transcription",
    "filename": "FILE_NAME"
  }
}
```

### CURL
```shell
curl --request POST --url 'https://qs.amberscript.com/jobs/upload-media?transcriptionType=transcription&jobType=direct&language=nl&apiKey={{YOUR_API_KEY}}' --form file=@./my-file.mp3
```

### NodeJs
```javascript
const request = require('request');
const fs = require('fs');

let options = {
  method: 'POST',
  url: 'https://qs.amberscript.com/jobs/upload-media',
  qs: {
    apiKey: 'YOUR_API_KEY',
    transcriptionType: 'transcription',
    jobType: 'direct',
    language: 'nl',
    numberOfSpeakers: '2'
  },
  headers: {'content-type': 'multipart/form-data'},
  formData: {file: fs.createReadStream("./path/to/my_file.mp3")}
};

request(options, function (error, response, body) {
  if (error) throw new Error(error);

  console.log(body);
});
```

### Python
```python
import requests

url = 'https://qs.amberscript.com/jobs/upload-media'
filepath = '/Users/userA/Downloads/my-file.mp3'
querystring = {"jobType":"direct","language":"nl","transcriptionType":"transcription","apiKey":"YOUR_API_KEY"}
files = {'file': open(filepath, 'rb')}

response = requests.post(url, files=files, verify=False, params=querystring)

print(response.status_code)
print(response.text)
```

## Getting the status of a transcription
`GET /jobs/status`

Supported parameters:
- `jobId`: [`YOUR_JOB_ID`]

Returns:
```json
{
  "jobStatus": {
    "jobId": "{{JOB_ID}}",
    "created": 1553871202831,
    "language": "nl",
    "status": "OPEN",
    "jobType": "perfect",
    "nrAudioSeconds": 0,
    "transcriptionType": "transcription",
    "filename": "FILE_NAME"
  }
}
```

### CURL
```shell
curl --request GET --url 'https://qs.amberscript.com/jobs/status?jobId=JOB_ID&apiKey=YOUR_API_KEY'
```

### NodeJs
```javascript
var request = require("request");

var options = { method: 'GET',
  url: 'https://qs.amberscript.com/jobs/status',
  qs: { jobId: 'JOB_ID', apiKey: 'YOUR_API_KEY' } };

request(options, function (error, response, body) {
  if (error) throw new Error(error);

  console.log(body);
});

```

### Python
```python
import requests

url = "https://qs.amberscript.com/jobs/status"

querystring = {"jobId":"JOB_ID","apiKey":"YOUR_API_KEY"}

payload = ""
response = requests.request("GET", url, data=payload, params=querystring)

print(response.text)
```

### Java
```java
HttpResponse<String> response = Unirest.get("https://qs.amberscript.com/jobs/status?jobId=JOB_ID&apiKey=YOUR_API_KEY")
  .asString();
```

## Exporting Files

### Exporting a finished file
`GET /jobs/export`

|:warning:|DEPRECATED|
|---------|:---------|

Supported parameters:
- `jobId`: [`YOUR_JOB_ID`]
- `format`: [`xml`, `json`, `srt`]
- `maxCharsPerSubtitle`: (optional) any integer. Determines the maximum number of characters per subtitle frame. A subtitle frame has two lines. Default: 50 (srt only)
- `subtitleDurationMax`: (optional) any integer. Determines the max duration (in milliseconds) a single subtitle frame should be shown. Default: 4500 (srt only)

Returns (xml):
```xml
<?xml version='1.0' encoding='UTF-8'?>
<AudioDoc name="{{JOB_ID}}" path="{{FILE_NAME}}">
  <SpeakerList>
    <Speaker spkid="spk1">Speaker 1</Speaker>
    <Speaker spkid="spk2">Speaker 2</Speaker>
    <Speaker spkid="spk3">Speaker 3</Speaker>
  </SpeakerList>
  <SegmentList>
    <SpeechSegment spkid="spk1">
      <word dur="0.5700002" stime="2.11">Goedemiddag</word>
      <word dur="0.32999992" stime="2.68">welkom</word>
      <word dur="0.1500001" stime="3.01">bij</word>
      <word dur="0.17999983" stime="3.16">het</word>
      <word dur="0.5500002" stime="3.37">mondelinge.</word>
    </SpeechSegment>
    <SpeechSegment spkid="spk2">
      <word dur="0.7600002" stime="4.14">Vragen</word>
      <word dur="0.42000008" stime="4.98">uur.</word>
    </SpeechSegment>
  <SegmentList>
</AudioDoc>
```

Returns (json):
```json
{
  "id": "5c9e45057103e464a4c6f477",
  "recordId": "RECORD_ID",
  "speakers": [
    {
      "spkid": "spk1",
      "name": "Speaker 1"
    },
    {
      "spkid": "spk2",
      "name": "Speaker 2"
    },
    {
      "spkid": "spk3",
      "name": "Speaker 3"
    }
  ],
  "segments": [
    {
      "speaker": "spk1",
      "words": [
        {
          "text": "Goedemiddag",
          "start": 2.11,
          "end": 2.68,
          "duration": null,
          "conf": null
        },
        {
          "text": "welkom",
          "start": 2.68,
          "end": 3.01,
          "duration": null,
          "conf": null
        },
        {
          "text": "bij",
          "start": 3.01,
          "end": 3.16,
          "duration": null,
          "conf": null
        },
        {
          "text": "het",
          "start": 3.16,
          "end": 3.34,
          "duration": null,
          "conf": null
        },
        {
          "text": "mondelinge.",
          "start": 3.37,
          "end": 3.92,
          "duration": null,
          "conf": null
        }
      ]
    },
    {
      "speaker": "spk2",
      "words": [
        {
          "text": "Vragen",
          "start": 4.14,
          "end": 4.9,
          "duration": null,
          "conf": null
        },
        {
          "text": "uur.",
          "start": 4.98,
          "end": 5.4,
          "duration": null,
          "conf": null
        }
      ]
    }
  ]
}
```

#### CURL
```shell
curl --request GET --url 'https://qs.amberscript.com/jobs/export?jobId=JOB_ID&apiKey=YOUR_API_KEY&format=json'
 ```
 
 ### NodeJS
 ```javascript
 var request = require("request");

var options = { method: 'GET',
  url: 'https://qs.amberscript.com/jobs/export',
  qs: { jobId: 'JOB_ID', apiKey: 'YOUR_API_KEY', format: 'json' } };

request(options, function (error, response, body) {
  if (error) throw new Error(error);

  console.log(body);
});
```

#### Python
```python
import requests

url = "https://qs.amberscript.com/jobs/export"

querystring = {"jobId":"JOB_ID","apiKey":"YOUR_API_KEY","format":"json"}

payload = ""
response = requests.request("GET", url, data=payload, params=querystring)

print(response.text)
```

#### Java
```java
HttpResponse<String> response = Unirest.get("https://qs.amberscript.com/jobs/export?jobId=JOB_ID&apiKey=YOUR_API_KEY&format=json")
  .asString();
```

### Export to STL
`GET /jobs/export-stl`

Supported parameters:
- `jobId`: [`YOUR_JOB_ID`]
- `maxNumberOfRows`: (optional) any integer. Sets the maximum number of rows for each subtitle.
- `maxScreenTimePerRowSeconds`: (optional) any float. Sets the maximum number seconds given for each row of subtitles.

Returns (json):
```json
{
    "downloadUrl": "https://PROD-SERVER/ebu-stl/FILENAME?X-Amz-Algorithm=HASH_ALGORITHM&X-Amz-Date=CURRENT_DATE&X-Amz-SignedHeaders=host&X-Amz-Expires=TIME_TO_EXPIRATION&X-Amz-Credential=AMZ_CREDENTIAL&X-Amz-Signature=AMZ_SIGNATURE"
}
```

#### CURL
```shell
curl --request GET --url 'https://qs.amberscript.com/jobs/export-stl?jobId=JOB_ID&apiKey=YOUR_API_KEY'
 ```

 #### NodeJS
 ```javascript
 var request = require("request");

var options = { method: 'GET',
  url: 'https://qs.amberscript.com/jobs/export-stl',
  qs: { jobId: 'JOB_ID', apiKey: 'YOUR_API_KEY' };

request(options, function (error, response, body) {
  if (error) throw new Error(error);

  console.log(body);
});
```

#### Python
```python
import requests

url = "https://qs.amberscript.com/jobs/export-stl"

querystring = {"jobId":"JOB_ID" "apiKey":"YOUR_API_KEY"}

payload = ""
response = requests.request("GET", url, data=payload, params=querystring)

print(response.text)
```

#### Java
```java
HttpResponse<String> response = Unirest.get("https://qs.amberscript.com/jobs/export-stl?jobId=JOB_ID&apiKey=YOUR_API_KEY")
  .asString();
```

### Export to SRT
`GET /jobs/export-srt`

Supported parameters:
- `jobId`: [`YOUR_JOB_ID`]
- `maxCharsPerRow`: (optional) any integer. Sets the maximum number of characters per row.
- `maxNumberOfRows`: (optional) any integer. Sets the maximum number of rows for each subtitle.
- `maxScreenTimePerRowSeconds`: (optional) any float. Sets the maximum number seconds given for each row of subtitles.

Returns (srt):
```
1
00:00:00,000 --> 00:00:02,190
Goedemiddag, welkom.

2
00:00:02,289 --> 00:00:05,369
Goedemiddag, dank u wel.

3
00:00:05,469 --> 00:00:08,289
Hoe gaat het met je vandaag

4
00:00:09,159 --> 00:00:12,689
Goed, dank je!
```

#### CURL
```shell
curl --request GET --url 'https://qs.amberscript.com/jobs/export-srt?jobId=JOB_ID&apiKey=YOUR_API_KEY'
 ```

 #### NodeJS
 ```javascript
 var request = require("request");

var options = { method: 'GET',
  url: 'https://qs.amberscript.com/jobs/export-srt',
  qs: { jobId: 'JOB_ID', apiKey: 'YOUR_API_KEY' };

request(options, function (error, response, body) {
  if (error) throw new Error(error);

  console.log(body);
});
```

#### Python
```python
import requests

url = "https://qs.amberscript.com/jobs/export-srt"

querystring = {"jobId":"JOB_ID" "apiKey":"YOUR_API_KEY"}

payload = ""
response = requests.request("GET", url, data=payload, params=querystring)

print(response.text)
```

#### Java
```java
HttpResponse<String> response = Unirest.get("https://qs.amberscript.com/jobs/export-srt?jobId=JOB_ID&apiKey=YOUR_API_KEY")
  .asString();
```

### Export to VTT
`GET /jobs/export-vtt`

Supported parameters:
- `jobId`: [`YOUR_JOB_ID`]
- `maxCharsPerRow`: (optional) any integer. Sets the maximum number of characters per row.
- `maxNumberOfRows`: (optional) any integer. Sets the maximum number of rows for each subtitle.
- `maxScreenTimePerRowSeconds`: (optional) any float. Sets the maximum number seconds given for each row of subtitles.

Returns (vtt):
```
WEBVTT - created by www.amberscript.com

1
00:00:00.000 --> 00:00:02.190
Goedemiddag, welkom.

2
00:00:02.289 --> 00:00:05.369
Goedemiddag, dank u wel.

3
00:00:05.469 --> 00:00:08.289
Hoe gaat het met je vandaag?

4
00:00:09.159 --> 00:00:12.689
Goed, dank je!
```

#### CURL
```shell
curl --request GET --url 'https://qs.amberscript.com/jobs/export-vtt?jobId=JOB_ID&apiKey=YOUR_API_KEY'
 ```

 #### NodeJS
 ```javascript
 var request = require("request");

var options = { method: 'GET',
  url: 'https://qs.amberscript.com/jobs/export-vtt',
  qs: { jobId: 'JOB_ID', apiKey: 'YOUR_API_KEY' };

request(options, function (error, response, body) {
  if (error) throw new Error(error);

  console.log(body);
});
```

#### Python
```python
import requests

url = "https://qs.amberscript.com/jobs/export-vtt"

querystring = {"jobId":"JOB_ID" "apiKey":"YOUR_API_KEY"}

payload = ""
response = requests.request("GET", url, data=payload, params=querystring)

print(response.text)
```

#### Java
```java
HttpResponse<String> response = Unirest.get("https://qs.amberscript.com/jobs/export-vtt?jobId=JOB_ID&apiKey=YOUR_API_KEY")
  .asString();
```
---
## Additional Information

### Callbacks

#### What happens if a callback fails?

- We track the callback in our history log.
- After **1 hour**, we make a new attempt to update the client.
- If the attempt is unsuccessful we retry the update again.
- The maximum number of attempts is **10**.

#### Storing the `jobId`
You have to store the `jobId` if you don't use any callbacks.

## Support
If you need any technical assistance, feel free to contact `info (at) amberscript (dot) com`
