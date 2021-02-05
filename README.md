| :warning: WARNING          |
|:---------------------------|
| <h3><b>We've moved our API documentation and you can find the new version [here](https://amberscript.github.io/api-docs/#introduction).</b></h3>|

# AmberScript Transcription API


You will receive an apiKey from us, which you will have to provide as query parameter `apiKey` for all endpoints below. After uploading the file using endpoint 1), our transcribers or our automatic speech recognition servers (depending on the jobType) will get to work and create a transcription. `direct` transcriptions are usually ready within 1h, `perfect` ones within 5 business days. You can query the `status` of the job using endpoint 2). When `status` changed to `DONE` the job is finished and you can download the file in json or xml format using endpoint 3).

Possible `status` values: "OPEN", "ERROR", "DONE".

## Table Of Contents
- [Uploading a file](#uploading-a-file)
  - [How To Upload Files](#how-to-upload-files)
- [Getting the status of a transcription](#getting-the-status-of-a-transcription)
- [Exporting Files](#exporting-files)
  - [Exporting a finished file (DEPRECATED)](#exporting-a-finished-file)
  - [Export to STL](#export-to-stl)
  - [Export to SRT](#export-to-srt)
  - [Export to VTT](#export-to-vtt)
  - [Export to TEXT](#export-to-text)
  - [Export to JSON](#export-to-json)
- [Delete a job](#delete-a-job)
- [Get list of jobs](#get-list-of-jobs)
- [Support](#support)

---
## Uploading a file
`POST /jobs/upload-media`

Supported parameters:
- `language`: [`nl`, `en`, `de`, `fr`, `da`, `sv`, `fi`, `no`, `es`] 
- `transcriptionType`: [`transcription`, `translation`]
- `jobType`: [`perfect`, `direct`]
- `numberOfSpeakers`: [`1`, `2`, `3`, `4`, `5`]
- `callbackUrl`: (optional) `YOUR_CALLBACK_URL`

### How To Upload Files

#### **With `callbackUrl`**
1. Make a request with the `callbackUrl` specified. We'll send the final status of your upload request to this url.
2. When we finish processing, we'll send a `POST` request to your `callbackUrl` with the `status` of the upload.
   - `status` can either be `DONE` or `ERROR`.
3. Your `callbackUrl` should respond with any `2xx` if you successfully receive the `status` update.

Example of `POST` request to your `callbackUrl`:
```json
{
  "jobStatus": {    
    "jobId": "{{JOB_ID}}",
    "created": 1553871202831,
    "language": "nl",
    "status": "DONE",
    "jobType": "direct",
    "nrAudioSeconds": 0,
    "transcriptionType": "transcription",
    "filename": "FILE_NAME"
  }
}
```

What if a callback fails?
- When a callback fails, we retry sending the status update every hour.
- The maximum number of retry attempts is `10`.

#### **Without `callbackUrl`**
1. Make a request without the `callbackUrl` specified.
2. Store the value of the `jobId` that is returned with a successful call to this endpoint.
3. Use the `jobId` to periodically check the status of the upload request (e.g. every 5 mins).
   - You can check the status with the [`GET /jobs/status`](#getting-the-status-of-a-transcription) endpoint.
   - `status` can either be `OPEN`, `DONE` or `ERROR`.

### File requirements:
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

---
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

---
## Exporting Files

### Exporting a finished file
`GET /jobs/export`

|:no_entry:|DEPRECATED|
|----------|:---------|

Supported parameters:
- `jobId`: [`YOUR_JOB_ID`]
- `format`: [`xml`, `json`, `srt`]
- `maxCharsPerSubtitle`: (optional) integer (default: `50`). Determines the maximum number of characters per subtitle frame. A subtitle frame has two lines. (srt only)
- `subtitleDurationMax`: (optional) integer [default: `4500`). Determines the max duration (in milliseconds) a single subtitle frame should be shown. (srt only)

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
- `maxNumberOfRows`: (optional) integer between `1` and `2` (default: `2`). Sets the maximum number of rows for each subtitle.
- `maxScreenTimePerRowSeconds`: (optional) float (default: `2`). Sets the maximum number of seconds given for each row of subtitles.

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
- `maxCharsPerRow`: (optional) integer between `30` and `45` (default: `42`). Sets the maximum number of characters per row.
- `maxNumberOfRows`: (optional) integer between `1` and `2` (default: `2`). Sets the maximum number of rows for each subtitle.
- `maxScreenTimePerRowSeconds`: (optional) float (default: `2`). Sets the maximum number of seconds given for each row of subtitles.

Returns (srt):
```
1
00:00:00,449 --> 00:00:03,769
Hi, welcome to Amber script, in this short

2
00:00:03,869 --> 00:00:07,400
video, we would like to show you how
you can use Amber script at its full

3
00:00:07,500 --> 00:00:10,189
potential. The first
function we would like to show

4
00:00:10,289 --> 00:00:13,639
you is the edit function.
This allows you to edit errors
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
- `maxCharsPerRow`: (optional) integer between `30` and `45` (default: `42`). Sets the maximum number of characters per row.
- `maxNumberOfRows`: (optional) integer between `1` and `2` (default: `2`). Sets the maximum number of rows for each subtitle.
- `maxScreenTimePerRowSeconds`: (optional) float (default: `2`). Sets the maximum number of seconds given for each row of subtitles.

Returns (vtt):
```
WEBVTT - created by www.amberscript.com

1
00:00:00.449 --> 00:00:03.769
Hi, welcome to Amber script, in this short

2
00:00:03.869 --> 00:00:07.400
video, we would like to show you how
you can use Amber script at its full

3
00:00:07.500 --> 00:00:10.189
potential. The first
function we would like to show

4
00:00:10.289 --> 00:00:13.639
you is the edit function.
This allows you to edit errors
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

### Export to TEXT
`GET /jobs/export-txt`

Supported parameters:
- `jobId`: [`YOUR_JOB_ID`]
- `includeTimestamps`: (optional) boolean (default: `true`).
- `includeSpeakers`: (optional) boolean (default: `true`).
- `highlightsOnly`: (optional) boolean (default: `false`).
- `maxCharsPerRow`: (optional) integer (default: `no default`). Sets the maximum number of characters per row.

Returns (txt):
```
00:00:00
Speaker 1: Hi, welcome to Amber script, in this short video, we would like to show you how you can use Amber script at its full potential. The first function we would like to show you is the edit function. This allows you to edit errors
```

#### CURL
```shell
curl --request GET --url 'https://qs.amberscript.com/jobs/export-txt?jobId=JOB_ID&apiKey=YOUR_API_KEY'
 ```

 #### NodeJS
 ```javascript
 var request = require("request");

var options = { method: 'GET',
  url: 'https://qs.amberscript.com/jobs/export-txt',
  qs: { jobId: 'JOB_ID', apiKey: 'YOUR_API_KEY' };

request(options, function (error, response, body) {
  if (error) throw new Error(error);

  console.log(body);
});
```

#### Python
```python
import requests

url = "https://qs.amberscript.com/jobs/export-txt"

querystring = {"jobId":"JOB_ID" "apiKey":"YOUR_API_KEY"}

payload = ""
response = requests.request("GET", url, data=payload, params=querystring)

print(response.text)
```

#### Java
```java
HttpResponse<String> response = Unirest.get("https://qs.amberscript.com/jobs/export-txt?jobId=JOB_ID&apiKey=YOUR_API_KEY")
  .asString();
```

### Export to JSON
`GET /jobs/export-json`

Supported parameters:
- `jobId`: [`YOUR_JOB_ID`]

Returns (json):
```json
{
  "id": "5c9e45057103e464a4c6f477",
  "recordId": "RECORD_ID",
  "filename": "FILENAME",
  "startTimeOffset": 0.0,
  "speakers": [
    {
      "spkid": "spk1",
      "name": "Speaker 1"
    }
  ],
  "segments": [
    {
      "speaker": "spk1",
      "words": [
        {
          "start": 0.45,
          "end": 1.08,
          "duration": 0.63000005,
          "text": "Hi,",
          "conf": 1.0,
          "pristine": true
        },
        {
          "start": 1.11,
          "end": 1.65,
          "duration": 0.53999996,
          "text": "welcome",
          "conf": 1.0,
          "pristine": true
        },
        {
          "start": 1.65,
          "end": 1.8,
          "duration": 0.14999998,
          "text": "to",
          "conf": 1.0,
          "pristine": true
        },
        {
          "start": 1.8,
          "end": 2.1,
          "duration": 0.29999995,
          "text": "AmberScript",
          "conf": 0.65,
          "pristine": true
        }
      ]
    }
  ],
  "highlights": []
}
```

#### CURL
```shell
curl --request GET --url 'https://qs.amberscript.com/jobs/export-json?jobId=JOB_ID&apiKey=YOUR_API_KEY'
```

 #### NodeJS
 ```javascript
 var request = require("request");

var options = { method: 'GET',
  url: 'https://qs.amberscript.com/jobs/export-json',
  qs: { jobId: 'JOB_ID', apiKey: 'YOUR_API_KEY' };

request(options, function (error, response, body) {
  if (error) throw new Error(error);

  console.log(body);
});
```

#### Python
```python
import requests

url = "https://qs.amberscript.com/jobs/export-json"

querystring = {"jobId":"JOB_ID" "apiKey":"YOUR_API_KEY"}

payload = ""
response = requests.request("GET", url, data=payload, params=querystring)

print(response.text)
```

#### Java
```java
HttpResponse<String> response = Unirest.get("https://qs.amberscript.com/jobs/export-json?jobId=JOB_ID&apiKey=YOUR_API_KEY")
  .asString();
```

---
## Delete a job
`DELETE /jobs`

Supported parameters:
- `jobId`: [`YOUR_JOB_ID`]

Returns (json):
```json
{
    "message": "Job deleted"
}
```

### CURL
```shell
curl --request DELETE --url 'https://qs.amberscript.com/jobs?jobId=JOB_ID&apiKey=YOUR_API_KEY'
 ```

### NodeJS
```javascript
 var request = require("request");

var options = { method: 'DELETE',
  url: 'https://qs.amberscript.com/jobs',
  qs: { jobId: 'JOB_ID', apiKey: 'YOUR_API_KEY' };

request(options, function (error, response, body) {
  if (error) throw new Error(error);

  console.log(body);
});
```

### Python
```python
import requests

url = "https://qs.amberscript.com/jobs"

querystring = {"jobId":"JOB_ID" "apiKey":"YOUR_API_KEY"}

payload = ""
response = requests.request("DELETE", url, data=payload, params=querystring)

print(response.text)
```

### Java
```java
HttpResponse<String> response = Unirest.delete("https://qs.amberscript.com/jobs?jobId=JOB_ID&apiKey=YOUR_API_KEY")
  .asString();
```

---
## Get list of jobs
`GET /jobs`

Supported parameters:
- `jobId`: (optional). [`YOUR_JOB_ID`]
- `jobType`: (optional). Type of the job e.g. `perfect`
- `status`: (optional). `OPEN`, `ERROR` or `DONE`.
- `transcriptionType`: (optional). Type of transcription e.g. `transcription`.
- `page`: (optional) integer (default: `0`). Page to be retrieved.
- `pageSize`: (optional) integer (default: `20`, maximum: `100`). Number of records to be retrieved for each page.

Returns (json):
```json
[
    {
        "jobId": "5f686d2d8c996402a02bbb92",
        "created": 1600679213751,
        "language": "nl",
        "status": "DONE",
        "nrAudioSeconds": 59,
        "transcriptionType": "transcription",
        "filename": "test.mp4",
        "jobType": "perfect",
        "jobOptions": {
            "transcriptionStyle": "cleanread"
        },
        "notes": null
    },
    {
        "jobId": "5f686d068c996402a02bbb85",
        "created": 1600679174451,
        "language": "nl",
        "status": "DONE",
        "nrAudioSeconds": 191,
        "transcriptionType": "transcription",
        "filename": "test.mp4",
        "jobType": "perfect",
        "jobOptions": {
            "transcriptionStyle": "cleanread"
        },
        "notes": null
    },
    {
        "jobId": "5f686d068c996402a02bbb82",
        "created": 1600679174415,
        "language": "nl",
        "status": "DONE",
        "nrAudioSeconds": 59,
        "transcriptionType": "transcription",
        "filename": "test.mp4",
        "jobType": "perfect",
        "jobOptions": {
            "transcriptionStyle": "cleanread"
        },
        "notes": null
    },
]
```

### CURL
```shell
curl --request GET --url 'https://qs.amberscript.com/jobs?apiKey=YOUR_API_KEY'
 ```

### NodeJS
```javascript
 var request = require("request");

var options = { method: 'GET',
  url: 'https://qs.amberscript.com/jobs',
  qs: { apiKey: 'YOUR_API_KEY' };

request(options, function (error, response, body) {
  if (error) throw new Error(error);

  console.log(body);
});
```

### Python
```python
import requests

url = "https://qs.amberscript.com/jobs"

querystring = {"apiKey":"YOUR_API_KEY"}

payload = ""
response = requests.request("GET", url, data=payload, params=querystring)

print(response.text)
```

### Java
```java
HttpResponse<String> response = Unirest.get("https://qs.amberscript.com/jobs?apiKey=YOUR_API_KEY")
  .asString();
```
---

## Support
If you need any technical assistance, feel free to contact `info (at) amberscript (dot) com`
