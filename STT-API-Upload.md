# **Speech to Text API**
API STT Documentation is guidance for communicate with bahasakita speech recognition service.

#### **Table of Contents**
  - [General API Information](#general-api-information)
  - [Tech Stack](#tech-stack)
  - [Diagram Process](#diagram-process)
  - [API Upload](#api-upload) 

## **General API Information**
  - The base endpoint is: 
    - [https://api.bahasakita.co.id](https://api.bahasakita.co.id) for [REST](https://restfulapi.net/)
     - All endpoints return JSON object.

## **Tech Stack**
  - **[REST](https://restfulapi.net/)**
  

## **Diagram Process**
  ![Diagram Process](/asset/image.png "Diagram Process")
 
 
## **API Upload**
  In order to use this API, you required to create account by registering yourself.

### **"How to Use" Flow**
  1. Get your token with [Our API](./Auth-API.md) 
  2. Upload the audio you want to transcribe. 
  3. Wait for response, the transcript will be included in the response body.
   
### **Host:**
  [https://api.bahasakita.co.id](https://api.bahasakita.co.id)

### **Endpoint**
  `/v1/prod/stt/upload`

### **Method:**
  `POST`

### **Request**
##### **Headers**
  | Name | Format |
  | ------ | ------ |
  | Content-Type | `multipart/form-data` |
   | Authorization | `Bearer token` |

##### **Body**
  | Field | Data Type | Description |
  | ------ | ------ | ------ |
  | file | File |Your audio file  |

### **Response**
  | Field | Data Type | Description |
  | ------ | ------ | ------ |
  | message | String | `success` or `error` message|
  | text | String |  |

### **Response Example :**
```
{
  "bk":{
    "message":"success or error message",
    "data" :{
      "text":"transcript"
    }
  }
}

```

### **Sample Call in Python:**
```
import requests
import sys
import os
from argparse import ArgumentParser

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

    url = "https://api.bahasakita.co.id/v1/prod/stt/upload"
    
    headers={'Authorization': 'Bearer <your token>'}
    
    
    file = {'file': open(args.filename,'rb')}
    response = requests.request("POST", url,headers=headers, files = file)
    print(response.text)

if __name__ == "__main__":
    main()

```