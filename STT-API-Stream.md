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
  `wss://v1/prod/api.bahasakita.co.id/stream?token=xxxxxxxxxx`


## **"How to Use" Flow**
  1. get token with access [Our API](./Auth-API.md)
  2. Send request message [AudioConn state](#state-1-audioconn) to get `sess_id=********`  from speech recognition service.
  3. Read bytes of the audio from your computer source ( microphone or file).
  4. Add `sess_id=********` in request message [AudioStream state](#state-2-audiostream), please send  `audio raw` format `signed integer` chunks size of `3200 bytes`  with `samplerate 16000` and `mono channel`.
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
  import os
  import sys
  import json
  import base64
  import time
  import websocket
  import threading
  from argparse import ArgumentParser


  bBreak = False

  def recv_message(self): 
      global bBreak
      while True:
          
          reply = self.recv()
          
          if not reply is None:
              
              reply = json.loads(reply)            
              
              if "bk" in reply :
                  if reply["bk"]["status"] == 400:                    
                      bBreak = True
                      print("Break Process")
                      break      
                  
                  elif "stt" in reply["bk"]["service"]:

                      if "data" in reply["bk"]:
                          print(reply["bk"]["data"])
                  
                      if reply["bk"]["type"] == "audioStop":
                          print("Audio Stop")
                          break
                      

  def main():


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

      websocket.enableTrace(False)
      ws = websocket.WebSocket()

      # change with your token.
      ws.connect("wss://v1/prod/api.bahasakita.co.id/stream?token=****************************") 
      
      f = open(args.filename, "rb")
      f.seek(0, os.SEEK_END)
      audiosize = f.tell()
      f.seek(44, os.SEEK_SET)
      rate = 16000
      chunksize = 3200
      remainder = audiosize

      #######################################
      # Example for processing "audioConn" type 
      #######################################

      msg = {
          "bk": {
              "service": "stt",
              "type": "audioConn",            
          }
      }
      msg = json.dumps(msg)

      ws.send(msg)
      reply = ws.recv()
      reply = json.loads(reply)
      
      status_code = reply["bk"]["status"]
      
      if status_code == 400:
          print(reply["bk"]["error"])
          return
      
      sess_id = reply["bk"]["data"]["sess_id"]
    
      if status_code == 200:
          
      
          sendThread = threading.Thread(target=recv_message, args=[ws])
          sendThread.setDaemon(True)
          sendThread.start()

          # if you set this read audio file
          while remainder > 0 :
              if bBreak :
                  break    
              if remainder < chunksize:
                  chunksize = remainder

              time.sleep(0.01)
              chunk = f.read(chunksize)
              bschunk = base64.b64encode(chunk)

              #######################################
              # Example for processing "audioStream" type 
              #######################################

              # please, add "sess_id"  with session id after you get from audio connect process.
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
              #print("Send audio stream... ")
              ws.send(msg)
          
              remainder = remainder - chunksize

          #######################################
          # Example for processing "audioStop" type 
          #######################################
            
          msg = {
              "bk": {
                  "service": "stt",
                  "sess_id": sess_id,                    
                  "type": "audioStop"                
              }
          }
          
          msg = json.dumps(msg)

          print("Send eos audio ...")
          ws.send(msg)
          sendThread.join()
          

      print("Streaming end...")
      ws.close()


  if __name__ == "__main__":
      main()

  ```