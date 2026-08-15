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

<figure><img src="../../.gitbook/assets/lua-interaction.png" alt=""><figcaption><p>Diagram of Lua Scripts</p></figcaption></figure>

The "custom functions" will be detailed above in the next topics as just "functions" and separated into domain areas. We hope to provide usefull examples, and the limitation is on each person creativity.&#x20;

## Functions

### send\_usp\_message()

#### Params:

1.  Serial Number \[string]

    The CPE unique identifier.
2.  JSON payload \[string]

    Request body to be sent to the CPE.\
    Current supported bodies are : \
    \- Get\
    \- Set

Return:

* If the usp message generates an error:

```json
{
    "error_message": "error to set param test.software",
    "error_code: "500"
}
```

* In the case of a successful transaction each key of the _table_ will correspond to one of the parameters requested in the payload.

Example:

{% tabs %}
{% tab title="Get" %}
```lua
local uspMsg = [[
{
    "header": {
        "msg_id": "b7dc38ea-aefb-4761-aa55-edaa97adb2f0",
        "msg_type": 1
    },
    "body": {
        "request": {
            "get": {
                "paramPaths": [
                    "Device.STOMP."
                ],
                "maxDepth": 2
            }
        }
    }
}
]]

local serial_number = "oktopus-0-stomp"

local uspMessageResult = send_usp_message(serial_number, uspMsg)
if uspMessageResult["error_message"] ~= nil then
    print("SN: " .. serial_number .. " | Error Message: " .. uspMessageResult["error_message"] .. " | Code: " .. uspMessageResult["error_code"])
else
    for key, value in pairs(uspMessageResult) do
      print(key .. ": " .. value)
    end
end
```


{% endtab %}

{% tab title="Set" %}
```lua
local uspMsg = [[
{
    "header": {
        "msg_id": "b7dc38ea-aefb-4761-aa55-edaa97adb2f0",
        "msg_type": 4
    },
    "body": {
        "request": {
            "set": {
                "allow_partial":true,
                "update_objs":[
                    {
                        "obj_path":"Device.LocalAgent.",
                        "param_settings":[
                            {
                            "param":"X_VANTIVA-COM_PreConnectTimeout",
                            "value": "55",
                            "required":true
                            }
                        ]
                    }
                ]
            }
        }
    }
}
]]

local serial_number = "oktopus-0-stomp"

local uspMessageResult = send_usp_message(serial_number, uspMsg)
if uspMessageResult["error_message"] ~= nil then
    print("SN: " .. serial_number .. " | Error Message: " .. uspMessageResult["error_message"] .. " | Code: " .. uspMessageResult["error_code"])
else
    for key, value in pairs(uspMessageResult) do
      print(key .. ": " .. value)
    end
end
```


{% endtab %}

{% tab title="Delete" %}
```lua
local uspMsg = [[
{
    "header": {
        "msg_id": "b7dc38ea-aefb-4761-aa55-edaa97adb2f0",
        "msg_type": 10
    },
    "body": {
        "request": {
            "delete": {
                "allow_partial": true,
                "obj_paths": [
                    "Device.LocalAgent.Subscription.*."
                ]
            }
        }
    }
}
]]

local serial_number = "oktopus-0-mqtt"

local uspMessageResult = send_usp_message(serial_number, uspMsg)
if uspMessageResult["error_message"] ~= nil then
    print("SN: " .. serial_number .. " | Error Message: " .. uspMessageResult["error_message"] .. " | Code: " .. uspMessageResult["error_code"])
else
    for key, _ in pairs(uspMessageResult) do
      print(key)
    end
end
```
{% endtab %}

{% tab title="Add" %}
```lua
local uspMsg =[[
{
    "header": {
        "msg_id": "b7dc38ea-aefb-4761-aa55-edaa97adb2f0",
        "msg_type": 8
    },
    "body": {
        "request": {
            "add": {
                "allow_partial": true,
                "create_objs": [
                    {
                        "obj_path": "Device.LocalAgent.Subscription.",
                        "param_settings": [
                            {
                                "param": "NotifType",
                                "value": "OperationComplete",
                                "required": true
                            },
                            {
                                "param": "Persistent",
                                "value": "1",
                                "required": true
                            },
                            {
                                "param": "ReferenceList",
                                "value": "Device.IP.Diagnostics.DownloadDiagnostics(),Device.IP.Diagnostics.UploadDiagnostics()",
                                "required": true
                            },
                            {
                                "param": "NotifRetry",
                                "value": "0",
                                "required": true
                            },
                            {
                                "param": "Enable",
                                "value": "1",
                                "required": true
                            },
                          	{
                                "param": "Recipient",
                                "value": "Device.LocalAgent.Controller.1",
                                "required": true
                            }
                        ]
                    }
                ]
            }
        }
    }
}
]]

local serial_number = "oktopus-0-mqtt"

local uspMessageResult = send_usp_message(serial_number, uspMsg)
if uspMessageResult["error_message"] ~= nil then
    print("SN: " .. serial_number .. " | Error Message: " .. uspMessageResult["error_message"] .. " | Code: " .. uspMessageResult["error_code"])
else
    for key, _ in pairs(uspMessageResult) do
      print(key)
    end
end
```
{% endtab %}
{% endtabs %}

### send\_cwmp\_message()

#### Params:

1.  Serial Number \[string]

    The CPE unique identifier.
2.  XML payload \[string]

    Request body to be sent to the CPE.
3.  Message Type \[integer]

    CWMP message type. \
    0 = getParameterValues\
    1 = setParameterValues\
    2 = addObject\
    3 = deleteObject\
    4 = getParameterNames

Return:

The data returned depends on the TR-069 message type:

* If the CWMP message generates an error:

```json
{
    "error_message": "error to set param test.software",
    "error_code: "500"
}
```

* If it is a read or add operation the function will return a _table_.&#x20;
* If it is a delete or set operation the data returned will be a _boolean_ indicating if the result was a success or failure.

Example:

{% tabs %}
{% tab title="GetParameterValues" %}
<pre class="language-lua"><code class="lang-lua">local getParameterValuesType = 0

