# **Text to Speech API**

Synchronous Text-to-Speech API Documentation is guidance for communicate with bahasakita speech synthesize service, wait for the processing to finish. The text in each synchronous request is limited to 300 characters.


#### **Table of Contents**
  - [General API Information](#general-api-information)
  - [Tech Stack](#tech-stack)
  - [Synchronous TTS API](#synchronous-tts-api) 

## **General API Information**
  - The base endpoint is: 
    - [https://api.bahasakita.co.id](https://api.bahasakita.co.id) for [REST](https://restfulapi.net/)
     - All endpoints return JSON object.

## **Tech Stack**
  - **[REST](https://restfulapi.net/)**
  
 
## **Synchronous TTS API**
  In order to use this API, you required to create account by registering yourself.

### **"How to Use" Flow**
  1. Get your token with [Our API](./Auth-API.md), and set in the request header for authorization. 
  2. Send your message in `plantext` or `SSML`. How to use SSML, you can check this link [SSML](https://github.com/bahasa-kita/ssml-doc) 
  3. You will get the message response in the json format containing audio file into base64-encoded data.
   
### **Host:**
  [https://api.bahasakita.co.id](https://api.bahasakita.co.id)

### **Endpoint**
  `/v1/prod/tts/sync`

### **Method:**
  `POST`

### **Request**
#### **Headers**
  | Name | Format |
  | ------ | ------ |
   | Authorization | `Bearer token` |

#### **Body**
  | Field | Data Type | Description |
  | ------ | ------ | ------ |
  | label | String | text type `planttext` or `ssml` |
  | text | String | text to be generated |

#### **Example Request with plantext**

This is an example of generating text using the plantext type

```
{
    "bk": {
        "data": {
            "label": "text",
            "text": "selamat datang"
        },
        "config": {
            "volume": 1,
            "pitch": "normal",
            "speed": 1.0
        }
    }
}

```

#### **Example Request with SSML**
This is an example of generating text using SSML type. To understand more, you can follow this link [SSML](https://github.com/bahasa-kita/ssml-doc)

```
{
    "bk": {
        "data": {
            "label": "ssml",
            "text": "<speak> Kamu urutan <say-as interpret-as="ordinal">10</say-as> dalam baris. </speak>"
        }
    }
}

```


#### **Example Response:**
```
{
  "bk": {
    "data": {
        "text": "selamat datang.", 
        "audio": base64(audio),
        "audio_format": { 
          "format": "wav", 
          "sample-rate": 16000, 
          "precision": 16, 
          "channel": "mono", 
          "encoding":"signed-integer"
        }
    }, 
    "config": {
      "volume": 1, 
      "pitch": "normal", 
      "speed": 1.0
    }  
  }
}

```

### **Sample Call in Python:**
```
import requests
import json
import base64

url = "https://apidev.bahasakita.co.id/v1/prod/tts/sync"
token = "<your_token>" #set your token   

def main():

    headers={'Authorization': 'Bearer '+token+''} 
    
    msg = {
        "bk": {
            "data": {
                "label" : "text",
                "text"  : "halo TTS bahasakita"
            },
            "config": {
                "volume": 1,
                "pitch" : "normal",
                "speed" : 1.0
            }
        }
    }

    response = requests.request("POST", url,headers=headers, json=msg)

    if response.status_code == 200:

        print('audio succeed')        
        body = json.loads(response.text)

        try:
            encoded_audio = body['bk']['data']['audio']
            format_audio = body['bk']['data']['audio_format']['format']
            
            audio = base64.b64decode(encoded_audio) 
            audiofile = "audiofile." + format_audio
        
            with open(audiofile, "wb") as f:
                f.write(audio)
                f.close()
           
            #os.system("play "+audiofile) #install sox tools for play recording audio .

        except:
            print("audio failed")
    else:
        print(response.text)

if __name__ == "__main__":
    main()   

```