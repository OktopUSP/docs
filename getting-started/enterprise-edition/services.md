---
description: A set of parameters to define the CPE configuration.
---

# Services

The services are used in conjunction with TR-x69 discovery mechanisms to achieve Z**ero Touch Provisioning** flows. The CPE is added to the system beforehand, and the proper service is attached to it. When the field technician plugs the CPE into the network infrastructure, the CPE is provisioned with the parameters defined in the ACS specific to that customer.

## Service Definition

A service definition is a set of parameters, variables, and operations that dictate the communication structure with the CPE when applying the service to its configuration.

## Scripts

Scripts in the zero-touch-provisioning system are written in **Lua** and provide the provisioning logic for CPE (Customer Premises Equipment) devices. They define how services are applied to devices, including configuration changes, credential management, and parameter updates.

### Execution Flow

1. **Script Initialization**: A Lua script is loaded with the device context (device ID, tenant ID, and variables)
2. **Lua Environment Setup**: The script runs in a secure Lua sandbox with access to predefined global variables and functions
3. **Function Calls**: The script can call Go functions to interact with devices and services
4. **Status Return**: The script must set a `provisioning_status` global variable to indicate success or failure

#### Global Variables

These variables are automatically available to scripts:

| Variable              | Type   | Description                                                              |
| --------------------- | ------ | ------------------------------------------------------------------------ |
| `device_id`           | string | The serial number or unique identifier of the target device              |
| `variables`           | table  | A table containing service-specific variables passed during provisioning |
| `provisioning_status` | string | Must be set to indicate the final status (see below)                     |

**Provisioning Status Values**

| Value           | Meaning                      |
| --------------- | ---------------------------- |
| `"provisioned"` | Service successfully applied |
| `"failed"`      | Service provisioning failed  |

**Important**: Scripts must explicitly set `provisioning_status` before completion. If not set, the provisioning will fail.

#### Example Structure

```lua
-- Access device information
local device_sn = get_device_id()
print("Processing device: " .. device_sn)

-- Access variables
local username = get_variable_value("pppoe_username")
local password = get_variable_value("pppoe_password")

-- Perform operations
local result = send_cwmp_message_and_forget(device_sn, message_xml, timeout)

-- Check result and set status
if result == false or type(result) == "table" then
    set_provisioning_status("failed")
else
    set_provisioning_status("provisioned")
end
```

### Available Functions

#### Provisioning Functions

**`set_provisioning_status(status)`**

Sets the final provisioning status for the script execution.

**Parameters:**

* `status` (string): Either `"provisioned"` or `"failed"`

**Returns:** Nothing

**Example:**

```lua
if success then
    set_provisioning_status("provisioned")
else
    set_provisioning_status("failed")
end
```

***

**`get_device_id()`**

Retrieves the device serial number or identifier currently being provisioned.

**Parameters:** None

**Returns:**

* (string) The device ID

**Example:**

```lua
local sn = get_device_id()
print("Device SN: " .. sn)
```

***

**`get_variable_value(variable_name)`**

Retrieves a variable value from the service instance configuration.

**Parameters:**

* `variable_name` (string): The name of the variable to retrieve

**Returns:**

* (string) The variable value, or nil if not found

**Example:**

```lua
local pppoe_username = get_variable_value("pppoe_username")
local pppoe_password = get_variable_value("pppoe_password")
```

***

#### CPE Interaction Functions

**`send_cwmp_message_and_forget(device_id, message_body, timeout)`**

Enqueue a CWMP message to the device and does not wait for a response.

**Parameters:**

* `device_id` (string): The serial number of the target device
* `message_body` (string): XML-formatted CWMP message
* `timeout` (number): Timeout in seconds (for the operation)

**Returns:**

* (boolean) `true` if the message was successfully sent
* (table) Error table with `error_message` and `error_code` fields if failed

**Example:**

