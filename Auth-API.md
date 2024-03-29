
# AUTH API DOCUMENTATION
Auth API Documentation is instructions for creating an user authorization that will be used to take advantage of all bahasakita services.


#### **Table of Contents**
  - [General API Information](#general-api-information)
  - [Tech Stack](#tech-stack)
  - [Diagram Process](#diagram-process)
  - `API`
    - [Register](#api-register) 
    - [Get Token](#api-get-token)    

## **General API Information**
  - The base endpoint is: 
    - [https://api.bahasakita.co.id](https://api.bahasakita.co.id) for [REST](https://restfulapi.net/)
     - All endpoints return JSON object.

## **Tech Stack**
  - **[REST](https://restfulapi.net/)**
  

## **Diagram Process**
  ![Diagram Process](/asset/image.png "Diagram Process")
 
 
## **API Register**
In order to use this API services, you are required to create an account by registering yourself through this endpoint.

### **"How to Use" Flow**
  1. Send request to the endpoint with required fields. 
  2. Wait for response, if `success` you can use your newly created account to use our api service.

### **Host:**
  [https://api.bahasakita.co.id](https://api.bahasakita.co.id)

### **Endpoint**
  `/v1/prod/register`

### **Method:**
  `POST`

### **Request**
##### **Headers**
  | Name | Format |
  | ------ | ------ |
  | Content-Type | `application/json` |
##### **Body**
  | Field | Data Type | Description |
  | ------ | ------ | ------ |
  | email | String |standard email |
  | username| String |  your unique name|
  | firstname| String | first name|
  | lastname| String | last name |
  | password| String | your password |

##### **Request Example:**
```
    {"bk":
      {"data":
        {   "email":  "example@somemail.com",
            "username":  "Your Name",
            "firstname": "First Name",
            "lastname": "Last Name", 
            "password":"your password"
        }
      }
    }
```      

### **Response**
##### **Body**
  | Field | Data Type | Description |
  | ------ | ------ | ------ |
  | message | String | `success` or `error` message|

##### **Response Example:**
```
    {"bk":
        {
            "message":"success"
        }
    }  
```
   

## **API Get Token**
  In order to use STT service, you need to get the token with your registered account.

### **"How to Use" Flow**
  1. Send request to the endpoint with required fields. 
  2. Wait for response, if `success` you can use the token to use our API service.


### **Host:**
  [https://api.bahasakita.co.id](https://api.bahasakita.co.id)

### **Endpoint**
  `/v1/prod/getToken`

### **Method:**
  `POST`

### **Request**
##### **Headers**
  | Name | Format |
  | ------ | ------ |
  | Authorization | `Basic Base64(email:password)` |

### **Response**
  | Field | Data Type | Description |
  | ------ | ------ | ------ |
  | message | String | `success` or `error` message|
  | token | String | |

##### **Response Example:**
```
    {"bk":{
        "messsage":"success",
        "data" : {
            "token":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
            }
        }
    }  
```
