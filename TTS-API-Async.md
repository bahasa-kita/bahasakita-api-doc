# **Text to Speech API**

Asynchronous Text-to-Speech API Documentation is guidance for communicate with bahasakita speech synthesize service, without having to wait for synthesis processing to complete and return an audio url path result. The text in each asynchronous request is limited to 3000 characters.


#### **Table of Contents**
  - [General API Information](#general-api-information)
  - [Tech Stack](#tech-stack)
  - `API`
    - [Asynchronous TTS API](#asynchronous-tts-api) 
    - [Retrieving Result](#retrieving-result-api)

## **General API Information**
  - The base endpoint is: 
    - [https://api.bahasakita.co.id](https://api.bahasakita.co.id) for [REST](https://restfulapi.net/)
     - endpoints return can be JSON object or audio content-type.

## **Tech Stack**
  - **[REST](https://restfulapi.net/)**
  
 
## **Asynchronous TTS API**
  In order to use this API, you required to create account by registering yourself.

### **"How to Use" Flow**
  1. Get your token with [Our API](./Auth-API.md), and set in the request header for authorization.  
  2. Send your message in `plantext` or `SSML`. How to use SSML, you can check this link [SSML](https://github.com/bahasa-kita/ssml-doc) 
  3. You will get return an audio url path and an expired date response indicating when the audio can still be retrieved.
   
### **Host:**
  [https://api.bahasakita.co.id](https://api.bahasakita.co.id)

### **Endpoint**
  `/v1/prod/tts/async`

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
            "text": "selamat datang di rumah kami ini"
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
This is an example of generating text using SSML type. To understand more, you can follow this [SSML](https://github.com/bahasa-kita/ssml-doc)

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
      "text":"selamat datang di rumah kami ini.",
      "path":"https://api.bahasakita.co.id/v1/prod/tts/async/content/330576b24f47dd7501a08af9ebcd2e7b4cdd7f2d30a67b245f291359d31687f353d0478d9540367cbeb039a854cac080.wav",
      "expired": "2022-07-22T04:11:01Z"
    }, 
    "config": {
      "volume": 1, 
      "pitch": "normal", 
      "speed": 1.0
    }
  }
}

```

## **Retrieving Result API**
  In order to use this API, you required to create account by registering yourself. 
  Using `urlpath` object we previously from [async TTS API](#asynchronous-tts-api) response.  

### **"How to Use" Flow**
  1. Get your token with [Our API](./Auth-API.md), and set in the request header for authorization.  
  2. Get audio result by accessing an audio URL path.
  3. It's sucessful, you can play audio result. 

### **Host:**
  [https://api.bahasakita.co.id](https://api.bahasakita.co.id)

### **Endpoint**
  `/v1/prod/tts/async/content/<audio_id>`

### **Method:**
  `GET`

### **Request**
#### **Headers**
  | Name | Format |
  | ------ | ------ |
   | Authorization | `Bearer token` |

#### **Example Request**

This is an example of get audio from an audio url path.

```
GET https://api.bahasakita.co.id/v1/prod/tts/async/content/330576b24f47dd7501a08af9ebcd2e7b4cdd7f2d30a67b245f291359d31687f353d0478d9540367cbeb039a854cac080.wav

```
### **Responses**
  | Status | Meaning | Description |
  | ------ | ------ | ------ |
  | 200 | OK, Audio Available |  audio file is available and can be picked up, with header `'content-type':'audio/wav'`|
  | 204 | OK, No Content | synthesis audio  ongoing process. It's ok, but audio still not complete |
  | 400 | Request Failed  |  a processing error occurred|
  | 404 | Not Found  | audio not found, maybe because it hasn't been synthesized yet|
  | 410 | Gone  | audio has passed expired date, so it's time to generate again |



### **Sample Call in Python:**
```
import requests
import time
import json

url = "https://apidev.bahasakita.co.id/v1/prod/tts/async"
token ="<your_token>" # set your token

def main():

    text= "Halo saya TTS bahasakita"
    label="text"
    
    urlpath, expired_date = tts_async(text,label)
    
  
    if urlpath is not None :
        
        print("expired-date audio :", expired_date)
      
        audio = get_audio(urlpath)
        audiofile = "audiofile.wav"

        try:
            with open(audiofile, "wb") as f:
                f.write(audio)
                f.close()
           
            #os.system("play "+audiofile) #install sox tools for play recording audio .

        except:
            print("audio failed")
    else:
        print(" TTS failed")

def tts_async(text : str,label : str):
    path = None
    expired = None
    
    headers={'Authorization': 'Bearer '+token+''}
    
    msg = {
        "bk": {
            "data": {
                "label" : "text",
                "text"  : "mencoba menghasilkan audio dari tts bahasakita"
            }
        }
    }

    response = requests.request("POST", url,headers=headers, json=msg)
    
    if response.status_code == 200:

        body = json.loads(response.text)        
        path = body['bk']['data']['path']
        expired = body['bk']['data']['expired']
        
    else:
        print(response.text)

    return path,expired

def get_audio(path : str):
    result = None

    headers={'Authorization': 'Bearer '+token+''}        
    
    while True:
        
        response = requests.get(path,headers=headers)

        if response.status_code == 204:
            time.sleep(1) # wait 1 second, and then check again audio.
            continue
        elif response.status_code == 200:
            result = response.content 
            break
        elif response.status_code == 400:
            print("processing failed")
            break
        elif response.status_code == 410:
            print("audio url path expired")    
            break
        else:
            print("unidentified")
            break

    return result

if __name__ == "__main__":
    main()   
        
```