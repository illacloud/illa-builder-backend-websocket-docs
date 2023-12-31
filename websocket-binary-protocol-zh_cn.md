ILLA Builder Backend WebSocket Binary Protocols
------------------------------------------------


# Desc 

二进制 WebSocket 协议目前只用于同步鼠标指针轨迹和 Components 的移动.


# Index
- [Desc](#desc)
- [Index](#index)
- [Protocol Buffer Design](#protocol-buffer-design)


# Protocol Buffer Design

```pb
syntax = "proto3";
package tutorial;

// [START go_declaration]
option go_package = "./ws";
// [END go_declaration]

enum Signal {
  SIGNAL_PING = 0;
  SIGNAL_ENTER = 1;
  SIGNAL_LEAVE = 2;
  SIGNAL_CREATE_STATE = 3;
  SIGNAL_DELETE_STATE = 4;
  SIGNAL_UPDATE_STATE = 5;
  SIGNAL_MOVE_STATE = 6;
  SIGNAL_CREATE_OR_UPDATE_STATE = 7;
  SIGNAL_BROADCAST_ONLY = 8;
  SIGNAL_PUT_STATE = 9;
  SIGNAL_GLOBAL_BROADCAST_ONLY = 10;
  SIGNAL_COOPERATE_ATTACH = 11;
  SIGNAL_COOPERATE_DISATTACH = 12;
  SIGNAL_MOVE_CURSOR = 13;
}

enum Target {
  TARGET_NOTHING = 0;            // placeholder for nothing
  TARGET_COMPONENTS = 1;         // ComponentsState
  TARGET_DEPENDENCIES = 2;       // DependenciesState
  TARGET_DRAG_SHADOW = 3;        // DragShadowState
  TARGET_DOTTED_LINE_SQUARE = 4; // DottedLineSquareState
  TARGET_DISPLAY_NAME = 5;       // DisplayNameState
  TARGET_APPS = 6;               // only for broadcast
  TARGET_RESOURCE = 7;           // only for broadcast
  TARGET_ACTION = 8;             // only for broadcast
  TARGET_CURSOR = 9;             // only for broadcast
}

message MovingMessageBin {
  Signal  signal            = 1;  // see package ws const define
  Target  target            = 2;  // 
  string  clientID          = 3;  // message sender client ID
  bool    needBroadcast     = 4;
  string  userID            = 5;  // message sender user ID
  string  nickname          = 6;  // message sender user ID
  int32   status            = 7;  // set instance to status, 0 for empty status
  string  parentDisplayName = 8;  // parent component displayname
  string  displayNames      = 9;  // message affected component display names, separate with comma ","
  uint32  cursorXInteger    = 10; // cursor's position with dot number
  uint32  cursorYInteger    = 11; // cursor's position with dot number
  float   cursorXMod        = 12; // cursor's position with dot number
  float   cursorYMod        = 13; // cursor's position with dot number
  uint32  widgetX           = 14; // widget'position with dot number
  uint32  widgetY           = 15; // widget'position with dot number
  uint32  widgetW           = 16; // widget'shape with dot number
  uint32  widgetH           = 17; // widget'shape with dot number
}
// [END messages]


```