```lua
local cwmp_msg = [[
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
    <cwmp:SetParameterValues>
      <!-- parameters -->
    </cwmp:SetParameterValues>
  </soap:Body>
</soap:Envelope>
]]

local result = send_cwmp_message_and_forget(get_device_id(), cwmp_msg, 2)

if result == false or type(result) == "table" then
    print("Error: " .. result["error_message"] .. " (" .. result["error_code"] .. ")")
    set_provisioning_status("failed")
else
    print("Message sent successfully")
end
```

***

#### Utility Functions

**`sleep(seconds)`**

Pauses script execution for the specified number of seconds.

**Parameters:**

* `seconds` (number): Number of seconds to sleep

**Returns:** Nothing

**Example:**

```lua
print("Waiting for device to process...")
sleep(5)
print("Continuing...")
```

***

**`generate_random_string(length)`**

Generates a random alphanumeric string of the specified length.

**Parameters:**

* `length` (number): Length of the random string to generate (must be > 0)

**Returns:**

* (string) Random string of the specified length, or empty string if length is invalid

**Example:**

```lua
local random_password = generate_random_string(12)
print("Generated password: " .. random_password)
```

***

**`create_or_update_device_credential(username, password)`**

Creates or updates device credentials for authentication.

**Parameters:**

* `username` (string): The username (cannot contain dots, cannot be empty)
* `password` (string): The password (cannot contain dots, cannot be empty)

**Returns:**

* (table) Result table with the following fields:
  * `ok` (boolean): Success flag
  * `username` (string): The full username (prefixed with organization ID)
  * `password` (string): The password
  * `error_msg` (string): Error message if failed, empty if successful

**Validation:**

* Username and password cannot be empty
* Username and password cannot contain dots
* Requires valid organization context

**Example:**

```lua
local result = create_or_update_device_credential("admin_user", "secure_pass_123")

if result["ok"] then
    print("Credential created: " .. result["username"])
    -- Now you can use result["username"] and result["password"] for device login
else
    print("Failed to create credential: " .. result["error_msg"])
    set_provisioning_status("failed")
end
```

***

**`refresh_device_parameters(device_id)`**

Triggers a parameter refresh on the device, forcing it to send an updated parameter list to the ACS.

**Parameters:**

* `device_id` (string): The serial number of the device

**Returns:**

* (boolean) `true` if refresh was successfully triggered, `false` otherwise

**Example:**

```lua
if refresh_device_parameters(get_device_id()) then
    print("Device parameters refresh initiated")
else
    print("Failed to refresh device parameters")
    set_provisioning_status("failed")
end
```

#### Best Practices

1.  **Always Check Return Values**: Never assume a function succeeded

    ```lua
    local result = send_cwmp_message_and_forget(device_id, msg, 2)
    if result == false or type(result) == "table" then
        -- Handle error
    end
    ```
2.  **Use Descriptive Logging**: Leverage the `print()` function

    ```lua
    print("Step 1: Creating PPPoE WAN connection")
    local result = send_cwmp_message_and_forget(device_id, msg, 2)
    if result == false then
        print("Failed to create PPPoE WAN")
        set_provisioning_status("failed")
        return
    end
    ```
3.  **Set Status Explicitly**: Always set provisioning status before script ends

    ```lua
    -- Good
    if success then
        set_provisioning_status("provisioned")
    else
        set_provisioning_status("failed")
    end

    -- Avoid - status not set
    if success then
        print("Success")
    end
    ```
4.  **Handle Sequential Operations**: Use proper error handling for multi-step processes

    ```lua
    -- Step 1
    local step1 = send_cwmp_message_and_forget(device_id, msg1, 2)
    if step1 == false then
        set_provisioning_status("failed")
        return
    end

    -- Step 2
    sleep(1)
    local step2 = send_cwmp_message_and_forget(device_id, msg2, 2)
    if step2 == false then
        set_provisioning_status("failed")
        return
    end

    set_provisioning_status("provisioned")
    ```

### Limitations

* Scripts run in a restricted Lua sandbox for security
* File I/O is not available
* Network operations are only available through the provided functions
* Scripts must complete within reasonable timeout limits

### Real-World Example: PPPoE Service Configuration

