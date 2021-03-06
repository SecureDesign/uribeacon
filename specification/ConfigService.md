# UriBeacon Configuration Service 

***Revision v2r1***

The material contained on this page is informative only and subject to change. 

## 1 Introduction

The UriBeacon Config Service allows setting UriBeacon fields and transmission characteristics. This information includes the following:

*   Lock code
*   Uri
*   Flags
*   Transmit Power
*   Duty Cycle

This document is formatted according to the Bluetooth 
[service](https://developer.bluetooth.org/TechnologyOverview/Pages/ANS.aspx) and 
[profile](https://developer.bluetooth.org/gatt/profiles/Pages/ProfileViewer.aspx?u=org.bluetooth.profile.blood_pressure.xml) formatting styles.


### 1.1 Service Dependencies

This service is not dependent upon any other [services](https://developer.bluetooth.org/gatt/services/Pages/ServicesHome.aspx).

### 1.2 Transport Dependencies

|Transport   | Supported |
|:-----------|-----------|
| Classic    | false	 |
| Low Energy | true      |

### 1.3 Error Codes

| Code   | Description                |
|:-------|:---------------------------|
| 0x08   | Insufficient Authorization |
| 0x0d   | Invalid Attribute Length   |


## 2 Service Declaration

The assigned number for `<<UriBeacon Config Service>>` is

        ee0c2080-8786-40ba-ab96-99b91ac981d8

## 3 Service Characteristics


| Characteristic             | Ref. | Requirement  | 
|:---------------------------|:-----|:--------------------|
| Lock State                 | [3.1](#31-lock-state) | Mandatory|
| Lock                       | [3.2](#32-lock) | Optional| 
| Unlock                     | [3.3](#33-unlock) | Optional|
| Uri Data                   | [3.4](#34-uri-data) | Mandatory|
| Uri Flags                  | [3.5](#35-flags)  | Mandatory|
| Advertised Tx Power Levels | [3.6](#36-advertised-tx-power-levels)  | Mandatory|
| Tx Power Mode              | [3.7](#37-tx-power-mode) | Mandatory |
| Beacon Period              | [3.8](#38-beacon-period)| Mandatory|
| Reset                      | [3.9](#39-reset) | Mandatory|

||<sup>Broadcast</sup>|<sup>Read</sup>|<sup>Write<br>without<br> Response</sup>|<sup>Write</sup>|<sup>Notify</sup>|<sup>Indicate</sup>|<sup>Signed Write</sup>|<sup>Reliable Write</sup>|<sup>Writable Auxiliaries</sup>|
|-------------------------:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|Lock State                |X|M|X|X  |X|X|X|X|X|
|Lock                      |X|X|X|O\*|X|X|X|X|X|
|Unlock                    |X|X|X|O  |X|X|X|X|X|
|Uri Data                  |X|M|X|M\*|X|X|X|X|X|
|Uri Flags                 |X|M|X|M\*|X|X|X|X|X|
|Advertised Tx Power Levels|X|M|X|M\*|X|X|X|X|X|
|Tx Power Mode             |X|M|X|M\*|X|X|X|X|X|
|Beacon Period             |X|M|X|M\*|X|X|X|X|X|
|Reset                     |X|X|X|M\*|X|X|X|X|X|

\* Must be in unlock state.

### 3.1 Lock State

| Name        | Lock State                                  |
|:------------|:--------------------------------------------|
| UUID        | ee0c<b>2081</b>-8786-40ba-ab96-99b91ac981d8 |
| Description | Reads the lock state.                       |
| Type        | boolean                              |
| Lock State  | Either locked or unlocked.                  |

Read returns true if the device is locked.

### 3.2 Lock

| Name | Lock |
|:------------|:--------------------------------------------|
| UUID  | ee0c<b>2082</b>-8786-40ba-ab96-99b91ac981d8|
|  Description| Locks the beacon. |
|  Type | uint128 |
|  Lock State | Must be unlocked. Will be locked after successful write.|

### 3.3 Unlock

| Name | Unlock |
|:------------|:--------------------------------------------|
| UUID  | ee0c<b>2083</b>-8786-40ba-ab96-99b91ac981d8|
|  Description| Unlocks the beacon. |
|  Type | uint128 |
|  Lock State | Will be unlocked after successful write.|

If the beacon is unlocked then the write will return success regardless of the
contents of the parameter.

### 3.4 Uri Data

| Name | Uri Data |
|:------------|:--------------------------------------------|
| UUID  | ee0c<b>2084</b>-8786-40ba-ab96-99b91ac981d8|
|  Description| Reads/writes the Uri. |
|  Type | uint8[]|
|  Lock State | For write, must be unlocked.|

The Uri Data characteristic is a variable length structure. The first byte contains the [Uri Scheme Prefix](https://github.com/google/uribeacon/tree/master/specification#uribeacon-uri-scheme-prefix).
The remaining bytes contain either [urn:uuid encoding](https://github.com/google/uribeacon/tree/master/specification#uribeacon-urnuuid-encoding) or  [HTTP URL encoding](https://github.com/google/uribeacon/tree/master/specification#uribeacon-http-url-encoding).

Note: Uri Data must be between 0 and 18 bytes in length.

### 3.5 Flags

| Name | Flags |
|:------------|:--------------------------------------------|
| UUID  | ee0c<b>2085</b>-8786-40ba-ab96-99b91ac981d8|
|  Description| Reads/writes the flags. |
|  Type | uint8 |
|  Lock State | For write, must be unlocked.|

The Flags characteristic is a sinlge unsigned byte value containing the
[UriBeacon Flags](https://github.com/google/uribeacon/tree/master/specification#uribeacon-flags).

### 3.6 Advertised TX Power Levels

| Name | Advertised TX Power Levels |
|:------------|:--------------------------------------------|
| UUID  | ee0c<b>2086</b>-8786-40ba-ab96-99b91ac981d8|
|  Description| Reads/writes the Advertised Power Levels array. |
|  Type | int8[4] |
|  Lock State | For write, must be unlocked.|

This characteristic is a fixed length array of values, in dBm, to be included in the 
[UriBeacon TX Power Level](https://github.com/google/uribeacon/tree/master/specification#uribeacon-tx-power-level) field of the advertisement when that mode is active. The index into the array is [TX Power Mode](#37-tx-power-mode). 

Note to developers: The Advertised TX Power Levels is not the same as values set internally into the radio tx power. You need to implement an internal array indexed by TX Power Mode that is used for the internal facing radio setting. The Advertised TX Power Levels is also indexed by TX Power Mode but is outward facing, and is the value that is put into the adv packet. These are related but distinct because numbers used by radio function may not be the same as those exposed in adv packets.

### 3.7 TX Power Mode

| Name | TX Power Mode |
|:------------|:--------------------------------------------|
| UUID  | ee0c<b>2087</b>-8786-40ba-ab96-99b91ac981d8|
|  Description| Reads/writes the TX Power Mode. |
|  Type | uint8 |
|  Lock State | For write, must be unlocked.|

Sets the transmission power mode to one of:

| Power Level | Value |
|:-----|:----------|
| TX_POWER_MODE_HIGH | 3 |
| TX_POWER_MODE_MEDIUM | 2 | 
| TX_POWER_MODE_LOW | 1 | 
| TX_POWER_MODE_LOWEST | 0 |


### 3.8 Beacon Period

| Name        | Beacon Period                               |
|:------------|:--------------------------------------------|
| UUID        | ee0c<b>2088</b>-8786-40ba-ab96-99b91ac981d8 |
| Description | The period in milliseconds that a UriBeacon packet is transmitted. |
| Type        | uint16                                      |
| Lock State  | For write, must be unlocked.                |

The period in milliseconds that a UriBeacon packet is transmitted. A value of zero disables UriBeacon transmissions.

### 3.9 Reset

| Name        | Reset                                       |
|:------------|:--------------------------------------------|
| UUID        | ee0c<b>2089</b>-8786-40ba-ab96-99b91ac981d8 |
| Description | Reset to default values.                    |
| Type        | boolean                                     |
|  Lock State | Must be unlocked.                           |

Writing a non-zero value to this characteristic will set all characteristics to their initial values:

| Characteristic | Default Value |
|:---------------|:--------------|
| Uri Data | vendor specified |
| Uri Flags | 0 |
| TX Power Mode | TX_POWER_MODE_LOW |
| Beacon Period | 1000 (1 second) |
| Lock | 00000000-00000-0000-0000-000000000000 |

### 4 Reserved Characteristics

The following Characteristic UUIDs are are reserved for future use:

| UUID |
|:----------|
|ee0c<b>2090</b>-8786-40ba-ab96-99b91ac981d8|




