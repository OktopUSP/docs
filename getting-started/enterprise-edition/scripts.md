---
description: >-
  Automate your workflow with tasks based on events, jobs, schedules, and more.
  Limitless capabilities with custom resources to fit your needs.
cover: >-
  https://images.unsplash.com/photo-1515879218367-8466d910aaa4?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwxfHxjb2RlfGVufDB8fHx8MTczMjI5NjY0OXww&ixlib=rb-4.0.3&q=85
coverY: 0
---

# Scripts

## Introduction

Code is wrritten in **Lua**, which is a lightweight, high-level programming language designed for embedded use, known for its simplicity and efficiency. Created in Brazil in 1993, it features dynamic typing, first-class functions, and a powerful table data structure, making it versatile for various applications. Lua is widely used in game development for scripting game logic, as well as in embedded systems and applications due to its **extensibility and small footprint**, allowing developers to easily integrate it with other languages and platforms.

[LUA 5.1 Manual](https://www.lua.org/manual/5.1/)&#x20;

### How It Works

To extend Oktopus capabilities and adress the most diverse possible use cases we opted to use a scripting language on top of the actual code, so the [gopher-lua](https://github.com/yuin/gopher-lua) library provides Go APIs that allows to easily embed LUA scripts to Go programs.

The Go software can interact to the LUA script and vice-versa. That way, it's possible to pass functions, parameters and events through both USP Controller/ACS and the user created automations.

### Core Concepts

A "custom function", is as a function called from LUA which translates to a Go function that can interact with NATS, MongoDB, USP Controller, TR-069 ACS and all the other components of the software stack.

The "custom functions" will be detailed above in the next topics as just "functions" and separated into domain areas. We hope to provide usefull examples, and the limitation is on each person creativity.&#x20;

## Functions

### send\_cwmp\_message()

#### Params:

1.  Serial Number \[string]

    The CPE unique identifier.
2.  XML payload \[string]

    Request body to be sent to the CPE.
3.  Message Type \[integer]

    CWMP message type. \
    0 = getParameterValues\
    1 = setParameterValues

Return:

The data returned depends on the TR-069 message type. If it is a read operation as "getParameterValues" than the function will return a table. If it is a create/delete/set operation the data returned will be a boolean indicating if the result was a success or failure.

Example:

{% tabs %}
{% tab title="GetParameterValues" %}
```lua
local getParameterValuesType = 0

local xmlContentGet = [[
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:cwmp="urn:dslforum-org:cwmp-1-0" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:schemaLocation="urn:dslforum-org:cwmp-1-0 ..\schemas\wt121.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <soap:Header/>
  <soap:Body soap:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <cwmp:GetParameterValues>
      <ParameterNames>
        <string>InternetGatewayDevice.LANDevice.1.WLANConfiguration.1.</string>
       </ParameterNames>
    </cwmp:GetParameterValues>
  </soap:Body>
</soap:Envelope>
]]

local getTest = send_cwmp_message("HUAWNFYC-35454645", xmlContentGet, getParameterValuesType)
if getTest ~= nil then
  for key, value in pairs(getTest) do
    print(key .. ": " .. value)
  end
else
  print("GetParameterValues failed")
end
```
{% endtab %}

{% tab title="SetParameterValues" %}
```lua
local setParameterValuesType = 1

local xmlContentSet = [[
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:cwmp="urn:dslforum-org:cwmp-1-0" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:schemaLocation="urn:dslforum-org:cwmp-1-0 ..\schemas\wt121.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <soap:Header/>
  <soap:Body soap:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <cwmp:SetParameterValues>
      <ParameterList>
        <ParameterValueStruct>
          <Name>InternetGatewayDevice.LANDevice.1.WLANConfiguration.1.Enable</Name>
          <Value>1</Value>
        </ParameterValueStruct>
      </ParameterList>
    </cwmp:SetParameterValues>
  </soap:Body>
</soap:Envelope>
]]

local setTest = send_cwmp_message("HUAWNFYC-35454645", xmlContentSet, setParameterValuesType)
if setTest then
  print("SetParameterValues worked")
else
  print("SetParameterValues failed")
end
```
{% endtab %}
{% endtabs %}

### listen\_to\_cwmp\_event()

Params:

1.  Event \[string] (optional)\
    Possible events:

    ```
    "0"- BOOTSTRAP
    "1"- BOOT
    "2"- PERIODIC
    "3"- SCHEDULED
    "4"- VALUE CHANGE
    "5"- KICKED
    "6"- CONNECTION REQUEST
    "7"- TRANSFER COMPLETE
    "8"- DIAGNOSTICS COMPLETE
    "9"- REQUEST DOWNLOAD
    "10"- AUTONOMOUS TRANSFER COMPLETE
    "11"- DU STATE CHANGE COMPLETE
    "12"- AUTONOMOUS DU STATE CHANGE COMPLETE
    ```

    If no value is set, than it listens to all events.
2. Serial Number \[string] (optional)\
   CPE unique identifier. If it's not set then it listens to all CPEs event(s).

Return:

```json
{
    "serial_number": "HUAWNFYC-35454645",
    "event": "0"
}
```

Example:

```lua
while true do
  local cwmp_event = listen_to_cwmp_event("2")
  print("CWMP event " .. cwmp_event.event .. ", triggered by device: " .. cwmp_event.serial_number)
end
```

### get\_device()

#### Params:

1.  Serial Number \[string]

    The CPE unique identifier.

Return:

If the CPE was not found in the database the function returns a boolean with false value. In case the CPE is found it returns a table with all attributes:

```json
{
    "sn": "HUAWNFYC-35454645",
    "model": "WS7001-40",
    "vendor": "Huawei Technologies Co., Ltd.",
    "version": "",
    "product_class": "Huawei",
    "alias": "",
    "status": 2
}
```

Example:

```lua
SERIAL_NUMBER = "HUAWNFYC-35454645"
local deviceAttributes = get_device(SERIAL_NUMBER)
if type(deviceAttributes) == "boolean" then
  print("device " .. SERIAL_NUMBER .. " not found")
else
  for key, value in pairs(deviceAttributes) do
    print(key .. ": " .. value)
  end
end
```





















\




