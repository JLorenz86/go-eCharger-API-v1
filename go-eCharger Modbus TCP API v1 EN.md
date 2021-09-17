# GO-E Charger Modbus TCP/IP Documentation English

| Version | Date       | Author      | Description     |
| ------- | ---------- | ----------- | --------------- |
| 1.0     | 2020-10-01 | Peter Pötzi | Initial version |

# Index

* [Activation](#activation)
* [Connection](#connection)
* [Modbus Register](#modbusregister)
* [Code example](#example)
    - [ Connect to Charger ](#connect)
    - [ Get Data from Input Register ](#getinputregister)
    - [ Get Data from Holding Register ](#getholdingregister)
    - [ Set Data for Holding Register ](#setholdingregister)
    - [ Close Socket ](#close)
    - [ Open Socket ](#open)

### Documentation to the used libraries

* [net](https://nodejs.org/api/net.html)
* [jsmodbus](https://www.npmjs.com/package/jsmodbus)

<a id="activation"></a>
## API activation

To activate the API, you have to allow API access in the go-eCharger app under Advanced Settings.<br> After activation, the charging box must be restarted.<br>
The API is only available for firmware version 0.40 and newer.

<a id="connection"></a>
## Connection

The Modbus TCP connection goes as follows:

| Connection | Port |
| ---------- | ---- |
| WLAN       | 502  |

The go-e Charger has the unitId of 1.

<a id="modbusregister"></a>
## Modbus Register

The Register ID is 0-indexed. Because of that some clients have to increase the Register ID by 1.

Registers are designed as holding registers or input registers.

* Holding Register allows reading and writing.
* Input Register allows only reading.

For values ​​that are distributed over several registers are all 16-Bit packages a big-endian.

| Register                               | Name                 | Register Type      | Data type              | Lenght | Description                                                                                                                                                                                                                                                                                                                                                   |
| -------------------------------------- | --------------------------- | ----------------- | --------------------- | ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 100                                    | CAR_STATE                   | Input Register    | unsigned integer (16) | 1     | **Status PWM signaling**<br>0: Unknown, Charging station defective<br>1: Charging station ready, no car<br>2: Car is charging<br>3: Waiting for car<br>4: Charging finnished, Car still connected                                                                                                                                      |
| 101                                    | PP_CABLE                    | Input Register    | unsigned integer (16) | 1     | Type2 **Cable ampere coding**<br>13-32: ampere coding<br>0: no cable                                                                                                                                                                                                                                                                                |
| 105 <br>106                            | FWV                         | Input Register    | ascii (4 byte)        | 2     | Firmware version as ASCII                                                                                                                                                                                                                                                                                                                                 |
| 107                                    | ERROR                       | Input Register    | unsigned integer (16) | 1     | error:<br>1: RCCB (Residual current circuit breaker)<br>3: PHASE (phase disturbance)<br>8: NO_GROUND (Ground detection)<br>10, default: INTERNAL (Others)                                                                                                                                                                                                          |
| 108<br>109                             | VOLT_L1                     | Input Register    | unsigned integer (32) | 2     | Voltage on L1 in volts                                                                                                                                                                                                                                                                                                                                    |
| 110 <br>111                            | VOLT_L2                     | Input Register    | unsigned integer (32) | 2     | Voltage on L2 in volts                                                                                                                                                                                                                                                                                                                                    |
| 112 <br>113                            | VOLT_L3                     | Input Register    | unsigned integer (32) | 2     | Voltage on L3 in volts                                                                                                                                                                                                                                                                                                                                    |
| 114<br>115                             | AMP_L1                      | Input Register    | unsigned integer (32) | 2     | Amps on L1 in 0.1A (123 equals 12, 3A)                                                                                                                                                                                                                                                                                                               |
| 116<br>117                             | AMP_L2                      | Input Register    | unsigned integer (32) | 2     | Amps on L2 in 0.1A (123 equals 12, 3A)                                                                                                                                                                                                                                                                                                               |
| 118 <br>119                            | AMP_L3                      | Input Register    | unsigned integer (32) | 2     | Amps on L3 in 0.1A (123 equals 12, 3A)                                                                                                                                                                                                                                                                                                               |
| 120<br>121                             | POWER_TOTAL                 | Input Register    | unsigned integer (32) | 2     | Total power in 0.01kW (360 corresponds to 3.6kW)                                                                                                                                                                                                                                                                                                           |
| 128 <br>129                            | ENERGY_TOTAL                | Input Register    | unsigned integer (32) | 2     | Total amount of energy charged in 0.1kWh                                                                                                                                                                                                                                                                                                                     |
| 132<br>133                             | ENERGY_CHARGE               | Input Register    | unsigned integer (32) | 2     | **Amount of energy charged** in <br> Deka-Watt-Seconds <br> Example: 100, 000 means 1, 000, 000 <br> Ws (= 277Wh = 0.277kWh) were loaded in this charging process.                                                                                                                                                                                              |
| 144 <br>145                            | VOLT_N                      | Input Register    | unsigned integer (32) | 2     | Voltage on N in volts                                                                                                                                                                                                                                                                                                                                     |
| 146 <br>147                            | POWER_L1                    | Input Register    | unsigned integer (32) | 2     | Power on L1 in 0.1kW (36 corresponds to 3.6kW)                                                                                                                                                                                                                                                                                                             |
| 148<br>149                             | POWER_L2                    | Input Register    | unsigned integer (32) | 2     | Power on L2 in 0.1kW (36 corresponds to 3.6kW)                                                                                                                                                                                                                                                                                                             |
| 150 <br>151                            | POWER_L3                    | Input Register    | unsigned integer (32) | 2     | Power on L3 in 0.1kW (36 corresponds to 3.6kW)                                                                                                                                                                                                                                                                                                             |
| 152 <br>153                            | POWER_FACTOR_L1             | Input Register    | unsigned integer (32) | 2     | Power factor on L1 in %                                                                                                                                                                                                                                                                                                                                |
| 154<br>155                             | POWER_FACTOR_L2             | Input Register    | unsigned integer (32) | 2     | Power factor on L2 in %                                                                                                                                                                                                                                                                                                                                |
| 156<br>157                             | POWER_FACTOR_L3             | Input Register    | unsigned integer (32) | 2     | Power factor on L3 in %                                                                                                                                                                                                                                                                                                                                |
| 158 <br>159                            | POWER_FACTOR_N              | Input Register    | unsigned integer (32) | 2     | Power factor on N in %                                                                                                                                                                                                                                                                                                                                 |
| 200                                    | ALLOW                       | Holding Register  | unsigned integer (16) | 1     | **allow_charging:** PWM Signal is allowed<br>to abut<br>0: no<br>1: yes                                                                                                                                                                                                                                                                                        |
| 201                                    | ACCESS_STATE                | Holding Register  | unsigned integer (16) | 1     | **access_state**: access control.<br>0: open<br>1: RFID / App needed<br>2: electricity price / automatic<br>3: scheduler                                                                                                                                                                                                                                   |
| 202                                    | ADAPTER_INPUT               | Input Register    | unsigned integer (16) | 1     | **adapter_in**: The charging box<br>is connected with an adapter<br>0: NO_ADAPTER<br>1: 16A_ADAPTER                                                                                                                                                                                                                                                                   |
| 203                                    | UNLOCKED_BY                 | Input Register    | unsigned integer (16) | 1     | Number of the RFID card that <br>activated the current charging process<br>                                                                                                                                                                                                                                                                               |
| 204                                    | CABLE_LOCK_MODE             | Holding Register  | unsigned integer (16) | 1     | Cable lock setting<br>0: Lock while the car is connected<br>1: Unlock automatically after<br>charging<br>2: Always keep the cable locked                                                                                                                                                                                                |
| 205                                    | PHASES                      | Input Register    | unsigned integer (16) | 1     | **Phases** before and after the contactor<br>binary flags: 0b00ABCDEF<br>A... phase 3, before the contactor<br> B... phase 2 before the contactor<br>C... phase 1 before the contatctor<br>D... phase 3 after the contactor<br>E... phase 2 after the contactor<br>F... phase 1 after the contactor<br>pha 0b00001000: Phase 1 is<br>available<br>pha 0b00111000: Phase1-3 is<br>available<br> |
| 206                                    | LED_BRIGHTNESS              | Holding Register  | unsigned integer (16) | 1     | **LED brightness** from 0-255<br>0: LED off<br>255: LED brightness max                                                                                                                                                                                                                                                                                   |
| 207                                    | LED_SAVE_ENERGY             | Holding Register  | unsigned integer (16) | 1     | **led_save_energy**: LED switch off<br> automatically after 10 seconds<br>0: Energy saving function deactivated<br>1: Energy saving function activated                                                                                                                                                                                                              |
| 208                                    | ELECTRICITY_PRICES_HOURS    | Holding Register  | unsigned integer (16) | 1     | Minimum **count** ​of hours that must be<br>charged with "Electricity price - automatic"<br>Example: 2 ("Car is full enough after 2 hours")                                                                                                                                                                                                     |
| 209                                    | ELECTRICITY_PRICES_FINISHED | Holding Register  | unsigned integer (16) | 1     | Hour (**​time**) in which with "electricity price<br>- automatic" the charge must have lasted<br>at least aho hours.<br>Example: 7 ("Ready by 7:00, so charged at <br>least 2 hours before that")                                                                                                      |
| 210                                    | ELECTRICITY_PRICES_ZONE     | Holding Register  | unsigned integer (16) | 1     | Awattar price zone<br>0: Austria<br>1: Germany                                                                                                                                                                                                                                                                                                       |
| 211                                    | AMPERE_MAX                  | Holding Register  | unsigned integer (16) | 1     | Absolute Max Amps: Maximum<br>value for Amps setting<br>Example: 20 (setting to more than<br>20A in the app not possible)                                                                                                                                                                                                                  |
| 212                                    | AMPERE_L1                   | Holding Register  | unsigned integer (16) | 1     | Ampere level 1 for push button on the<br>device.<br>6-32: Ampere level activated<br>0: Level deactivated (will be skipped)                                                                                                                                                                                                                                      |
| 213                                    | AMLERE_L2                   | Holding Register  | unsigned integer (16) | 1     | Ampere level 2 for push button on the<br>device.<br>6-32: Ampere level activated<br>0: Level deactivated (will be skipped)                                                                                                                                                                                                                                      |
| 214                                    | AMPERE_L3                   | Holding Register  | unsigned integer (16) | 1     | Ampere level 3 for push button on the<br>device.<br>6-32: Ampere level activated<br>0: Level deactivated (will be skipped)                                                                                                                                                                                                                                      |
| 215                                    | AMPERE_L4                   | Holding Register  | unsigned integer (16) | 1     | Ampere level 4 for push button on the<br>device.<br>6-32: Ampere level activated<br>0: Level deactivated (will be skipped)                                                                                                                                                                                                                                      |
| 216                                    | AMPERE_L5                   | Holding Register  | unsigned integer (16) | 1     | Ampere level 5 for push button on the<br>device.<br>6-32: Ampere level activated<br>0: Level deactivated (will be skipped)                                                                                                                                                                                                                                      |
| 217                                    | CLOUD_DISABLED              | Holding Register  | unsigned integer (16) | 1     | **Cloud disabled**<br>0: cloud enabled<br>1: cloud disabled                                                                                                                                                                                                                                                                                                |
| 218                                    | NORWAY_MODE                 | Holding Register  | unsigned integer (16) | 1     | **Norway mode**​ activated<br>0: deactivated (ground detection <br> activated)<br>1: activated (no ground detection, <br>only intended for IT networks)                                                                                                                                                                                                      |
| 299                                    | AMPERE_VOLATILE             | Holding Register  | unsigned integer (16) | 1     | Amps Value for the PWM<br>signaling in whole amps of<br>**6-32A**<br><br>Is not saved in the EEPROM <br> and is **reset** to the value last saved in the EEPROM during the next boot process.<br><br> For energy control                                                                                                   |
| 300                                    | AMPERE_EEPROM               | Holding Register  | unsigned integer (16) | 1     | Amps Value for the PWM<br>signaling in whole amps of<br>**6-32A**<br><br>Is saved in the EEPROM (max. <br> write cycles approx. 100, 000)                                                                                                                                                                                                        |
| 301<br>302<br>303                      | MAC                         | Input Register    | unsigned integer (48) | 3     | MAC address of the WLAN station, binary                                                                                                                                                                                                                                                                                                                        |
| 304<br>305<br>306<br>307<br>308<br>309 | SNR                         | Input Register    | ascii (12 byte)       | 6     | Serial number of the go-eCharger, as <br> ASCII                                                                                                                                                                                                                                                                                                                 |
| 310<br>311<br>312<br>313<br>314<br>315 | HOSTNAME                    | Input Register    | ascii (12 byte)       | 6     | Host name of the go-eCharger, as ASCII                                                                                                                                                                                                                                                                                                                        |
| 315<br>316<br>317<br>318               | IP                          | Input Register    | ascii (8 byte)        | 4     | IP address of the go-eCharger, 1 byte per register                                                                                                                                                                                                                                                                                                             |
| 319<br>320<br>321<br>322               | SUBNET                      | Input Register    | unsigned integer (64) | 4     | Subnet mask of the go-eCharger, 1 <br> byte per register                                                                                                                                                                                                                                                                                                       |
| 323<br>324<br>325<br>326               | GATEWAY                     | Input Register    | ascii (4 byte)        | 4     | Go-eCharger gateway, 1 byte per register                                                                                                                                                                                                                                                                                                                |

<a id="example"></a>
<a id="connect"></a>

## Connect to Charger

``` javascript
// clientE.js
const net = require("net"); // import net
const modbus = require("jsmodbus"); // import jsmodbus
const socket = new net.Socket(); // create new socket
const client = new modbus.client.TCP(socket, 1); // create client that has been given the socket and the unitId as parameters
const options = { // config for connection
    port: 502, // port
    host: 'xxx.xxx.xxx.xxx' // ip
};

socket.on("error", (err) => { // when the computer has an error while connecting
    console.log(err); // prints error in the console
})
openSocket(socket, options);
```

<a id="getinputregister"></a>

## Get Data from Input Register

``` javascript
/**
 * @function
 * @name getInputRegisters
 * @param {object} socket 
 * @param {object} client 
 * @param {number} register - attribute from the column "Register" of the GO-E Charger Modbus API
 * @param {number} count - attribute from the column "Länge" of the GO-E Charger Modbus API 
 * 
 * This function prints the JSON Object that the Charger Sends 
 * 
 */
function getInputRegisters(socket, client, register, count) {
    socket.on("connect", function() { // when the computer connects to the given ip
        /**
         * @method 
         * @name readInputRegisters - read only option from Modbus (From the GO-E Charger Modbus API --> column Register Typ)
         * @param {number} register - attribute from the column "Register" of the GO-E Charger Modbus API
         * @param {number} count - attribute from the column "Länge" of the GO-E Charger Modbus API 
         */
        client.readInputRegisters(register, count).then(function(response) {
            console.log(response); // prints response in the console

        }).catch(function(err) {
            console.log(err); // prints error in the console
            closeSocket(socket);

        });
    })
}
```

<a id="getholdingregister"></a>

## Get Data from Holding Register

``` javascript
/**
 * @function
 * @name getHoldingRegisters
 * @param {object} socket 
 * @param {object} client 
 * @param {number} register - attribute from the column "Register" of the GO-E Charger Modbus API
 * @param {number} count - attribute from the column "Länge" of the GO-E Charger Modbus API 
 * 
 * This function prints the JSON Object that the Charger Sends 
 * 
 */
function getHoldingRegisters(socket, client, register, count) {
    socket.on("connect", function() { // when the computer connects to the given ip
        /**
         * @method 
         * @name readHoldingRegisters - read and write option from Modbus
         * @param {number} register - attribute from the column "Register" of the GO-E Charger Modbus API
         * @param {number} count - attribute from the column "Länge" of the GO-E Charger Modbus API 
         */
        client.readHoldingRegisters(register, count).then(function(response) {
            console.log(response); // prints response in the console

        }).catch(function(err) {
            console.log(err); // prints error in the console
            closeSocket(socket);

        });
    })
}
```

<a id="setholdingregister"></a>

## Set Data for Holding Register

``` javascript
/**
 * @function
 * @name setHoldingRegisters
 * @param {object} socket 
 * @param {object} client 
 * @param {number} register - attribute from the column "Register" of the GO-E Charger Modbus API
 * @param {number} value - attribute to set the value of the given register
 * 
 * This function sets the value of a given register
 * 
 */
function setHoldingRegisters(socket, client, register, value) {
    socket.on("connect", function() {
        /**
         * @method 
         * @name writeSingleRegister - read and write option from Modbus
         * @param {number} register - attribute from the column "Register" of the GO-E Charger Modbus API
         * @param {number} value - attribute to set the value of the given register
         */
        client.writeSingleRegister(register, value).then(function(response) {
            console.log(response); // prints the response
        }).catch(function(err) {
            console.log(err); // prints the error
            closeSocket(socket);
        })
    })
}
```

<a id="close"></a>

## Close Socket

``` javascript
/**
 * @function
 * @name closeSocket
 * @param {object} socket 
 * 
 * This function closes the socket
 * 
 */
function closeSocket(socket) {
    socket.end();
}
```

<a id="open"></a>

## Open Socket

``` javascript
/**
 * @function
 * @name openSocket
 * @param {object} socket 
 * @param {object} option - config for connection
 * 
 * This function opens the socket
 *  
 */
function openSocket(socket, option) {
    socket.connect(options); // connect to the socket with the given config
}
```