local xmlContentGet = [[
&#x3C;?xml version="1.0" encoding="UTF-8"?>
&#x3C;soap:Envelope xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:cwmp="urn:dslforum-org:cwmp-1-0" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:schemaLocation="urn:dslforum-org:cwmp-1-0 ..\schemas\wt121.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  &#x3C;soap:Header/>
  &#x3C;soap:Body soap:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    &#x3C;cwmp:GetParameterValues>
      &#x3C;ParameterNames>
        &#x3C;string>InternetGatewayDevice.LANDevice.1.WLANConfiguration.1.&#x3C;/string>
       &#x3C;/ParameterNames>
    &#x3C;/cwmp:GetParameterValues>
  &#x3C;/soap:Body>
&#x3C;/soap:Envelope>
]]
local serial_number = "HUAWNFYC-35454645"

<strong>local getTest = send_cwmp_message(serial_number, xmlContentGet, getParameterValuesType)
</strong>if getTest["error_message"] ~= nil then
   print("SN: " .. serial_number .. " | Error Message: " .. getTest["error_message"] .. " | Code: " .. getTest["error_code"])
else
  for key, value in pairs(getTest) do
    print(key .. ": " .. value)
  end
end
</code></pre>
{% endtab %}

{% tab title="AddObject" %}
<pre class="language-lua"><code class="lang-lua">local addObjectType = 2

<strong>local xmlContentAdd = [[
</strong>&#x3C;?xml version="1.0" encoding="UTF-8"?>
&#x3C;soap:Envelope xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:cwmp="urn:dslforum-org:cwmp-1-0" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:schemaLocation="urn:dslforum-org:cwmp-1-0 ..\schemas\wt121.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  &#x3C;soap:Header/>
  &#x3C;soap:Body soap:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    &#x3C;cwmp:AddObject>
      &#x3C;ObjectName>InternetGatewayDevice.LANDevice.1.WLANConfiguration.&#x3C;/ObjectName>
      &#x3C;ParameterKey>&#x3C;/ParameterKey>
    &#x3C;/cwmp:AddObject>
  &#x3C;/soap:Body>
&#x3C;/soap:Envelope>
]]
local serial_number = "HUAWNFYC-OPA123-0"

local addTest = send_cwmp_message(serial_number, xmlContentAdd, addObjectType)
if addTest["error_message"] ~= nil then
    print("SN: " .. serial_number .. " | Error Message: " .. addTest["error_message"] .. " | Code: " .. addTest["error_code"])
 else
    print("instance_number: " .. addTest.instance_number)
    print("status: " .. addTest.status)
end
</code></pre>
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
local serial_number = "HUAWNFYC-35454645"

local setTest = send_cwmp_message(serial_number, xmlContentSet, setParameterValuesType)
if type(setTest) == "table" then
   print("SN: " .. serial_number .. " | Error Message: " .. setTest["error_message"] .. " | Code: " .. setTest["error_code"])
else
  print("SetParameterValues worked")
end
```
{% endtab %}

{% tab title="DeleteObject" %}
```lua
local deleteObjectType = 3

local xmlContentDelete = [[
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:cwmp="urn:dslforum-org:cwmp-1-0" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:schemaLocation="urn:dslforum-org:cwmp-1-0 ..\schemas\wt121.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <soap:Header/>
  <soap:Body soap:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <cwmp:DeleteObject>
      <ObjectName>InternetGatewayDevice.LANDevice.1.WLANConfiguration.8.</ObjectName>
      <ParameterKey></ParameterKey>
    </cwmp:DeleteObject>
  </soap:Body>
</soap:Envelope>
]]
local serial_number = "HUAWNFYC-OPA123-0"

local deleteTest = send_cwmp_message(serial_number, xmlContentDelete, deleteObjectType)
if type(deleteTest) == "table" then
    print("SN: " .. serial_number .. " | Error Message: " .. deleteTest["error_message"] .. " | Code: " .. deleteTest["error_code"])
 else
   print("DeleteObject worked")
 end
```
{% endtab %}

{% tab title="GetParameterNames" %}
```lua
local getParameterNamesType = 4

local xmlContentGetParameterNames = [[<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:cwmp="urn:dslforum-org:cwmp-1-0" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:schemaLocation="urn:dslforum-org:cwmp-1-0 ..schemaswt121.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <soap:Header/>
  <soap:Body soap:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <cwmp:GetParameterNames>
      <ParameterPath>InternetGatewayDevice.LANDevice.1.WLANConfiguration.</ParameterPath>
      <NextLevel>1</NextLevel>
    </cwmp:GetParameterNames>
  </soap:Body>
</soap:Envelope>
]]
local serial_number = "HUAWNFYC-OPA123-0"

local getParameternames = send_cwmp_message(serial_number, xmlContentGetParameterNames, getParameterNamesType)
if getParameternames["error_message"] ~= nil then
    print("SN: " .. serial_number .. " | Error Message: " .. getParameternames["error_message"] .. " | Code: " .. getParameternames["error_code"])
 else
  for _, value in pairs(getParameternames) do
    print("Parameter Name" .. value)
  end
end
```
{% endtab %}
{% endtabs %}

### send\_cwmp\_async\_message()

Works the same way as the[ send\_cwmp\_message()](scripts.md#send_cwmp_message) function, but does not use the CWMP Connection Request mechanism, which means this RPC will only reach the CPE in the next Inform event of it. This allows to reach CPEs behind NAT, although it takes longer for the RPC to reach and be processed by the device.

### listen\_to\_cwmp\_event()

Listen to TR-069 events as defined in the standard.

#### Params:

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

    If no value is set or the value is "\*", than it listens to all events.
2. Serial Number \[string] (optional)\
   CPE unique identifier. If it's not set or the value is "\*", then it listens to all CPEs event(s).
3. Callback function (required)
4. Number of events to process in parallel \[integer] (optional)
5. Discard if processing \[boolean] (optional)\
   When `true`, an incoming event for a device that already has an event being processed (i.e. still running the callback) is discarded instead of waiting for a free worker slot. Useful to avoid piling up stale events for a slow or unresponsive device. Defaults to `false`, which queues the event until a worker slot frees up.

Return:

```json
{
    "serial_number": "HUAWNFYC-35454645",
    "event": "0"
}
```

Example:

```lua
function listen_to_cwmp_event_callback(cwmp_event)
    for key, value in pairs(cwmp_event) do
        print(key .. ": " .. tostring(value))
    end
end
listen_to_cwmp_event("*", "*", listen_to_cwmp_event_callback)
```

To process up to 10 events concurrently, and discard new events arriving for a device that's still being processed:

```lua
function listen_to_cwmp_event_callback(cwmp_event)
    for key, value in pairs(cwmp_event) do
        print(key .. ": " .. tostring(value))
    end
