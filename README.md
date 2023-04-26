![alt text](https://hello.last2ticket.com/wp-content/uploads/2018/10/Logo-L2T-01.svg)

# API Docs
This documentation is targeted at external developers integrating with Last2Ticket public API.

### API - https://api.last2ticket.com

To access the API you must request an api_key to Last2Ticket admin. 
All requests to the api must provide a valid key 

- Access with the api_key is done with a query parameter: 
```
https://api.last2ticket.com/some/endpoint?api_key=<YOUR_API_KEY>
```
- If no api_key is provided the API return error 
```
{
  "code": 100,
  "stt": "NOK",
  "msg": "100 invalid_auth",
  "info": "Forbidden : not authenticated",
  "wer": "Forbidden : not authenticated"
}
```


---
# Endpoints

###  /ticket/checkin/code/{code}
Ticket checking, this endpoint is used to checkin in last2ticket system a ticket with your specific vendor id (external_id) that you use in your platform. 
```
https://api.last2ticket.com/ticket/checkin/code/<YOUR_CODE>?api_key=<YOUR_API_KEY>
```

##### Success Response
```
{
  "stt": "OK",
  "message": {
    "external_code": "<YOUR_CODE>",
    "text": "Checkin completed."
  }
}
```

##### Errors
External code not found - this code is not associated with any sold ticket in Last2Ticket platform.
```
{
  "stt": "NOK",
  "message": {
    "external_code": "<YOUR_CODE>",
    "text": "External code not found."
  }
}
```