1. Creates a PPPoE WAN connection on the device
2. Sets PPPoE credentials using variables
3. Changes the WAN type to PPPoE
4. Refreshes device parameters
5. Properly handles errors at each step

The example demonstrates:

* Using variables from the service instance
* Building XML CWMP messages
* Sequential provisioning steps
* Error handling
* Device parameter refresh

```lua
local function createCWMPAddPPPoEWANMsg(wanInstanceNumber)
    return [[
        <?xml version="1.0" encoding="UTF-8"?>
        <soap:Envelope xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:cwmp="urn:dslforum-org:cwmp-1-0" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:schemaLocation="urn:dslforum-org:cwmp-1-0 ..schemaswt121.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
        <soap:Header/>
        <soap:Body soap:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
            <cwmp:AddObject>
            <ObjectName>InternetGatewayDevice.WANDevice.1.WANConnectionDevice.]] .. wanInstanceNumber ..[[.WANPPPConnection.</ObjectName>
            <ParameterKey></ParameterKey>
            </cwmp:AddObject>
        </soap:Body>
        </soap:Envelope>
    ]]
end

local function createCWMPSetPPPoECredentialsMsg(wanInstanceNumber, pppoeInstanceNumber)
    return [[
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:cwmp="urn:dslforum-org:cwmp-1-0" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:schemaLocation="urn:dslforum-org:cwmp-1-0 ..schemaswt121.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <soap:Header/>
  <soap:Body soap:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <cwmp:SetParameterValues>
      <ParameterList soapenc:arrayType="cwmp:ParameterValueStruct[4]">
        <ParameterValueStruct>
            <Name>InternetGatewayDevice.WANDevice.1.WANConnectionDevice.]] .. wanInstanceNumber .. [[.WANPPPConnection.]].. pppoeInstanceNumber ..[[.Enable</Name>
            <Value xsi:type="xsd:boolean">1</Value>
        </ParameterValueStruct>
        <ParameterValueStruct>
            <Name>InternetGatewayDevice.WANDevice.1.WANConnectionDevice.]] .. wanInstanceNumber .. [[.WANPPPConnection.]].. pppoeInstanceNumber ..[[.Username</Name>
            <Value xsi:type="xsd:string">]] .. get_variable_value("pppoe_username") .. [[</Value>
        </ParameterValueStruct>
        <ParameterValueStruct>
            <Name>InternetGatewayDevice.WANDevice.1.WANConnectionDevice.]] .. wanInstanceNumber .. [[.WANPPPConnection.]].. pppoeInstanceNumber ..[[.Password</Name>
            <Value xsi:type="xsd:string">]] .. get_variable_value("pppoe_password") .. [[</Value>
        </ParameterValueStruct>
        <ParameterValueStruct>
            <Name>InternetGatewayDevice.WANDevice.1.WANConnectionDevice.]] .. wanInstanceNumber .. [[.WANPPPConnection.]].. pppoeInstanceNumber ..[[.Name</Name>
            <Value xsi:type="xsd:string">internet-pppoe</Value>
        </ParameterValueStruct>
      </ParameterList>
      <ParameterKey>LC1309946174</ParameterKey>
    </cwmp:SetParameterValues>
  </soap:Body>
</soap:Envelope>
    ]]
end

local function createCWMPChangeWanToPPPoE()
    return [[
        <?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:cwmp="urn:dslforum-org:cwmp-1-0" xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:schemaLocation="urn:dslforum-org:cwmp-1-0 ..schemaswt121.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <soap:Header/>
  <soap:Body soap:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
    <cwmp:SetParameterValues>
      <ParameterList soapenc:arrayType="cwmp:ParameterValueStruct[1]">
        <ParameterValueStruct>
            <Name>InternetGatewayDevice.WANDevice.1.WANCommonInterfaceConfig.X_AVM-DE_InternetConnectionLinkType</Name>
            <Value xsi:type="xsd:string">PPPoE</Value>
        </ParameterValueStruct>
      </ParameterList>
      <ParameterKey>LC1309986174</ParameterKey>
    </cwmp:SetParameterValues>
  </soap:Body>
</soap:Envelope>
    ]]
end 

print("Processing device SN: " .. get_device_id() .. ", adding PPPoE WAN connection")
--add PPPoE WAN Connection
local addPPPoEMsgResult = send_cwmp_message_and_forget(get_device_id(), createCWMPAddPPPoEWANMsg(1), 2)
if type(addPPPoEMsgResult) == "table" or addPPPoEMsgResult == false then
    print("Failed to add PPPoE WAN, SN: " ..
    get_device_id() ..
    " | Error Message: " ..
    addPPPoEMsgResult["error_message"] .. " | Code: " .. addPPPoEMsgResult["error_code"])
    set_provisioning_status("failed")
else

    print("Added PPPoE WAN successfully done, SN: " .. get_device_id() .. ", now setting credentials and changing WAN to PPPoE")
    -- Set PPPoE credentials
    local setPppoeCredentialsMsgResult = send_cwmp_message_and_forget(get_device_id(), createCWMPSetPPPoECredentialsMsg(1, 1), 1)
    
    if type(setPppoeCredentialsMsgResult) == "table" or setPppoeCredentialsMsgResult == false then
        print("Failed to set PPPoE credentials, SN: " .. get_device_id() .. " | Error Message: " .. setPppoeCredentialsMsgResult["error_message"] .. " | Code: " .. setPppoeCredentialsMsgResult["error_code"])
        set_provisioning_status("failed")
    else
        
        print("Set PPPoE credentials successfully done, SN: " .. get_device_id() .. ", now changing WAN to PPPoE")
        -- Change WAN from IPoE to PPPoE
        local changeWanToPPPoEMsgResult = send_cwmp_message_and_forget(get_device_id(), createCWMPChangeWanToPPPoE(), 1)
        
        if type(changeWanToPPPoEMsgResult) == "table" or changeWanToPPPoEMsgResult == false then
            print("Failed to change WAN to PPPoE, SN: " .. get_device_id() .. " | Error Message: " .. changeWanToPPPoEMsgResult["error_message"] .. " | Code: " .. changeWanToPPPoEMsgResult["error_code"])
            set_provisioning_status("failed")
        else
            print("Changed WAN to PPPoE successfully done")
            if refresh_device_parameters(get_device_id()) == false then
                print("Failed to refresh device parameters")
                set_provisioning_status("failed")
            else
                print("Finished processing device SN: " .. get_device_id() .. ", messages queued")
                set_provisioning_status("provisioned")
            end
        end
    end
end
```