end
listen_to_cwmp_event("*", "*", listen_to_cwmp_event_callback, 10, true)
```

### get\_device()

Get device attributes from its unique identifier.

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
    "status": 2,
    "cwmp": true,
    "usp": false,
    "data_model": "TR-098"
}
```

`data_model` reflects the CWMP root data model reported by the CPE, either `"TR-098"` (`InternetGatewayDevice.`) or `"TR-181"` (`Device.`); it is empty for USP-only devices.

Example:

```lua
SERIAL_NUMBER = "HUAWNFYC-35454645"
local deviceAttributes = get_device(SERIAL_NUMBER)
if type(deviceAttributes) == "boolean" then
  print("device " .. SERIAL_NUMBER .. " not found")
else
  for key, value in pairs(deviceAttributes) do
    print(key .. ": " .. tostring(value))
  end
end
```

### listen\_to\_new\_device()

Receive all new device attributes that connect to Oktopus, independent of the protocol.

#### Params:

1. Callback function with new device data

```json
{
    "sn": "HUAWNFYC-35454645",
    "model": "WS7001-40",
    "vendor": "Huawei Technologies Co., Ltd.",
    "version": "",
    "product_class": "Huawei",
    "alias": "",
    "status": 2,
    "cwmp": true,
    "usp": false
}
```

Example:

```lua
function listen_to_new_device_callback(new_device)
  for key, value in pairs(new_device) do
    print(key .. ": " .. tostring(value))
  end
end
listen_to_new_device("*", "*", listen_to_new_device_callback)
```

### sleep()

Blocks the code execution for certain time.

#### Params:

1. Time duration \[integer]\
   Seconds of blocking the code execution.

Example:

```lua
while true do
  print("I'm going to appear again after 20 seconds")
  sleep(20)
end
```

### create\_or\_update\_device\_credential()

#### Params:

1. Username \[string]
2. Password \[string]

Return:

_Bool_ value indicating _true_ if the operation was successfull or _false_ if there was an error.

Example:

```lua
local username = "test"
local password = "test"

if create_or_update_device_credential(username, password) then
  print("success")
else
  print("error")
end
```

### generate\_random\_string()

#### Params:

1. Length \[int]\
   Size of the string to be generated

Example:

```lua
local username = "deviceX"
local password = generate_random_string(10)

if create_or_update_device_credential(username, password) then
  print("success")
else
  print("error")
end
```

### delete\_device()

Remove device from database.

#### Params:

1.  Serial Number \[string]

    The unique identifier of the CPE to be removed.

Return:

&#x20;_Bool_ value indicating _true_ if the operation was successfull and the device was removed from database or _false_ if there was an error.

Example:

```lua
local sn = "mqtt_client_92"
if delete_device(sn) then
  print("device " .. sn .. " successfully deleted")
else
  print("error to delete device ".. sn)
end
```

### set\_device\_model()

Update the model attribute of a device in the database.

#### Params:

1.  Serial Number \[string]

    The CPE unique identifier.
2.  Model \[string]

    The new model value to set.

Return:

_Bool_ value indicating _true_ if the operation was successfull or _false_ if there was an error.

Example:

```lua
local sn = "HUAWNFYC-35454645"
local model = "WS7001-40"

if set_device_model(sn, model) then
  print("device " .. sn .. " model updated to " .. model)
else
  print("error to update device " .. sn .. " model")
end
```

### get\_all\_devices()

Retrieve all devices saved on the database.

Return:

List of device objects.

<pre class="language-json"><code class="lang-json"><strong>[
</strong><strong>    {
</strong>        "sn": "HUAWNFYC-35454645",
        "model": "WS7001-40",
        "vendor": "Huawei Technologies Co., Ltd.",
        "version": "",
        "product_class": "Huawei",
        "alias": "",
        "status": 2,
        "cwmp": true,
        "usp": false
    },
    {
        "sn": "HUAWNFYC-test",
        "model": "WS7001-40",
        "vendor": "Huawei Technologies Co., Ltd.",
        "version": "",
        "product_class": "Huawei",
        "alias": "",
        "status": 2,
        "cwmp": true,
        "usp": false
    },
    ...
]
</code></pre>

Example:

```lua
local devices = get_all_devices()

for i = 1, #devices do
    local device = devices[i]
    print("  SN: " .. device.sn)
end
```

### refresh\_device\_parameters()

Sync device saved parameters, as PPPoE user, IP address and WAN MAC.

Params:

1. Serial number / USP agent endpoint id \[string]\
   The unique identifier of the CPE to be removed.

Return: _Bool_ value indicating f the operation was successfully executed ot not.

Example:

```lua
if refresh_device_parameters("sn-test-1") == false then
    print("Failed to refresh device parameters")
else
    print("Device parameters refreshed successfully")
end
```

### http\_request()

Make an HTTP call to a third-party REST API to gather or send data. Useful for integrating scripts with external systems (billing, ticketing, monitoring, etc).

Only `http`/`https` URLs are accepted; URLs with embedded credentials (e.g. `http://user:pass@host/`) are rejected; this function can only reach external/public hosts.

#### Params:

1.  Method \[string]

    HTTP method, e.g. `"GET"`, `"POST"`, `"PUT"`, `"PATCH"`, `"DELETE"`. Defaults to `"GET"` if empty.
2.  URL \[string]

    Full request URL. Must start with `http://` or `https://`.
3.  Headers \[table] (optional)

    Table of header name/value pairs to send with the request.
