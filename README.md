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
## Events
### /events/list
List the events the user has permissions to view.
```
https://api.last2ticket.com/events/list?api_key=<YOUR_API_KEY>
```

##### Success Response
```
{
    "stt": "OK",
    "message": {
        "text": "List of events user - [USER_ID] can view.",
        "events": [
            EVENT_1,
            EVENT_2,
            EVENT_3,
            ...
        ]
    }
}
```

List the event items. The list includes all items that are active and is not ('sold_out','not_available', 'available_soon’).
```
https://api.last2ticket.com/events/<EVENT_ID>/items?api_key=<YOUR_API_KEY>
```

##### Success Response
```
{
    "stt": "OK",
    "message": {
        "text": List of items for event [EVENT_ID].",
        "items": [
            {
                "id": ITEM_1,
                "event_id": EVENT_ID,
                "session_id": SESSION_A,
                "price": "10.0000",
                "category_name": "Bilhete XPTO"
            }, {
                "id": ITEM_2,
                "event_id": EVENT_ID,
                "session_id": SESSION_B,
                "price": "5.0000",
                "category_name": "Bilhete ABCD"
            }
        ]
    }
}
```

## Tickets
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


## Discount codes
Create new discount codes for specific events owned by you. Add and remove events and items to discount codes.

There are specific operations for /PUT /GET and /DELETE for just event_id and event_id + item_id combination. 
This means users can add specific items under an event just like in UX. But removing all event_id + item_id will not create a simple discount->item relation like it happens in UX, this is not intuitive in an API. 


Example: User lists records in api_discount_codes_events for discount_1 and gets

```
discount_1 - event_50 - item_1
discount_1 - event_50 - item_2
```

He then removes the 2 items with:
```
DELETE /discount-code/discount_1/event/event_50/item/item_1
DELETE /discount-code/discount_1/event/event_50/item/item_2
```

In the API the result of the following endpoints will be empty,

```
LIST /discount-code/discount_1/events
GET /discount-code/discount_1/event/event_50
```

In the Admin app UX, the result is actually 1 record, because the UX is adding a record for discount_1 - event_50 when all items are removed for an event.
To achieve similar behaviour add the relation explicitly after the 2 deletes: 
```
PUT /discount-code/discount_1/event/event_50
```

### List 
List discount codes for event_id
```
GET /discount-code/list/event/<EVENT_ID>?api_key=<YOUR_API_KEY>
```
Response sample
```
{
    "stt": "OK",
    "message": {
        "event_id": "<EVENT_ID>",
        "text": "Discount codes list.",
        "discount_codes": "[
            {\"discount_id\":12345,\"events_id\":500,\"item_id\":null,\"code\":\"SUPER\",\"type\":\"value\",\"value\":\"5.0000\",\"active\":\"yes\"},
            {\"discount_id\":54321,\"events_id\":500,\"item_id\":10,\"code\":\"MEGA\",\"type\":\"percent\",\"value\":\"5.0000\",\"active\":\"yes\"},
            {\"discount_id\":54321,\"events_id\":500,\"item_id\":20,\"code\":\"2020\",\"type\":\"percent\",\"value\":\"5.0000\",\"active\":\"no\"}
        ]"
    }
}
```

### Create
Create a discount code for a specific event and items. 
```
POST /discount-code/create?api_key=<YOUR_API_KEY>
```
Input parameters
- **code(*)** : string (max length 20) (*) - The discount code name, unique across L2T.
- **type(*)** : enum[percent, value] - The discount type, can be a value (e.g. 5€) or a percentage (e.g. 10%).
- **value(*)** : number (max length 8) - Actual discount value (e.g. 5).
- **limit** : number - Number of times this code can be used (e.g. 100 - after 100 usages in checkout the discount will no longer take effect).
- **expires** : date (format - yyyy-mm-dd) - Date until when the discount is valid (e.g. "2023-12-21").
- **active** : boolean - If the discount is active or inactive.
- **event_id(*)** : number - The event to which this discount will apply.
- **item_id** : array[number] - Specific items inside an event (e.g. [10, 20], the discount will apply only to event item 10 and 20). In no items are provided, the discount applies to the whole event_id.
- **show_invites**: number [0, 1] - This allows users using Generic invite UI to see the receipients list of discount code. If set to 0, users will not see the list, 1 users see the list.

**(*)** - required


Payload sample
```
{
    "code": "MEGA",
    "value": 5,
    "type": "percent",
    "limit": 100,
    "expires": "2023-12-21",
    "active": true,
    "event_id": 500,
    "item_id": [10, 20],
    "show_invites": 1
}
```


### Get details
Get a discount code detailed information.
``` 
GET /discount-code/<DISCOUNT_ID>?api_key=<YOUR_API_KEY>
```
Response
```
"message": {
    "discount_id": "12345",
    "text": "Discount details",
    "discount": "[{\"id\":12345,\"type\":\"percent\",\"value\":\"5.0000\",\"code\":\"MEGA\",\"limit\":100,\"in_cart\":0,\"used\":0,\"expires\":\"2023-12-21\",\"active\":\"yes\"}]"
},
```

### Get discount events
Get discount code events (and items) for discount_id
``` 
GET /discount-code/<DISCOUNT_ID>/events?api_key=<YOUR_API_KEY>
``` 
Response
```
"message": {
    "discount_id": "12345",
    "text": "Discount events list",
    "discount_events": "[
        {\"id\":444,\"discount_id\":12345,\"events_id\":500,\"item_id\":10},
        {\"id\":239007,\"discount_id\":12345,\"events_id\":500,\"item_id\":20}]"
},
```

### Activate
Activate a discount code.
```
PUT /discount-code/<DISCOUNT_ID>/activate?api_key=<YOUR_API_KEY>
```
Response
```
"message": {
    "discount_id": "DISCOUNT_ID",
    "text": "Activated discount: DISCOUNT_ID",
    "active": "yes"
},
```

### Deactivate
Deactivate a discount code.
```
PUT /discount-code/<DISCOUNT_ID>/deactivate?api_key=<YOUR_API_KEY>
```
Response
```
"message": {
    "discount_id": "DISCOUNT_ID",
    "text": "Deactivate discount: DISCOUNT_ID",
    "active": "no"
},
```

### Add item
Associate an item to an existing discount code

```
PUT /discount-code/<DISCOUNT_ID>/event/<EVENT_ID>/item/<ITEM_ID>?api_key=<YOUR_API_KEY>
```
Response
```
"message": {
    "discount_id": "DISCOUNT_ID",
    "event_id": "EVENT_ID",
    "item_id": "ITEM_ID",
    "text": "Added event+item to discount",
    "relation_id": 123456
}
```

### Get item
Get the relation id between an item to an existing discount code

```
GET discount-code/<DISCOUNT_ID>/event/<EVENT_ID>/item/<ITEM_ID>?api_key=<YOUR_API_KEY>
```

Response
```
"message": {
    "discount_id": "DISCOUNT_ID",
    "event_id": "EVENT_ID",
    "item_id": "ITEM_ID",
    "text": "Got event+item relation to discount",
    "relation": {
        "id": EVENT_ID,
        "discount_id": DISCOUNT_ID,
        "events_id": EVENT_ID,
        "item_id": 123456
    }
},
```

### Delete item
Delete the relation between an item and a discount code

```
DELETE /discount-code/<DISCOUNT_ID>/event/<EVENT_ID>/item/<ITEM_ID>?api_key=<YOUR_API_KEY>
```
Response
```
"message": {
    "discount_id": "DISCOUNT_ID",
    "event_id": "EVENT_ID",
    "item_id": "ITEM_ID",
    "text": "Deleted event+item from discount",
    "deleted": true
},
```

### Add event
Associate an event to an existing discount code

```
PUT /discount-code/<DISCOUNT_ID>/event/<EVENT_ID>?api_key=<YOUR_API_KEY>
```
Response
```
"message": {
    "discount_id": "DISCOUNT_ID",
    "event_id": "EVENT_ID",
    "text": "Added event to discount",
    "relation_id": 12345
},
```

### Get event
Get the relation id between an event and an existing discount code

``` 
GET /discount-code/<DISCOUNT_ID>/event/<EVENT_ID>?api_key=<YOUR_API_KEY>
```
Response
```
"message": {
    "discount_id": "DISCOUNT_ID",
    "event_id": "EVENT_ID",
    "text": "Got event+item relation to discount",
    "relations": [
        {
            "id": 12345,
            "discount_id": DISCOUNT_ID,
            "event_id": EVENT_ID,
            "item_id": 10
        },
        {
            "id": 54321,
            "discount_id": DISCOUNT_ID,
            "event_id": EVENT_ID,
            "item_id": 20
        }
    ]
},
```


### Delete event
Delete the relation id between an event and an existing discount code

``` 
DELETE /discount-code/<DISCOUNT_ID>/event/<EVENT_ID>?api_key=<YOUR_API_KEY>
```

```
"message": {
    "discount_id": "DISCOUNT_ID",
    "event_id": "EVENT_ID",
    "text": "Deleted event from discount",
    "relation_id": true
},
```


### Error messages
E.g. HTTP 400 - Invalid code

```
{
    "stt":"NOK",
    "message":"[Invalid request] Code: MEGA_SALE_2022 already in use, please select a new one."
}
```