## Service Instances

The service instance is the attachment of a service to a CPE, defining the specific variable values for that equipment. This will be applied the first time the CPE connects to the platform and any time it is factory reset.

### Event

Oktopus listens to CPE events and applies the services configuration on the equipment when an event of "0 BOOSTRAP" — first connection­ or factory reset — is triggered by the CPE.

### Status

The service attached to the CPE has a few statuses:

* Pending = It's waiting for an event to start running and being applied to the equipment.
* Running = The event has triggered, and the script is being executed.
* Provisioned = Execution was a success, the CPE received all the messages.&#x20;
* Failed = An issue happened during the execution of the script.

## In Practice

After learning the concepts, let's understand how it works in a real use case.&#x20;

### Configure the CPE to connect to the ACS

The CPE must have a way to connect to the Controller system. You can use one of these options:

* Firmware with ACS pre-configured
* DHCP with option 43
* PPPoE reply with ACS server URL

{% stepper %}
{% step %}
### Add a CPE

Manually insert a new device with its unique identifier (e.g, serial number, agent endpoint id).
{% endstep %}

{% step %}
### Create a service definition

Write the Lua scripts and necessary variables to apply that service to the CPE.
{% endstep %}

{% step %}
### Attach the service to the CPE

Associate the CPE with a service, with its specific variable values for that device.
{% endstep %}

{% step %}
### Plug the CPE into the network

As the CPE already has the ACS configured and a connection to it, all services will be provisioned when it establishes communication with the controller system, and the device will come online with the customer's specific configuration.
{% endstep %}
{% endstepper %}
