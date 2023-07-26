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
  - [Response (其他用户也会收到该广播)](#response-其他用户也会收到该广播)
- [SIGNAL\_COOPERATE\_DISATTACH](#signal_cooperate_disattach)
  - [Request](#request-6)
  - [Response (其他用户也会收到该广播)](#response-其他用户也会收到该广播-1)
- [Response Protocol Format](#response-protocol-format)
  - [Sample](#sample)
  - [Response Signal Const](#response-signal-const)
  - [Data Structure used by Frontend](#data-structure-used-by-frontend)
- [Request Sample](#request-sample-3)



# Consts


## Request Signal 

-  用于定义协议要执行的操作. 默认为0, 代表 ping 操作.

```go
const SIGNAL_PING                  = 0   // ping, 调试用
const SIGNAL_ENTER                 = 1   // 进入
const SIGNAL_LEAVE                 = 2   // 离开
const SIGNAL_CREATE_STATE          = 3   // 新建 state
const SIGNAL_DELETE_STATE          = 4   // 删除 state
const SIGNAL_UPDATE_STATE          = 5   // 更新 state
const SIGNAL_MOVE_STATE            = 6   // 移动 state
const SIGNAL_CREATE_OR_UPDATE      = 7   // 创建或者更新 state
const SIGNAL_ONLY_BROADCAST        = 8   // 只广播
const SIGNAL_PUT_STATE             = 9   // PUT, 全量更新
const SIGNAL_GLOBAL_BROADCAST_ONLY = 10  // 全局广播, 当前实例下的所有客户端均会收到广播, 适用于广播app, resource 发生变化的场景.
const SIGNAL_COOPERATE_ATTACH      = 11  // 协作 signal, 用户 Attach 到某个组件上, 目前只广播
const SIGNAL_COOPERATE_DISATTACH   = 12  // 协作 signal, 用户 Disattach 某个组件, 目前只广播
```

## Option

默认为0, 可选, 使用 int32 (32位有符号整数表示), 使用中根据 bit 按位与来判断该选项是否存在.


```go
const OPTION_BROADCAST_ROOM = 1 // 00000000000000000000000000000001; // use as signed int32 in typescript
```

## Request Target

```go
const TARGET_NOTNING            = 0 // 占位使用
const TARGET_COMPONENTS         = 1 // ComponentsState
const TARGET_DEPENDENCIES       = 2 // DependenciesState
const TARGET_DRAG_SHADOW        = 3 // DragShadowState
const TARGET_DOTTED_LINE_SQUARE = 4 // DottedLineSquareState
const TARGET_DISPLAY_NAME       = 5 // DisplayNameState, 注意, displayName 是个 set, 即无序数组, 其存储方式在 K-V 存储的基础上需要进行拼装.
const TARGET_APPS               = 6 // 目前只广播, 没有后端逻辑
const TARGET_RESOURCE           = 7 // 目前只广播, 没有后端逻辑
const TARGET_ACTION             = 8 // 目前只广播, 没有后端逻辑
```




# SIGNAL_PING

## Request Sample

```json
{
    "signal": 0,          
    "option": 0,          
    "target": 0,
    "payload": [],
    "broadcast": null // 客户端广播数据, 默认为 null
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
    "signal": 1,           // 进入
    "option": 0,           // 无操作选项
    "target": 0,           // 无需要操作的目标, 默认为 0
    "payload": [{          // 登录信息
        "authToken": ""    // 用户登录验证 token
    }],
    "broadcast": {         // 需要增加广播信息, 注意, 一定要设置, 否则不广播
        "type": "enter",
        "payload": []
    }
}
```

## Response Sample

- 其他在同一房间的用户也会收到该广播

```json
{
    "signal": 1,
    "message": "ok", 
    "data": [],
    "broadcast": {
        "type": "enter/remote", // 广播消息会在 type 值的最后面增加一个 /remote 作为前端的终止广播标记
        "payload": {
            "inRoomUsers": [  // 返回当前房间的用户列表的 json 对象数组, 服务器端会根据 enter 的时间进行倒排
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
    "signal": 2,           // 离开
    "option": 0,           // 无
    "target": 0,           // 无
    "payload": [],          // 无
    "broadcast": {         // 需要增加广播信息, 注意, 一定要设置, 否则不广播
        "type": "leave",
        "payload": []
    }
}
```

## Response Sample

离开直接断开 websocket, 无返回信息


## Broadcast Sample

- 离开房间后, 其他在这个房间的用户会收到该广播

```json
{
    "signal": 1,
    "message": "ok", 
    "data": [],
    "broadcast": {
        "type": "enter/remote", // 广播消息会在 type 值的最后面增加一个 /remote 作为前端的终止广播标记
        "payload": {
            "inRoomUsers": [  // 返回当前房间的用户列表的 json 对象数组, 服务器端会根据 enter 的时间进行倒排
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
    "signal": 3,           // 创建  state
    "option": 1,           // 00000000000000000000000000000001, 需要广播
    "target": 1,           // 创建的是 component 
    "payload": [           
        { 
                
          displayName: string
          parentNode: string | null
          childrenNode: ComponentNode[]
          type: string
          ... // 全部字段都要传
        },
        ... // 可以一次创建多个不相关联的节点或者树
    ],
    "broadcast": {
      "type": "", // 广播消息会在 type 值的最后面增加一个 /remote 作为前端的终止广播标记
      "payload": ""
    }       // 客户端广播数据, 默认为 null    
}
```

- displayNameState

```json
{
    "signal": 3, // 删除 
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
    "signal": 4,           // 删除  state
    "option": 1,           // 00000000000000000000000000000001, 需要广播
    "target": 1,           // 创建的是 component 
    "payload": [           // payload 是多个 displayName, 会删除这些 displayName 对应节点下的全部子树.
        {
          displayName: string // 只需要 displayName
        },
        ... // 可以一次删除多个
    ],
    "broadcast": {
      "type": "", // 广播消息会在 type 值的最后面增加一个 /remote 作为前端的终止广播标记
      "payload": ""
    }       // 客户端广播数据, 默认为 null    
}
```

- displayNameState

```json
{
    "signal": 4, // 删除
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
    "signal": 5,           // 更新 state
    "option": 1,           // 00000000000000000000000000000001, 需要广播
    "target": 1,           // 创建的是 component 
    "payload": [           // 对象数组, 对象有 before, after 两个属性.
        {
            "before":{
              displayName: string // before 中用来查询数据库中的update目标, 只传 displayName 即可.
            },
            "after":{
              displayName: string,
              parentNode: string | null,
              childrenNode: ComponentNode[],
              ... // 其余字段也要都带上, 不管是否变化, 因为数据库是序列化存储的.
            }
        },
        ... // 可以一次更新多个
    ],
    "broadcast": {
      "type": "", // 广播消息会在 type 值的最后面增加一个 /remote 作为前端的终止广播标记
      "payload": ""
    }       // 客户端广播数据, 默认为 null             
}
```

- displayNameState

```json
{
    "signal": 5, // 更新
    "option": 1,
    "target": 5, // displayName
    "payload": [
      {
          "before":"circleProgress1", // 需要以before & after 的形式进行更新
          "after":"circleProgress2"
      },
      ... // 可以一次更新多个, 但每一个 before & after 需要单独分开.
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

只有 tree state 可以执行 move 操作, k-v state 无 move 操作.

```json
{
    "signal": 6,           // 移动 state
    "option": 1,           // 00000000000000000000000000000001, 需要广播
    "target": 1,           // 创建的是 component 
    "payload": [           // 只需要传输 移动的节点的 displayName, parentNode, 和 childrenNode
      {
          displayName: string // 需要 displayName, parentNode, 和 childrenNode, 这些数据永远是变更后的数据, 变更前的数据服务端可以根据 displayName 从数据库拿到.
          parentNode: string | null
          childrenNode: ComponentNode[]
      },
        ... // 可以一次移动多个, 不管是否位于同一个层级或者 container 下. 这样设计避免了 retool 不能跨层级多选的问题.
    ],
    "broadcast": {
      "type": "", // 广播消息会在 type 值的最后面增加一个 /remote 作为前端的终止广播标记
      "payload": ""
    }       // 客户端广播数据, 默认为 null           
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

- 当前用户 Attach 到哪些 component 上
- 在用户初次进入 room 的时候也会返回该数据
  
## Request 


```json
{
    "signal": 11,    
    "option": 1,    
    "target": 1,    
    "payload": [ // payload 为 component 的 display name 列表
        "cnode12", "text13", ...
    ],
    "broadcast": {
        "type": "attachComponent/remote",
        "payload": []
    }           
}
```

- 另外, 如果需要主动获取 attach 列表, 可以发一个 payload 为空的请求, enter room 的时候会自动发.

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

## Response (其他用户也会收到该广播) 

```json
{
    "signal": 11,
    "message": "ok", 
    "target": 1,    
    "data": [],
    "broadcast": {
        "type": "componentAttachedUsers/remote", // 广播消息会在 type 值的最后面增加一个 /remote 作为前端的终止广播标记
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

- 当前用户从哪些 component 上 disattach.

## Request 


```json
{
    "signal": 12,    
    "option": 1,    
    "target": 1,    
    "payload": [ // payload 为 component 的 display name 列表
        "cnode12", "text13", ...
    ],
    "broadcast": {
        "type": "attachComponent/remote",
        "payload": []
    }           
}
```

## Response (其他用户也会收到该广播) 

```json
{
    "signal": 11,
    "message": "ok", 
    "target": 1,    
    "data": [],
    "broadcast": {
        "type": "componentAttachedUsers/remote", // 广播消息会在 type 值的最后面增加一个 /remote 作为前端的终止广播标记
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
    "errorCode": 0,        // 状态
    "errorMessage": "ok",  // 错误信息
    "broadcast": {         // 客户端定义的 广播信息, 默认为 null
      "type": "",          // 广播消息会在 type 值的最后面增加一个 /remote 作为前端的终止广播标记
      "payload": ""
      },   
    "data":[]              // 返回数据, 默认为空数组
}
```

## Response Signal Const

用于定义协议请求的执行结果结果.

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
