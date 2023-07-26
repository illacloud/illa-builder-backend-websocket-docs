ILLA Builder Backend WebSocket Text Protocols
---------------------------------------------

# Index
- [Index](#index)
- [Consts](#consts)
  - [Request Signal](#request-signal)
  - [Option](#option)
  - [Request Target](#request-target)
- [SIGNAL\_PING](#signal_ping)
  - [Request Sample](#request-sample)
    - [Response Sample](#response-sample)
- [SINGNAL\_ENTER](#singnal_enter)
  - [Request Sample](#request-sample-1)
  - [Response Sample](#response-sample-1)
- [SINGNAL\_LEAVE](#singnal_leave)
  - [Request Sample](#request-sample-2)
  - [Response Sample](#response-sample-2)
  - [Broadcast Sample](#broadcast-sample)
- [SIGNAL\_CREATE\_STATE](#signal_create_state)
  - [Request](#request)
- [SIGNAL\_DELETE\_STATE](#signal_delete_state)
  - [Request](#request-1)
- [SIGNAL\_UPDATE\_STATE](#signal_update_state)
  - [Request](#request-2)
- [SIGNAL\_MOVE\_STATE](#signal_move_state)
  - [Request](#request-3)
- [SIGNAL\_CREATE\_OR\_UPDATE\_STATE](#signal_create_or_update_state)
  - [Request](#request-4)
- [SIGNAL\_COOPERATE\_ATTACH](#signal_cooperate_attach)
  - [Request](#request-5)
  - [Response (Other users will also receive the broadcast)](#response-other-users-will-also-receive-the-broadcast)
- [SIGNAL\_COOPERATE\_DISATTACH](#signal_cooperate_disattach)
  - [Request](#request-6)
  - [Response (Other users will also receive the broadcast)](#response-other-users-will-also-receive-the-broadcast-1)
- [Response Protocol Format](#response-protocol-format)
  - [Sample](#sample)
  - [Response Signal Const](#response-signal-const)
  - [Data Structure used by Frontend](#data-structure-used-by-frontend)
- [Request Sample](#request-sample-3)



# Consts


## Request Signal 

- Defines the operation to be performed by the protocol. The default is 0, which represents a ping operation.

```go
const SIGNAL_PING                  = 0   // ping
const SIGNAL_ENTER                 = 1   // enter the room (app)
const SIGNAL_LEAVE                 = 2   // leave the room (app)
const SIGNAL_CREATE_STATE          = 3   
const SIGNAL_DELETE_STATE          = 4   
const SIGNAL_UPDATE_STATE          = 5   
const SIGNAL_MOVE_STATE            = 6   
const SIGNAL_CREATE_OR_UPDATE      = 7   
const SIGNAL_ONLY_BROADCAST        = 8   
const SIGNAL_PUT_STATE             = 9   
const SIGNAL_GLOBAL_BROADCAST_ONLY = 10  // Global broadcast, all clients under the current instance will receive the broadcast, suitable for broadcasting app, resource change scenario.
const SIGNAL_COOPERATE_ATTACH      = 11  // Collaboration signal, user Attach to a component, currently broadcast only
const SIGNAL_COOPERATE_DISATTACH   = 12  // Collaboration signal, user Disattach a component, currently broadcast only
```

## Option

Default is 0, optional, use int32 (32-bit signed integer), use bit by bit to determine if this option exists.

```go
const OPTION_BROADCAST_ROOM = 1 // 00000000000000000000000000000001; // use as signed int32 in typescript
```

## Request Target

```go
const TARGET_NOTNING            = 0 // placeholder
const TARGET_COMPONENTS         = 1 // ComponentsState
const TARGET_DEPENDENCIES       = 2 // DependenciesState
const TARGET_DRAG_SHADOW        = 3 // DragShadowState
const TARGET_DOTTED_LINE_SQUARE = 4 // DottedLineSquareState
const TARGET_DISPLAY_NAME       = 5 // DisplayNameState, note that displayName is a set, i.e. an unordered array, whose storage needs to be assembled on top of the K-V storage.
const TARGET_APPS               = 6 // broadcast only
const TARGET_RESOURCE           = 7 // broadcast only
const TARGET_ACTION             = 8 // broadcast only
```




# SIGNAL_PING

## Request Sample

```json
{
    "signal": 0,          
    "option": 0,          
    "target": 0,
    "payload": [],
    "broadcast": null // Client broadcast data, null by default
}
```
### Response Sample

```json
{
    "signal": 1,
    "message": "ok", 
    "data": []
}
```


# SINGNAL_ENTER

## Request Sample

```json
{
    "signal": 1,           // enter room
    "option": 0,           // no options
    "target": 0,           // no target
    "payload": [{          // 
        "authToken": ""    // user Authorization token
    }],
    "broadcast": {         // You need to add a broadcast message. Note that it must be set or it will not be broadcast.
        "type": "enter",
        "payload": []
    }
}
```

## Response Sample

- Other users in the same room will also receive the broadcast

```json
{
    "signal": 1,
    "message": "ok", 
    "data": [],
    "broadcast": {
        "type": "enter/remote", // Broadcast messages add a "/remote" to the end of the type value as a front-end termination broadcast token
        "payload": {
            "inRoomUsers": [  // Returns an array of json objects listing the users in the current room, which is sorted on the server by the time they entered.
                {         
                    "userID": "12",
                    "Nickname": "yourNickname",
                    "avatar": "https://cdn.illacloud.com/resource/sampleavatar.png"
                },
                {         
                    "userID": "14",
                    "Nickname": "jhon",
                    "avatar": "https://cdn.illacloud.com/resource/jhon.png"
                },
                ...
            ]
        }
    }       
}
```


# SINGNAL_LEAVE

## Request Sample

```json
{
    "signal": 2,           // leave
    "option": 0,           // no options
    "target": 0,           // no targets
    "payload": [],         // no payload
    "broadcast": {         // You need to add a broadcast message. Note that it must be set or it will not be broadcast.
        "type": "leave",
        "payload": []
    }
}
```

## Response Sample

Leaves the websocket directly disconnected, with no return message.


## Broadcast Sample

- After leaving the room, other users in the room will receive the broadcast.

```json
{
    "signal": 1,
    "message": "ok", 
    "data": [],
    "broadcast": {
        "type": "enter/remote", 
        "payload": {
            "inRoomUsers": [  
                {         
                    "userID": "12",
                    "Nickname": "yourNickname",
                    "avatar": "https://cdn.illacloud.com/resource/sampleavatar.png"
                },
                {         
                    "userID": "14",
                    "Nickname": "jhon",
                    "avatar": "https://cdn.illacloud.com/resource/jhon.png"
                },
                ...
            ]
        } 
    }      
}
```

# SIGNAL_CREATE_STATE



## Request 

- ComponentState

```json
{
    "signal": 3,           // create  state
    "option": 1,           
    "target": 1,           // target is components 
    "payload": [           
        { 
                
          displayName: string
          parentNode: string | null
          childrenNode: ComponentNode[]
          type: string
          ... 
        },
        ... // Can create multiple unrelated nodes or trees at once
    ],
    "broadcast": {
      "type": "", 
      "payload": ""
    }       
}
```

- displayNameState

```json
{
    "signal": 3, // create
    "target": 5, // displayName
    "option": 1,
    "payload": [
        "circleProgress1"
    ],
    "broadcast": {
        "type": "displayName/removeDisplayNameMultiReducer",
        "payload": [
            "circleProgress1",
            "circleProgress2",
        ]
    }
}
```


# SIGNAL_DELETE_STATE

## Request 

```json
{
    "signal": 4,           // delete state
    "option": 1,           // 00000000000000000000000000000001, need broadcast
    "target": 1,           // it is components 
    "payload": [           // If the payload is more than one displayName, all subtrees under those displayNames will be removed.
        {
          displayName: string // only displayName
        },
        ... // You can delete more than one at a time
    ],
    "broadcast": {
      "type": "", 
      "payload": ""
    }       
}
```

- displayNameState

```json
{
    "signal": 4, // delete
    "option": 1,
    "target": 5, // displayName
    "payload": [
        "circleProgress1",
        "circleProgress2",
    ],
    "broadcast": {
        "type": "displayName/removeDisplayNameMultiReducer",
        "payload": [
            "circleProgress1"
        ]
    }
}
```

# SIGNAL_UPDATE_STATE


## Request 

- components

```json
{
    "signal": 5,           // update state
    "option": 1,           // 00000000000000000000000000000001, need broadcast
    "target": 1,           // it is components 
    "payload": [           // Object array, objects have before, after attributes.
        {
            "before":{
              displayName: string //before is used to query the database for the update target, just pass the displayName.
            },
            "after":{
              displayName: string,
              parentNode: string | null,
              childrenNode: ComponentNode[],
              ... // The rest of the fields should be included, whether they change or not, because the database is serialized.
            }
        },
        ... // You can delete more than one at a time
    ],
    "broadcast": {
      "type": "", 
      "payload": ""
    }               
}
```

- displayNameState

```json
{
    "signal": 5, // update state
    "option": 1,
    "target": 5, // displayName
    "payload": [
      {
          "before":"circleProgress1", // Needs to be updated as before & after
          "after":"circleProgress2"
      },
      ... // It is possible to update more than one at a time, but each before & after needs to be separated.
    ],
    "broadcast": {
        "type": "displayName/removeDisplayNameMultiReducer",
        "payload": [
            "circleProgress1"
        ]
    }
}
```


# SIGNAL_MOVE_STATE


## Request 

Only tree state can perform move operation, k-v state has no move operation.

```json
{
    "signal": 6,           // move state
    "option": 1,           // 00000000000000000000000000000001, need broadcast
    "target": 1,           // it is components 
    "payload": [           // Only the displayName, parentNode, and childrenNode of the node being moved need to be transmitted.
      {
          displayName: string // Need displayName, parentNode, and childrenNode, these data are always after the change, the server can get the data from the database according to the displayName before the change.
          parentNode: string | null
          childrenNode: ComponentNode[]
      },
        ... // You can move more than one at a time, regardless of whether they are on the same level or under a container. This design avoids the problem of retool not being able to multi-select across hierarchies.
    ],
    "broadcast": {
      "type": "", 
      "payload": ""
    }               
}
```


# SIGNAL_CREATE_OR_UPDATE_STATE

## Request 

- componentState

```json
{
    "signal": 7,    
    "option": 1,    
    "target": 1,    
    "payload": [    
        {
            "displayName": "cnode12",
            "parentNode": "",
            "showName": "",
            "error": false,
            "isDragging": false,
            "childrenNode": [],
            "type": "",
            "containerType": null,
            "verticalResize": false,
            "h": 100,
            "w": 200,
            "minH": 50,
            "minW": 50,
            "x": 120,
            "y": 130,
            "z": 140,
            "props": null,
            "panelConfig": null
        }
    ],
    "broadcast": {
        "type": "type/broadcast",
        "payload": "payload/broadcast"
    }           
}
```

# SIGNAL_COOPERATE_ATTACH

- Which components the current user is attached to
- This data is also returned when the user enters the room for the first time.
  
## Request 


```json
{
    "signal": 11,    
    "option": 1,    
    "target": 1,    
    "payload": [ // payload is a list of display names for the component
        "cnode12", "text13", ...
    ],
    "broadcast": {
        "type": "attachComponent/remote",
        "payload": []
    }           
}
```

- Alternatively, if you want to actively get the attach list, you can send a request with an empty payload, which will be sent automatically when you enter the room.

```json
{
    "signal": 11,    
    "option": 1,    
    "target": 1,    
    "payload": [ 
    ],
    "broadcast": {
        "type": "attachComponent",
        "payload": []
    }           
}
```

## Response (Other users will also receive the broadcast) 

```json
{
    "signal": 11,
    "message": "ok", 
    "target": 1,    
    "data": [],
    "broadcast": {
        "type": "componentAttachedUsers/remote", 
        "payload": {
            "componentAttachedUsers":{
                "componentDispalyName-A": [
                    {         
                        "userID": "12",
                        "Nickname": "yourNickname",
                        "avatar": "https://cdn.illacloud.com/resource/sampleavatar.png"
                    },
                    ...
                ],
                "componentDispalyName-B": [
                    {         
                        "userID": "12",
                        "Nickname": "yourNickname",
                        "avatar": "https://cdn.illacloud.com/resource/sampleavatar.png"
                    },
                    ...
                ]
            }
        } 
    }      
}
```

# SIGNAL_COOPERATE_DISATTACH

- Which components the current user is disattaching from.

## Request 


```json
{
    "signal": 12,    
    "option": 1,    
    "target": 1,    
    "payload": [ 
        "cnode12", "text13", ...
    ],
    "broadcast": {
        "type": "attachComponent/remote",
        "payload": []
    }           
}
```

## Response (Other users will also receive the broadcast) 

```json
{
    "signal": 11,
    "message": "ok", 
    "target": 1,    
    "data": [],
    "broadcast": {
        "type": "componentAttachedUsers/remote", 
        "payload": {
            "componentAttachedUsers":{
                "componentDispalyName-A": [
                    {         
                        "userID": "12",
                        "Nickname": "yourNickname",
                        "avatar": "https://cdn.illacloud.com/resource/sampleavatar.png"
                    },
                    ...
                ],
                "componentDispalyName-B": [
                    {         
                        "userID": "12",
                        "Nickname": "yourNickname",
                        "avatar": "https://cdn.illacloud.com/resource/sampleavatar.png"
                    },
                    ...
                ]
            }
        } 
    }      
}
```


# Response Protocol Format

## Sample

```json
{
    "errorCode": 0,        
    "errorMessage": "ok",  
    "broadcast": {         
      "type": "",         
      "payload": ""
      },   
    "data":[]              
}
```

## Response Signal Const

Used to define the result of the execution of the protocol request.

```go
const ERROR_CODE_OK = 0
const ERROR_CODE_FAILED = 1
const ERROR_CODE_NEED_ENTER = 2
const ERROR_CODE_BROADCAST = 0
const ERROR_CODE_PONG = 3
const ERROR_CODE_LOGGEDIN = 0
const ERROR_CODE_LOGIN_FAILED = 4
const ERROR_CREATE_STATE_OK = 0
const ERROR_CREATE_STATE_FAILED = 5
const ERROR_DELETE_STATE_OK = 0
const ERROR_DELETE_STATE_FAILED = 6
const ERROR_UPDATE_STATE_OK = 0
const ERROR_UPDATE_STATE_FAILED = 7
const ERROR_MOVE_STATE_OK = 0
const ERROR_MOVE_STATE_FAILED = 8
const ERROR_CREATE_OR_UPDATE_STATE_OK = 0
const ERROR_CREATE_OR_UPDATE_STATE_FAILED = 9
const ERROR_CAN_NOT_MOVE_KVSTATE = 10
const ERROR_CAN_NOT_MOVE_SETSTATE = 11

```


## Data Structure used by Frontend

```json
{
    "appInfo": {
        "appId": 15,
        "appName": "test",
        "release_version": 12,
        "mainline_version": 12,
        "updatedAt": "2022-07-15T15:56:00.28703Z",
        "updatedBy": "smallSohoSolo"
    },
    "actions": [
        {
            "actionId": 45,
            "displayName": "transformer1",
            "actionType": "transformer",
            "createdAt": "2022-07-15T16:06:50.69252Z",
            "createdBy": 1,
            "updatedAt": "2022-07-15T16:06:50.692521Z",
            "updatedBy": 1
        },
        {
            "actionId": 48,
            "resourceId": 10,
            "displayName": "mysql3",
            "actionType": "mysql",
            "actionTemplate": {
                "mode": "sql",
                "query": ""
            },
            "createdAt": "2022-07-15T16:10:44.336918Z",
            "createdBy": 1,
            "updatedAt": "2022-07-15T16:10:44.336918Z",
            "updatedBy": 1
        }
    ],
    "components": {
        "rootDsl": {
            "displayName": "root",
            "parentNode": "",
            "showName": "root",
            "error": false,
            "isDragging": false,
            "isResizing": false,
            "childrenNode": [],
            "type": "DOT_PANEL",
            "containerType": "EDITOR_DOT_PANEL",
            "verticalResize": true,
            "h": 0,
            "w": 0,
            "minH": 0,
            "minW": 0,
            "x": -1,
            "y": -1,
            "z": 0,
            "props": null,
            "panelConfig": null
        }
    },
    "dependenciesState": {},
    "dragShadowState": {},
    "dottedLineSquareState": {},
    "displayNameState": []
}
```



# Request Sample


```json
{
    "signal": 3,
    "target": 1,
    "option": 1,
    "payload": [
        {
            "w": 16,
            "h": 5,
            "minH": 3,
            "minW": 1,
            "verticalResize": false,
            "isDragging": true,
            "isResizing": false,
            "error": false,
            "x": 17,
            "y": 25,
            "z": 0,
            "showName": "radioButton",
            "type": "RADIO_BUTTON_WIDGET",
            "displayName": "radioButton1",
            "containerType": "EDITOR_SCALE_SQUARE",
            "parentNode": "root",
            "childrenNode": [],
            "props": {
                "optionConfigureMode": "static",
                "label": "Label",
                "labelAlign": "left",
                "labelPosition": "left",
                "labelWidth": "{{33}}",
                "manualOptions": [
                    {
                        "id": "option-9ff663d1-a270-4321-899e-b6e49c610a92",
                        "label": "Option 1",
                        "value": "Option 1"
                    },
                    {
                        "id": "option-b0d5b0e5-7916-477c-bf72-7aaf8c0072d6",
                        "label": "Option 2",
                        "value": "Option 2"
                    },
                    {
                        "id": "option-e907849d-58ea-482c-af0f-3de809f5b398",
                        "label": "Option 3",
                        "value": "Option 3"
                    }
                ],
                "dataSources": "{{[]}}",
                "colorScheme": "blue",
                "hidden": false
            }
        }
    ],
    "broadcast": {
        "type": "components/addComponentReducer",
        "payload": {
            "w": 16,
            "h": 5,
            "minH": 3,
            "minW": 1,
            "verticalResize": false,
            "isDragging": true,
            "isResizing": false,
            "error": false,
            "x": 17,
            "y": 25,
            "z": 0,
            "showName": "radioButton",
            "type": "RADIO_BUTTON_WIDGET",
            "displayName": "radioButton1",
            "containerType": "EDITOR_SCALE_SQUARE",
            "parentNode": "root",
            "childrenNode": [],
            "props": {
                "optionConfigureMode": "static",
                "label": "Label",
                "labelAlign": "left",
                "labelPosition": "left",
                "labelWidth": "{{33}}",
                "manualOptions": [
                    {
                        "id": "option-9ff663d1-a270-4321-899e-b6e49c610a92",
                        "label": "Option 1",
                        "value": "Option 1"
                    },
                    {
                        "id": "option-b0d5b0e5-7916-477c-bf72-7aaf8c0072d6",
                        "label": "Option 2",
                        "value": "Option 2"
                    },
                    {
                        "id": "option-e907849d-58ea-482c-af0f-3de809f5b398",
                        "label": "Option 3",
                        "value": "Option 3"
                    }
                ],
                "dataSources": "{{[]}}",
                "colorScheme": "blue",
                "hidden": false
            }
        }
    }
}
```

```json
{
    "signal": 3,
    "target": 1,
    "option": 1,
    "payload": [
        {
            "w": 6,
            "h": 12,
            "minH": 3,
            "minW": 1,
            "verticalResize": false,
            "isDragging": true,
            "isResizing": false,
            "error": false,
            "x": 41,
            "y": 10,
            "z": 0,
            "showName": "circleProgress",
            "type": "CIRCLE_PROGRESS_WIDGET",
            "displayName": "circleProgress1",
            "containerType": "EDITOR_SCALE_SQUARE",
            "parentNode": "root",
            "childrenNode": [],
            "props": {
                "value": "50",
                "alignment": "center",
                "color": "blue",
                "trailColor": "gray",
                "strokeWidth": "4px"
            }
        }
    ],
    "broadcast": {
        "type": "components/addComponentReducer",
        "payload": {
            "w": 6,
            "h": 12,
            "minH": 3,
            "minW": 1,
            "verticalResize": false,
            "isDragging": true,
            "isResizing": false,
            "error": false,
            "x": 41,
            "y": 10,
            "z": 0,
            "showName": "circleProgress",
            "type": "CIRCLE_PROGRESS_WIDGET",
            "displayName": "circleProgress1",
            "containerType": "EDITOR_SCALE_SQUARE",
            "parentNode": "root",
            "childrenNode": [],
            "props": {
                "value": "50",
                "alignment": "center",
                "color": "blue",
                "trailColor": "gray",
                "strokeWidth": "4px"
            }
        }
    }
}
```

```json
{
    "signal": 4,
    "target": 5,
    "option": 1,
    "payload": [
        "circleProgress1"
    ],
    "broadcast": {
        "type": "displayName/removeDisplayNameMultiReducer",
        "payload": [
            "circleProgress1"
        ]
    }
}
```

```json
{
    "signal": 4,
    "target": 5,
    "option": 1,
    "payload": [
        "text1"
    ],
    "broadcast": {
        "type": "displayName/removeDisplayNameMultiReducer",
        "payload": [
            "text1"
        ]
    }
}
```

```json
{
    "signal": 4,
    "target": 1,
    "option": 1,
    "payload": [
        "text1"
    ],
    "broadcast": {
        "type": "components/deleteComponentNodeReducer",
        "payload": {
            "displayNames": [
                "text1"
            ]
        }
    }
}
```

```json
{
    "signal": 5,
    "target": 2,
    "option": 1,
    "payload": [
        {}
    ],
    "broadcast": {
        "type": "dependencies/setDependenciesReducer",
        "payload": {}
    }
}
```


```json
{
    "signal": 7,
    "target": 3,
    "option": 1,
    "payload": [
        {
            "parentNode": "root",
            "displayName": "text1",
            "renderX": 778.6875,
            "renderY": 77,
            "w": 194.625,
            "h": 40,
            "isConflict": false
        }
    ],
    "broadcast": {
        "type": "dragShadow/addOrUpdateDragShadowReducer",
        "payload": {
            "parentNode": "root",
            "displayName": "text1",
            "renderX": 778.6875,
            "renderY": 77,
            "w": 194.625,
            "h": 40,
            "isConflict": false
        }
    }
}
```
