# **Speech to Text API**
Documentation API STT to bahasakita speech recognition service with the WebSocket protocol as defined in RFC 6455

#### **Table of Contents**
  - [General API Information](#general-api-information)
  - [Diagram Activity](#diagram-activity)
  - [API](#list-of-states)
    - [Audio Connection](#state-1-audioconn)
    - [Audio Stream](#state-2-audiostream)
    - [Audio Stop](#state-3-audiostop)

## **General API Information**
  - The base endpoint is: 
    - `wss://api.bahasakita.co.id` used WEBSOCKET RFC 6455.
    - All endpoints return JSON object.

## **Diagram Activity**
  ![Diagram Activity](/asset/activity_diagram.jpg "Diagram Activity")

## **List of States**
  | Name | State Types | Description |
  | ------ | ------ | ------ |
  | [start stream audio](#state-1-audioconn) |  type : AudioConn    | initialize stream before send audio |
  | [send stream audio](#state-2-audiostream) | type : AudioStream  | send audio after stream initialized |
  | [stop stream audio](#state-3-audiostop) | type : AudioStop    | end stream. you need to request [start stream audio](#start-stream-audio) if you want to send audio again after [stop stream audio](#stop-stream-audio) |

## **URL ACCESS**
  `wss://api.bahasakita.co.id/v1/prod/stream?token=xxxxxxxxxx`


## **"How to Use" Flow**
  1. get token with access [Our API](./Auth-API.md)
  2. Send request message [AudioConn state](#state-1-audioconn) to get `sess_id=********`  from speech recognition service.
  3. Read bytes of the audio from your computer source ( microphone or file).
  4. Add `sess_id=********` in request message [AudioStream state](#state-2-audiostream), please send  `raw audio` format `signed integer` chunks size of `3200 bytes`  with `samplerate 16000` and `mono channel`.
  5. Speech recognition service will return transcript response
  6. Send request message [AudioStop state](#state-3-audiostop) to end process.



### **STATE 1 AudioConn**
  AudioConn is the first step to start sending the audio bytes you want to transcribe. If successful, ASR service will returned `session id` to mark the bytes audio can be sent.

### **Request**
##### **Body**
  | Field |Data Type | Description |
  | ------ | ------ | ------ |
  | service | String | `stt` for speech recognition service|
  | type | String | `AudioConn` |

##### **Data Structure**
  ```
{
  "bk": {
      "service": "stt",
      "type": "audioConn"            
  }
}
  ```
### **Response**
##### **Body**
  | Field | Data Type | Description |
  | ------ | ------ | ------ |
  | status | Int | `200` is `success` or `400` is `failed` |
  | sess_id | String |  unique id with size of 32 characters|  
  | message | String | only filled if status `failed`, description of the error that occurred |
##### **Data Structure**
```
{
  "bk":{
    "service":"stt",
    "type":"audioConn",
    "status":200,
    "message":"error message",
    "data" :{
      "sess_id":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    }
  }
}
```

### **STATE 2 AudioStream**
### **Request**
##### **Body**
  | Field |Data Type | Description |
  | ------ | ------ | ------ |
  | service | String | `stt` for speech recognition service|
  | type | String | `AudioStream` |
  | sess_id | String |  unique id with size of 32 characters from audio connection|
  | audio | String | bytes audio with base64 encode |
##### **Data Structure**
```
{
  "bk": {
    "service": "stt",
    "type": "audioStream",            
    "sess_id":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "data":{
      "audio": base64(bytes)    
    }
  }
}

```

### **Response**
##### **Body**
  | Field | Data Type | Description |
  | ------ | ------ | ------ |
  | status | Int | `200` is `success` or `400` is `failed` |
  | message | String | only filled if status `failed`, description of the error that occurred |
  | partial | String | part of utterance, output text less than 2 words |
  | final | String | full transcript or utterance transcript  |

##### **Data Structure**
```
{
  "bk":{
    "service":"stt",
    "type":"audioStream",
    "status":200,
    "message":"error message",
    "data" :{
      "partial":"text"
    }
  }
}

```
OR

```
{
  "bk":{
    "service":"stt",
    "type":"audioStream",
    "status":200,
    "message":"error message",
    "data" :{
      "final":"text"
    }
  }
}

```

### **STATE 3 AudioStop**
### **Request**
##### **Body**
  | Field |Data Type | Description |
  | ------ | ------ | ------ |
  | service | String | `stt` for speech recognition service|
  | type | String | `AudioStop` |
  | sess_id | String |  unique id with size of 32 characters from audio connection|

##### **Data Structure**
```
{
  "bk":{
      "service": "stt",
      "type": "audioStop",            
      "sess_id":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"            
  }
}
```

### **Response**
##### **Body**
  | Field | Data Type | Description |
  | ------ | ------ | ------ |
  | status | Int | `200` is `success` or `400` is `failed` |
  | message | String | only filled if status `failed`, description of the error that occurred |
  
##### **Data Structure**
```
{
  "bk":{
    "service":"stt",
    "type":"audioStop",
    "status":200,
    "message":"error message"
  }
}
```

### **Sample Call in Python:**
```
import asyncio
import json
import websockets
import sys
import os
from argparse import ArgumentParser
import base64

async def main():

    parser = ArgumentParser()
    parser.add_argument("-f", "--file", dest="filename",
                help="write report to FILE", metavar="FILE")
    parser.add_argument("-v", "--verbose",
                action="store_false", dest="verbose", default=True,
                help="don't print status messages to stdout")


    args = parser.parse_args()

    if len(sys.argv) <= 2 or not os.path.exists(args.filename) :
      parser.print_help()
      return
    
    filename = args.filename

    # Websocket URL, changed with your token     
    url = "wss://api.bahasakita.co.id/v1/prod/stream?token=<your_token>"
    
    async with websockets.connect(url) as ws:

        # Configure the session
        msg = {
        "bk": {
            "service": "stt",
            "type": "audioConn"           
        }
        } 
        await ws.send(json.dumps(msg))
        reply = await ws.recv()  # Receive message
        # print("Response :",reply)    
        reply = json.loads(reply)
        if reply["bk"]["status"] == 200 :
           
            sess_id = reply["bk"]["data"]["sess_id"]
            # Concurrently send audio data and receive results
            await asyncio.gather(
                send_audio(filename, ws, sess_id),
                receive_message(ws)
            )


async def send_audio(filename: str, ws,sess_id, chunk_size: int = 3200):
    with open(filename, "rb") as f:  # Read file to send

        while data := f.read(chunk_size):       
            bschunk = base64.b64encode(data)
            msg = {
                "bk": {
                    "service": "stt",
                    "sess_id": sess_id,
                    "type": "audioStream", 
                    "data": {
                        "audio": bschunk.decode("utf-8")
                    },
                }
            }
            msg = json.dumps(msg)
            await ws.send(msg)

        msg = {
            "bk": {
                "service": "stt",
                "sess_id": sess_id,                    
                "type": "audioStop"                
            }
        }        
        msg = json.dumps(msg)    
        await ws.send(msg)

async def receive_message(ws):
    while True:
        
        reply = await ws.recv()  # Receive message
        if not reply is None:
            
            reply = json.loads(reply)   
            if "stt" in reply["bk"]["service"]:

                if "data" in reply["bk"]:
                    if "final" in reply["bk"]["data"]:
                        print("Final :",reply["bk"]["data"]["final"])
                    elif "partial" in reply["bk"]["data"]:
                        print("Partial :",reply["bk"]["data"]["partial"])
                        

if __name__ == '__main__':
    asyncio.run(main())

```