4.  Body \[string] (optional)

    Request body. Commonly used with `POST`/`PUT`/`PATCH`. Use [json\_encode()](scripts.md#json_encode) to build a JSON body.

Return:

A _table_ is always returned:

```json
{
    "ok": true,
    "status_code": 200,
    "body": "{\"plan\": \"gold\"}",
    "headers": {
        "Content-Type": "application/json"
    },
    "error_message": ""
}
```

* `ok` is `true` when the request completed with a `2xx` status code.
* `error_message` is populated when the request could not be completed at all (invalid URL, blocked target, timeout, connection error) — it does not mean the remote API returned an HTTP error.
* Use `body` together with [json\_decode()](scripts.md#json_decode) to parse JSON responses.

Example:

{% tabs %}
{% tab title="GET" %}
```lua
local response = http_request("GET", "https://api.example.com/customers/12345", {
    ["Authorization"] = "Bearer some-api-token",
    ["Accept"] = "application/json",
}, "")

if response.ok == false then
    print("HTTP request failed | status: " .. tostring(response.status_code) .. " | error: " .. tostring(response.error_message))
    return
end

local data, err = json_decode(response.body)
if err ~= nil then
    print("Failed to decode JSON response: " .. err)
    return
end

print("Customer plan: " .. tostring(data["plan"]))
```
{% endtab %}

{% tab title="POST" %}
```lua
local payload, encErr = json_encode({ status = "active", updated_by = "lua-script" })
if encErr ~= nil then
    print("Failed to encode JSON payload: " .. encErr)
    return
end

local response = http_request("POST", "https://api.example.com/customers/12345", {
    ["Authorization"] = "Bearer some-api-token",
    ["Content-Type"] = "application/json",
}, payload)

if response.ok then
    print("Customer updated successfully")
else
    print("Failed to update customer | status: " .. tostring(response.status_code))
end
```
{% endtab %}
{% endtabs %}

### json\_decode()

Parse a JSON string into a Lua table (or scalar value). Commonly used to read the `body` returned by [http\_request()](scripts.md#http_request).

#### Params:

1. JSON string \[string]

Return:

Two values are returned: the decoded value (table, string, number, boolean or `nil`), and an error message \[string] that is `nil` when decoding succeeds.

Example:

```lua
local data, err = json_decode('{"sn": "HUAWNFYC-35454645", "online": true}')
if err ~= nil then
    print("Failed to decode JSON: " .. err)
else
    print("sn: " .. data.sn)
end
```

### json\_encode()

Convert a Lua table (or scalar value) into a JSON string. Commonly used to build the `body` sent through [http\_request()](scripts.md#http_request).

#### Params:

1. Value \[table, string, number or boolean]

Return:

Two values are returned: the encoded JSON \[string], and an error message \[string] that is `nil` when encoding succeeds.

Example:

```lua
local json, err = json_encode({ sn = "HUAWNFYC-35454645", online = true })
if err ~= nil then
    print("Failed to encode JSON: " .. err)
else
    print(json)
end
```

### get\_service\_instances()

List the [service instances](services.md#service-instances) attached to a device.

#### Params:

1.  Serial Number \[string]

    The CPE unique identifier.

Return:

A table with `ok` \[boolean], `error_message` \[string], `error_code` \[number] and, on success, `instances` \[table] — an array of tables, each with `id`, `device_id`, `service_id`, `status` and `variables`:

```json
{
    "ok": true,
    "error_message": "",
    "error_code": 200,
    "instances": [
        {
            "id": "665f1b2e8c2a4a1b2c3d4e5f",
            "device_id": "HUAWNFYC-35454645",
            "service_id": "665f1a1e8c2a4a1b2c3d4e5e",
            "status": "provisioned",
            "variables": { "pppoe_username": "customer123" }
        }
    ]
}
```

Example:

```lua
local result = get_service_instances("HUAWNFYC-35454645")
if not result.ok then
    print("Failed to list service instances: " .. result.error_message)
else
    for _, instance in ipairs(result.instances) do
        print(instance.id .. " -> " .. instance.status)
    end
end
```

### create\_service\_instance()

Attach a service definition to a device, creating a new service instance.

#### Params:

1. Serial Number \[string] — the CPE unique identifier.
2. Service Definition ID \[string].
3. Variables \[table, optional] — the service-specific variable values for this device.

Return:

A table with `ok` \[boolean], `error_message` \[string] and `error_code` \[number]. The created instance's id is not returned — call [get\_service\_instances()](scripts.md#get_service_instances) to look it up.

Example:

```lua
local result = create_service_instance("HUAWNFYC-35454645", "665f1a1e8c2a4a1b2c3d4e5e", {
    pppoe_username = "customer123",
    pppoe_password = "s3cr3t",
})
if not result.ok then
    print("Failed to create service instance: " .. result.error_message)
end
```

### edit\_service\_instance()

Replace an existing service instance's device and variables. The service definition id cannot be changed.

#### Params:

1. Service Instance ID \[string].
2. Serial Number \[string] — the CPE unique identifier.
3. Service Definition ID \[string] — must match the instance's current service definition.
4. Variables \[table, optional] — the new service-specific variable values.

Return:

A table with `ok` \[boolean], `error_message` \[string] and `error_code` \[number].

Example:

```lua
local result = edit_service_instance(
    "665f1b2e8c2a4a1b2c3d4e5f",
    "HUAWNFYC-35454645",
    "665f1a1e8c2a4a1b2c3d4e5e",
    { pppoe_username = "customer123", pppoe_password = "new-password" }
)
if not result.ok then
    print("Failed to edit service instance: " .. result.error_message)
end
```

### delete\_service\_instance()

Delete a service instance.

#### Params:

1. Service Instance ID \[string].

Return:

A table with `ok` \[boolean], `error_message` \[string] and `error_code` \[number].

Example:

```lua
local result = delete_service_instance("665f1b2e8c2a4a1b2c3d4e5f")
if not result.ok then
    print("Failed to delete service instance: " .. result.error_message)
end
```

### apply\_service\_instance()

Run a service instance's provisioning script against its device right away, instead of waiting for its trigger event (see [Manual Execution](services.md#manual-execution)).

#### Params:

1. Service Instance ID \[string].

Return:

A table with `ok` \[boolean], `error_message` \[string], `error_code` \[number] and, on success, `status` \[string] (`"provisioned"` or `"failed"`). `ok` reflects whether the request itself succeeded; `status` is the actual provisioning outcome — a script can legitimately fail provisioning even though the request succeeded (`ok = true`, `status = "failed"`).

Example:

```lua
local result = apply_service_instance("665f1b2e8c2a4a1b2c3d4e5f")
if not result.ok then
    print("Failed to apply service instance: " .. result.error_message)
elseif result.status ~= "provisioned" then
    print("Service instance provisioning failed")
else
    print("Service instance provisioned")
end
```

### apply\_service\_instance\_and\_forget()

Trigger a service instance's provisioning script the same way [apply\_service\_instance()](scripts.md#apply_service_instance) does, but don't wait for it to run to completion — the request is enqueued and the function returns immediately. Because it doesn't wait for a response, it can't report the final `status`, and `ok` only reflects whether the message was enqueued locally, not whether the zero-touch-provisioning service actually received or ran it.

Use this when applying a service instance whose script may run long (e.g. it calls [sleep()](scripts.md#sleep) or does several sequential CPE round trips) and the caller doesn't need to wait for the outcome.

#### Params:

1. Service Instance ID \[string].

Return:

A table with `ok` \[boolean], `error_message` \[string] and `error_code` \[number]. There is no `status` field.

Example:

```lua
local result = apply_service_instance_and_forget("665f1b2e8c2a4a1b2c3d4e5f")
if not result.ok then
    print("Failed to enqueue service instance apply: " .. result.error_message)
end
```

## Examples

The scripts below are adapted from real-world scenario automations built by Oktopus.

### Migrate CWMP (TR-069) CPE to USP (TR-369)

Some CPE vendors ship devices with a proprietary, disabled-by-default USP agent running alongside the TR-069 agent (e.g. Zyxel's `X_ZYXEL_Enable` parameter). This is a common pattern to migrate such a fleet from CWMP to USP without a firmware upgrade: listen for CWMP events, bootstrap the on-board USP agent entirely through CWMP `SetParameterValues`/`AddObject` calls, confirm the device shows up connected over USP, then disable CWMP so the device is fully USP-managed going forward.

```lua
-- ---- Fill in with your own environment settings ----
local PASSWORD_LENGTH = 30
local MAX_ATTEMPTS_TO_GET_DEVICE = 3
local TIME_TO_WAIT_FOR_USP_DEVICE_CONNECTION = 10
local CONTROLLER_ENDPOINT_ID = ""     -- e.g. "proto::oktopus"
local CONTROLLER_ALIAS = "oktopus"
local MQTT_BROKER_ADDRESS = ""        -- your MQTT broker hostname
local MQTT_BROKER_PORT = 1883
local MQTT_BROKER_TLS_PORT = 8883
local TRANSPORT_PROTOCOL = "TCP/IP"
local TRANSPORT_PROTOCOL_ENCRYPTED = "TLS"
local CA_CERTIFICATE_URL = ""         -- CA cert the CPE should trust to reach your broker over TLS
-- ------------------------------------------------------

local function build_get_usp_endpoint_id_xml()
    return [[<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:cwmp="urn:dslforum-org:cwmp-1-0" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <soap:Header><cwmp:ID soap:mustUnderstand="1">]] .. generate_random_string(8) .. [[</cwmp:ID></soap:Header>
  <soap:Body>
    <cwmp:GetParameterValues>
      <ParameterNames soapenc:arrayType="xsd:string[1]">
        <string>Device.LocalAgent.EndpointID</string>
      </ParameterNames>
    </cwmp:GetParameterValues>
  </soap:Body>
</soap:Envelope>]]
end

local function build_add_object_xml(object_name)
    return [[<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:cwmp="urn:dslforum-org:cwmp-1-0" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <soap:Header><cwmp:ID soap:mustUnderstand="1">]] .. generate_random_string(8) .. [[</cwmp:ID></soap:Header>
  <soap:Body>
    <cwmp:AddObject>
      <ObjectName>]] .. object_name .. [[</ObjectName>
      <ParameterKey></ParameterKey>
    </cwmp:AddObject>
  </soap:Body>
</soap:Envelope>]]
end

local function build_set_zyxel_usp_enable_xml(enabled)
    return [[<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:cwmp="urn:dslforum-org:cwmp-1-0" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <soap:Header><cwmp:ID soap:mustUnderstand="1">]] .. generate_random_string(8) .. [[</cwmp:ID></soap:Header>
  <soap:Body>
    <cwmp:SetParameterValues>
      <ParameterList soapenc:arrayType="cwmp:ParameterValueStruct[1]">
        <ParameterValueStruct>
          <Name>Device.LocalAgent.X_ZYXEL_Enable</Name>
          <Value xsi:type="xsd:boolean">]] .. (enabled and "1" or "0") .. [[</Value>
        </ParameterValueStruct>
      </ParameterList>
      <ParameterKey></ParameterKey>
    </cwmp:SetParameterValues>
  </soap:Body>
</soap:Envelope>]]
end

-- Builds a SetParameterValues envelope from a list of { name, value, type } entries
local function build_set_parameter_values_xml(params)
    local structs = {}
    for _, p in ipairs(params) do
        table.insert(structs, [[
        <ParameterValueStruct>
          <Name>]] .. p.name .. [[</Name>
          <Value xsi:type="]] .. p.type .. [[">]] .. p.value .. [[</Value>
        </ParameterValueStruct>]])
    end
    return [[<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:cwmp="urn:dslforum-org:cwmp-1-0" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <soap:Header><cwmp:ID soap:mustUnderstand="1">]] .. generate_random_string(8) .. [[</cwmp:ID></soap:Header>
  <soap:Body>
    <cwmp:SetParameterValues>
      <ParameterList soapenc:arrayType="cwmp:ParameterValueStruct[]] .. #params .. [[]">]] .. table.concat(structs) .. [[
      </ParameterList>
      <ParameterKey></ParameterKey>
    </cwmp:SetParameterValues>
  </soap:Body>
</soap:Envelope>]]
end

-- Polls get_device() until the CPE shows up connected over USP, or gives up
local function is_device_connected_to_usp(usp_agent_endpoint_id)
    for attempt = 1, MAX_ATTEMPTS_TO_GET_DEVICE do
        sleep(TIME_TO_WAIT_FOR_USP_DEVICE_CONNECTION * attempt)
        local device = get_device(usp_agent_endpoint_id)
        if type(device) == "table" then
            return device.usp == true
        end
    end
    return false
end

local function migrate_cpe_to_usp(serial_number)
    local get_id_result = send_cwmp_message(serial_number, build_get_usp_endpoint_id_xml(), 0, true)
    if get_id_result["error_message"] ~= nil then
        print("SN: " .. serial_number .. " | failed to read USP endpoint id | " .. get_id_result["error_message"])
        return
    end

    local usp_agent_endpoint_id = get_id_result["Device.LocalAgent.EndpointID"]
    if usp_agent_endpoint_id == nil or usp_agent_endpoint_id == "" then
        print("SN: " .. serial_number .. " | USP endpoint id is empty, device may not support USP")
        return
    end

    local existing_device = get_device(usp_agent_endpoint_id)
    if type(existing_device) == "table" and existing_device.usp then
        print("SN: " .. serial_number .. " | already migrated, cleaning up old CWMP record")
        delete_device(serial_number)
        return
    end

    local password = generate_random_string(PASSWORD_LENGTH)
    local credential_result = create_or_update_device_credential(usp_agent_endpoint_id, password)
    if credential_result["ok"] == false then
        print("SN: " .. serial_number .. " | failed to create device credential | " .. credential_result["error_msg"])
        return
    end

    -- 1. Add Local Agent MTP
    local mtp_result = send_cwmp_message(serial_number, build_add_object_xml("Device.LocalAgent.MTP."), 2, true)
    if mtp_result["error_message"] ~= nil then
        print("SN: " .. serial_number .. " | failed to add MTP | " .. mtp_result["error_message"])
        return
    end
    local mtp_instance = mtp_result.instance_number

    -- 2. Add MQTT Client (plain MQTT for now, upgraded to TLS once USP is confirmed)
    local mqtt_result = send_cwmp_message(serial_number, build_add_object_xml("Device.MQTT.Client."), 2, true)
    if mqtt_result["error_message"] ~= nil or mqtt_result.instance_number == 0 then
        print("SN: " .. serial_number .. " | failed to add MQTT client | " .. mqtt_result["error_message"])
        return
    end
    local mqtt_instance = mqtt_result.instance_number

    -- 3. Add Local Agent Controller
    local controller_result = send_cwmp_message(serial_number, build_add_object_xml("Device.LocalAgent.Controller."), 2, true)
    if controller_result["error_message"] ~= nil then
        print("SN: " .. serial_number .. " | failed to add controller | " .. controller_result["error_message"])
        return
    end
    local controller_instance = controller_result.instance_number

    -- 4. Add Local Agent Controller MTP
    local controller_mtp_result = send_cwmp_message(
        serial_number,
        build_add_object_xml("Device.LocalAgent.Controller." .. controller_instance .. ".MTP."),
        2, true
    )
    if controller_mtp_result["error_message"] ~= nil or controller_mtp_result.instance_number == 0 then
        print("SN: " .. serial_number .. " | failed to add controller MTP | " .. controller_mtp_result["error_message"])
        return
    end
    local controller_mtp_instance = controller_mtp_result.instance_number

    -- 5. Bootstrap USP over plain MQTT and point it at this controller
    local bootstrap_result = send_cwmp_message(serial_number, build_set_parameter_values_xml({
        { name = "Device.LocalAgent.MTP." .. mtp_instance .. ".Enable", value = "1", type = "xsd:boolean" },
        { name = "Device.LocalAgent.MTP." .. mtp_instance .. ".Alias", value = CONTROLLER_ALIAS, type = "xsd:string" },
        { name = "Device.LocalAgent.MTP." .. mtp_instance .. ".Protocol", value = "MQTT", type = "xsd:string" },
        { name = "Device.LocalAgent.MTP." .. mtp_instance .. ".MQTT.Reference", value = "Device.MQTT.Client." .. mqtt_instance, type = "xsd:string" },
        { name = "Device.LocalAgent.MTP." .. mtp_instance .. ".MQTT.ResponseTopicConfigured", value = "oktopus/usp/v1/agent/" .. usp_agent_endpoint_id, type = "xsd:string" },
        { name = "Device.LocalAgent.Controller." .. controller_instance .. ".Enable", value = "1", type = "xsd:boolean" },
        { name = "Device.LocalAgent.Controller." .. controller_instance .. ".Alias", value = CONTROLLER_ALIAS, type = "xsd:string" },
        { name = "Device.LocalAgent.Controller." .. controller_instance .. ".EndpointID", value = CONTROLLER_ENDPOINT_ID, type = "xsd:string" },
        { name = "Device.LocalAgent.Controller." .. controller_instance .. ".AssignedRole", value = "Device.LocalAgent.ControllerTrust.Role.1", type = "xsd:string" },
        { name = "Device.LocalAgent.Controller." .. controller_instance .. ".MTP." .. controller_mtp_instance .. ".Enable", value = "1", type = "xsd:boolean" },
        { name = "Device.LocalAgent.Controller." .. controller_instance .. ".MTP." .. controller_mtp_instance .. ".Alias", value = CONTROLLER_ALIAS, type = "xsd:string" },
        { name = "Device.LocalAgent.Controller." .. controller_instance .. ".MTP." .. controller_mtp_instance .. ".Protocol", value = "MQTT", type = "xsd:string" },
        { name = "Device.LocalAgent.Controller." .. controller_instance .. ".MTP." .. controller_mtp_instance .. ".MQTT.Reference", value = "Device.MQTT.Client." .. mqtt_instance, type = "xsd:string" },
        { name = "Device.LocalAgent.Controller." .. controller_instance .. ".MTP." .. controller_mtp_instance .. ".MQTT.Topic", value = "oktopus/usp/v1/notify/" .. usp_agent_endpoint_id, type = "xsd:string" },
        { name = "Device.MQTT.Client." .. mqtt_instance .. ".Enable", value = "1", type = "xsd:boolean" },
        { name = "Device.MQTT.Client." .. mqtt_instance .. ".Alias", value = CONTROLLER_ALIAS, type = "xsd:string" },
        { name = "Device.MQTT.Client." .. mqtt_instance .. ".BrokerAddress", value = MQTT_BROKER_ADDRESS, type = "xsd:string" },
        { name = "Device.MQTT.Client." .. mqtt_instance .. ".BrokerPort", value = tostring(MQTT_BROKER_PORT), type = "xsd:unsignedInt" },
        { name = "Device.MQTT.Client." .. mqtt_instance .. ".Username", value = credential_result["username"], type = "xsd:string" },
        { name = "Device.MQTT.Client." .. mqtt_instance .. ".Password", value = credential_result["password"], type = "xsd:string" },
        { name = "Device.MQTT.Client." .. mqtt_instance .. ".TransportProtocol", value = TRANSPORT_PROTOCOL, type = "xsd:string" },
        { name = "Device.MQTT.Client." .. mqtt_instance .. ".ProtocolVersion", value = "5.0", type = "xsd:string" },
        { name = "Device.LocalAgent.X_ZYXEL_Enable", value = "1", type = "xsd:boolean" },
    }), 1, true)
    if type(bootstrap_result) == "table" or bootstrap_result == false then
        print("SN: " .. serial_number .. " | failed to set USP bootstrap parameters | " .. bootstrap_result["error_message"])
        return
    end

    -- 6. Wait for the CPE to show up connected over USP before going further
    if not is_device_connected_to_usp(usp_agent_endpoint_id) then
        print("SN: " .. serial_number .. " | device did not connect over USP, aborting")
        return
    end
    print("SN: " .. serial_number .. " | connected over USP (plain MQTT)")

    -- 7. Trust the broker's CA certificate before switching to TLS
    local cert_result = send_usp_message(usp_agent_endpoint_id, [[{
    "header": { "msg_id": "]] .. generate_random_string(8) .. [[", "msg_type": 6 },
    "body": {
        "request": {
            "operate": {
                "command": "Device.DeviceInfo.X_ZYXEL_SSLCAFile.Download()",
                "input_args": { "URL": "]] .. CA_CERTIFICATE_URL .. [[" },
                "send_resp": true
            }
        }
    }
}]])
    if cert_result["error_message"] ~= nil then
        print("SN: " .. serial_number .. " | failed to add CA certificate | " .. cert_result["error_message"])
        return
    end

    -- 8. Point the MQTT client at the TLS port and briefly flip X_ZYXEL_Enable to reload the config
    local tls_result = send_cwmp_message(serial_number, build_set_parameter_values_xml({
        { name = "Device.MQTT.Client." .. mqtt_instance .. ".BrokerPort", value = tostring(MQTT_BROKER_TLS_PORT), type = "xsd:unsignedInt" },
        { name = "Device.MQTT.Client." .. mqtt_instance .. ".TransportProtocol", value = TRANSPORT_PROTOCOL_ENCRYPTED, type = "xsd:string" },
        { name = "Device.LocalAgent.X_ZYXEL_Enable", value = "0", type = "xsd:boolean" },
    }), 1, true)
    if type(tls_result) == "table" or tls_result == false then
        print("SN: " .. serial_number .. " | failed to switch MQTT client to TLS | " .. tls_result["error_message"])
        return
    end

    local reenable_result = send_cwmp_message(serial_number, build_set_zyxel_usp_enable_xml(true), 1, true)
    if type(reenable_result) == "table" or reenable_result == false then
        print("SN: " .. serial_number .. " | failed to re-enable USP | " .. reenable_result["error_message"])
        return
    end
    print("SN: " .. serial_number .. " | migrated to USP over TLS")

    -- 9. Once USP over TLS is confirmed, disable CWMP and persist the config to survive a reboot
    sleep(5)
    local disable_cwmp_result = send_usp_message(usp_agent_endpoint_id, [[{
    "header": { "msg_id": "]] .. generate_random_string(8) .. [[", "msg_type": 4 },
    "body": {
        "request": {
            "set": {
                "allow_partial": true,
                "update_objs": [
                    { "obj_path": "Device.ManagementServer.", "param_settings": [
                        { "param": "EnableCWMP", "value": "false", "required": true }
                    ]}
                ]
            }
        }
    }
}]])
    if disable_cwmp_result["error_message"] ~= nil then
        print("SN: " .. serial_number .. " | failed to disable CWMP | " .. disable_cwmp_result["error_message"])
        return
    end

    send_usp_message(usp_agent_endpoint_id, [[{
    "header": { "msg_id": "]] .. generate_random_string(8) .. [[", "msg_type": 4 },
    "body": {
        "request": {
            "set": {
                "allow_partial": true,
                "update_objs": [
                    { "obj_path": "Device.ManagementServer.", "param_settings": [
                        { "param": "X_ZYXEL_ROMD_Action", "value": "1", "required": true }
                    ]}
                ]
            }
        }
    }
}]]) -- Zyxel-specific: persist the current config to ROM-D so it survives a reboot

    delete_device(serial_number)
    print("SN: " .. serial_number .. " | migration complete, old CWMP record removed")
end

-- Runs the migration for every CPE the next time it sends a CWMP event (e.g. Inform)
listen_to_cwmp_event("*", "*", function(cwmp_event)
    migrate_cpe_to_usp(cwmp_event.serial_number)
end, 10, true)
```

{% hint style="info" %}
This flow is vendor-specific in a few places — namely the `X_ZYXEL_Enable` and `X_ZYXEL_ROMD_Action` parameters, and the CA certificate download command. Other CPE vendors with an on-board USP agent typically expose an equivalent enable flag and a way to download/trust a CA certificate; adjust those parameter paths to match your CPE's data model.
{% endhint %}

### Provision USP settings across an existing fleet

Once devices are on USP, a common pattern is a script that walks every device with [get\_all\_devices()](scripts.md#get_all_devices) and applies a USP configuration if it isn't already present — useful for periodically rolling out a new setting to a whole fleet.

{% tabs %}
{% tab title="Bulk Data Collection" %}
Creates (idempotently) a `Device.BulkData.Profile.` that periodically publishes device data over MQTT, then enables it.

```lua
-- ---- Fill in with your own environment settings ----
local BULK_DATA_PROFILE_NAME = "USP MQTT Profile 1"
local MQTT_CLIENT_REFERENCE = "Device.MQTT.Client.1"
local REPORTING_INTERVAL_SECONDS = "300"
-- ------------------------------------------------------

local function build_get_bulk_data_profile_msg()
    return [[{
    "header": { "msg_id": "]] .. generate_random_string(8) .. [[", "msg_type": 1 },
    "body": { "request": { "get": {
        "paramPaths": ["Device.BulkData.Profile.[Name==]] .. BULK_DATA_PROFILE_NAME .. [[]."]
    }}}
}]]
end

local function build_add_bulk_data_profile_msg(usp_agent_endpoint_id)
    return [[{
    "header": { "msg_id": "]] .. generate_random_string(8) .. [[", "msg_type": 8 },
    "body": {
        "request": {
            "add": {
                "allow_partial": true,
                "create_objs": [
                    {
                        "obj_path": "Device.BulkData.Profile.",
                        "param_settings": [
                            { "param": "EncodingType", "value": "JSON", "required": true },
                            { "param": "Name", "value": "]] .. BULK_DATA_PROFILE_NAME .. [[", "required": true },
                            { "param": "ReportingInterval", "value": "]] .. REPORTING_INTERVAL_SECONDS .. [[", "required": true },
                            { "param": "Protocol", "value": "MQTT", "required": true },
                            { "param": "MQTT.PublishTopic", "value": "oktopus/usp/v1/bulkdata/]] .. usp_agent_endpoint_id .. [[", "required": true },
                            { "param": "MQTT.Reference", "value": "]] .. MQTT_CLIENT_REFERENCE .. [[", "required": true },
                            { "param": "MQTT.PublishQoS", "value": "0", "required": true }
                        ]
                    },
                    {
                        "obj_path": "Device.BulkData.Profile.[Name==]] .. BULK_DATA_PROFILE_NAME .. [[].Parameter.",
                        "param_settings": [
                            { "param": "Name", "value": "Hosts", "required": true },
                            { "param": "Reference", "value": "Device.Hosts.Host.*.", "required": true }
                        ]
                    }
                ]
            }
        }
    }
}]]
end

local uspEnableBulkDataCollection = [[{
    "header": { "msg_id": "]] .. generate_random_string(8) .. [[", "msg_type": 4 },
    "body": {
        "request": {
            "set": {
                "allow_partial": true,
                "update_objs": [
                    { "obj_path": "Device.BulkData.", "param_settings": [
                        { "param": "Enable", "value": "true", "required": true }
                    ]},
                    { "obj_path": "Device.BulkData.Profile.[Name==]] .. BULK_DATA_PROFILE_NAME .. [[].", "param_settings": [
                        { "param": "Enable", "value": "true", "required": true }
                    ]}
                ]
            }
        }
    }
}]]

local devices = get_all_devices()
for i = 1, #devices do
    local device = devices[i]
    if device.status == 2 and device.usp then -- online and connected over USP
        local usp_agent_endpoint_id = device.sn

        local profile = send_usp_message(usp_agent_endpoint_id, build_get_bulk_data_profile_msg())
        local already_provisioned = profile["error_message"] == nil and next(profile) ~= nil

        if already_provisioned then
            print("SN: " .. usp_agent_endpoint_id .. " | bulk data profile already present")
        else
            local add_result = send_usp_message(usp_agent_endpoint_id, build_add_bulk_data_profile_msg(usp_agent_endpoint_id))
            if add_result["error_message"] ~= nil then
                print("SN: " .. usp_agent_endpoint_id .. " | failed to add bulk data profile | " .. add_result["error_message"])
            else
                local enable_result = send_usp_message(usp_agent_endpoint_id, uspEnableBulkDataCollection)
                if enable_result["error_message"] ~= nil then
                    print("SN: " .. usp_agent_endpoint_id .. " | failed to enable bulk data collection | " .. enable_result["error_message"])
                else
                    print("SN: " .. usp_agent_endpoint_id .. " | bulk data collection enabled")
                end
            end
        end
    end
end
```
{% endtab %}

{% tab title="Notification Subscriptions" %}
Replaces any existing subscriptions with a standard set — one for operation-complete notifications on diagnostics, and one for transfer-complete events — and points them at this controller's MQTT notify topic.

```lua
local uspDeleteMsg = [[{
    "header": { "msg_id": "]] .. generate_random_string(8) .. [[", "msg_type": 10 },
    "body": { "request": { "delete": {
        "allow_partial": true,
        "obj_paths": ["Device.LocalAgent.Subscription.*."]
    }}}
}]]

local uspAddMsg = [[{
    "header": { "msg_id": "]] .. generate_random_string(8) .. [[", "msg_type": 8 },
    "body": {
        "request": {
            "add": {
                "allow_partial": true,
                "create_objs": [
                    {
                        "obj_path": "Device.LocalAgent.Subscription.",
                        "param_settings": [
                            { "param": "NotifType", "value": "OperationComplete", "required": true },
                            { "param": "Persistent", "value": "1", "required": true },
                            { "param": "ReferenceList", "value": "Device.IP.Diagnostics.DownloadDiagnostics(),Device.IP.Diagnostics.UploadDiagnostics()", "required": true },
                            { "param": "NotifRetry", "value": "0", "required": true },
                            { "param": "Enable", "value": "1", "required": true }
                        ]
                    },
                    {
                        "obj_path": "Device.LocalAgent.Subscription.",
                        "param_settings": [
                            { "param": "NotifType", "value": "Event", "required": true },
                            { "param": "Persistent", "value": "1", "required": true },
                            { "param": "ReferenceList", "value": "Device.LocalAgent.TransferComplete!", "required": true },
                            { "param": "NotifRetry", "value": "0", "required": true },
                            { "param": "Enable", "value": "1", "required": true }
                        ]
                    }
                ]
            }
        }
    }
}]]

local function build_set_mqtt_notify_topic_msg(usp_agent_endpoint_id)
    return [[{
    "header": { "msg_id": "]] .. generate_random_string(8) .. [[", "msg_type": 4 },
    "body": {
        "request": {
            "set": {
                "allow_partial": true,
                "update_objs": [
                    { "obj_path": "Device.LocalAgent.Controller.1.MTP.1.MQTT.", "param_settings": [
                        { "param": "Topic", "value": "oktopus/usp/v1/notify/]] .. usp_agent_endpoint_id .. [[", "required": true }
                    ]}
                ]
            }
        }
    }
}]]
end

local devices = get_all_devices()
for i = 1, #devices do
    local device = devices[i]
    if device.status == 2 and device.usp then -- online and connected over USP
        local usp_agent_endpoint_id = device.sn

        local topic_result = send_usp_message(usp_agent_endpoint_id, build_set_mqtt_notify_topic_msg(usp_agent_endpoint_id))
        if topic_result["error_message"] ~= nil then
            print("SN: " .. usp_agent_endpoint_id .. " | failed to set MQTT notify topic | " .. topic_result["error_message"])
            goto continue
        end

        local delete_result = send_usp_message(usp_agent_endpoint_id, uspDeleteMsg)
        if delete_result["error_message"] ~= nil then
            print("SN: " .. usp_agent_endpoint_id .. " | failed to delete old subscriptions | " .. delete_result["error_message"])
        end

        local add_result = send_usp_message(usp_agent_endpoint_id, uspAddMsg)
        if add_result["error_message"] ~= nil then
            print("SN: " .. usp_agent_endpoint_id .. " | failed to add subscriptions | " .. add_result["error_message"])
        else
            print("SN: " .. usp_agent_endpoint_id .. " | subscriptions provisioned")
        end

        ::continue::
    end
end
```
{% endtab %}
{% endtabs %}













<br>



