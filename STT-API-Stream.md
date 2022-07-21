# **Speech to Text API**
API STT Documentation is guidance for communicate with bahasakita speech recognition service with the WebSocket protocol as defined in RFC 6455

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
  3. Read bytes of the audio from your device computer ( microphone or file).
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
import base64
import json
import functools
import signal
import pyaudio
import websockets

FORMAT = pyaudio.paInt16
CHANNELS = 1
RATE = 16000
CHUNK = int(RATE / 10) # 3200 bytes / 100ms

queue: asyncio.Queue = None


def ask_exit(signame, stream):
    stream.stop_stream()    
    print("got signal %s: exit" % signame)
    

async def main():
    # Websocket URL, changed with your token
    url = f"wss://api.bahasakita.co.id/v1/prod/stream?token={AUTH_TOKEN}"

    global queue
    queue = asyncio.Queue()

    audio = pyaudio.PyAudio()
    stream = audio.open(
            format=FORMAT,
            channels=CHANNELS,
            rate=RATE,
            input=True,
            frames_per_buffer=CHUNK,
            stream_callback=pyaudio_callback,
        )
    
    loop = asyncio.get_running_loop()
    
    for signame in {'SIGINT', 'SIGTERM'}:
                loop.add_signal_handler(
                getattr(signal, signame),
                functools.partial(ask_exit, signame,stream))

    async with websockets.connect(url) as ws:

        # sending "audioConn type"
        msg = {"bk": {"service": "stt", "type": "audioConn"}}
        await ws.send(json.dumps(msg))
        reply = await ws.recv()  # Receive message

        print("Response :", reply)
        reply = json.loads(reply)
        if reply["bk"]["status"] == 200:
            sess_id = reply["bk"]["data"]["sess_id"]

            recieve_task = asyncio.create_task(receive_message(ws,stream))
            stream_task = asyncio.create_task(stream_mic(ws,sess_id,stream))

            await asyncio.gather(
                recieve_task,
                stream_task               
            )
            stream.close()
            print("Finished...")   

async def stream_mic(ws,sess_id,stream):

    print("Streaming MIC .... ")
    
    while not stream.is_stopped():
        data,status = await queue.get()  
        #print(len(data))
        bschunk = base64.b64encode(data)

        # sending "audioStream type"
        msg = {
            "bk": {
                "service": "stt",
                "sess_id": sess_id,
                "type": "audioStream",
                "data": {"audio": bschunk.decode("utf-8")},
            }
        }
        msg = json.dumps(msg)
        #print("send :")
        await ws.send(msg)
    
    # sending "audioStop type"
    msg = {"bk": {"service": "stt", "sess_id": sess_id, "type": "audioStop"}}
    msg = json.dumps(msg)
    await ws.send(msg)
    #time.sleep(0.1)

def pyaudio_callback(in_data, frame_count, time_info, status):
    # print("audio callback received")
    queue.put_nowait((in_data, status))
    # print("putted")
    return in_data, pyaudio.paContinue

async def receive_message(ws, stream):
    print("Waiting Response....")

    while not stream.is_stopped() :
        try:
            reply =  await asyncio.wait_for(ws.recv(), timeout=0.01) # Receive message
            #print("REPLY :",reply)
            if not reply is None:
                reply = json.loads(reply)                
                
                if "stt" in reply["bk"]["service"]:
                    if "type" in reply["bk"] and "audioStop" == reply["bk"]["type"]:  
                        print("end of stream audio")                        
                    elif "data" in reply["bk"]:
                        if "final" in reply["bk"]["data"]:
                            print("Final :", reply["bk"]["data"]["final"])
                        elif "partial" in reply["bk"]["data"]:
                            print("Partial :", reply["bk"]["data"]["partial"])      
                        
        except asyncio.TimeoutError:
            pass


if __name__ == "__main__":
    asyncio.run(main())

```
