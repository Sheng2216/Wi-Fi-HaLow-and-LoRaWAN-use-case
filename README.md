# Wi-Fi HaLow and LoRaWAN use case with RAK7391

## 1. Introduction

This guide explains how to use [Wi-Fi HaLow]() module [AHPI7292S](https://www.alfa.com.tw/products/ahpi7292s) in combination with LoRaWAN concentrators on RAK7391.

## 2. Hardware

- 2x AHPI7292S
  
- 2x antenna
  
- 1x LoRaWAN concentrator(use RAK5146 USB with GPS as an example)
  
- 2x RAK7391
  

## 3. Software

We provide a specfic [RAKPiOS image](https://drive.google.com/file/d/1QrR1LJYv7bMaHwGgTRIq-LPaGU8VzoKg/view?usp=sharing) for AHPI7292S, which has set everything up, you just need to download and flash it to your RAK7391. Notice that the default username is `rak`, and the password is `rakpios`.

## 4. Usage

There is a NRC7292 Software Package in the folder */home/rak/nrc_pkg* , and “start.py” in folder */home/rak/nrc_pkg/script* is the unified script used to initiate AP, STA.

The following is the parameters for start.py script file.

```
Usage:
        start.py [sta_type] [security_mode] [country] [channel] [sniffer_mode]
        start.py [sta_type] [security_mode] [country] [mesh_mode] [mesh_peering] [mesh_ip]
Argument:
        sta_type      [0:STA   |  1:AP  |  2:SNIFFER  | 3:RELAY |  4:MESH]
        security_mode [0:Open  |  1:WPA2-PSK  |  2:WPA3-OWE  |  3:WPA3-SAE | 4:WPS-PBC]
        country       [US:USA  |  JP:Japan  |  TW:Taiwan  | EU:EURO | CN:China |
                       AU:Australia  |  NZ:New Zealand]
        -----------------------------------------------------------
        channel       [S1G Channel Number]   * Only for Sniffer
        sniffer_mode  [0:Local | 1:Remote]   * Only for Sniffer
        mesh_mode     [0:MPP | 1:MP | 2:MAP] * Only for Mesh
        mesh_peering  [Peer MAC address]     * Only for Mesh
        mesh_ip       [Static IP address]    * Only for Mesh
Example:
        OPEN mode STA for US                : ./start.py 0 0 US
        Security mode AP for US                : ./start.py 1 1 US
        Local Sniffer mode on CH 40 for Japan  : ./start.py 2 0 JP 40 0
        SAE mode Mesh AP for US                : ./start.py 4 3 US 2
        Mesh Point with static ip              : ./start.py 4 3 US 1 192.168.222.1
        Mesh Point with manual peering         : ./start.py 4 3 US 1 8c:0f:fa:00:29:46
        Mesh Point with manual peering & ip    : ./start.py 4 3 US 1 8c:0f:fa:00:29:46 192.168.222.1
Note:
        sniffer_mode should be set as '1' when running sniffer on remote terminal
        MPP, MP mode support only Open, WPA3-SAE security mode
```

#### Access Point (AP) running procedure

The following shows an example of AP running for US channel, and its channel can be configured in *nrc_pkg/script/conf/US/ap_halow_open.conf*. For WPA2/WPA3 modes, please refer to *nrc_pkg/script/conf/US/ap_halow_[sae|owe|wpa2].conf* files.

```
cd nrc_pkg/script
./start.py 1 0 US
```

#### Station (STA) running procedure

The following shows an example of STA running for US channel, and its channel can be configured in *nrc_pkg/script/conf/US/sta_halow_open.conf*. For WPA2/WPA3 modes, please refer to *nrc_pkg/script/conf/US/sta_halow_[sae|owe|wpa2].conf* files.

```
cd nrc_pkg/script
./start.py 0 0 US
```

#### LoRaWAN network deployment procedure

Now you should have the Wi-Fi HaLow AP and station ready, the next step is to set up a typical LoRaWAN deployment involving a UDP packet forwarder on the Wi-Fi HaLow station and Chirpstack on the Wi-Fi HaLow AP, follow the steps below:

1. Download the repository on **both** gateways using the following command:

```bash
git clone https://github.com/Sheng2216/Wi-Fi-HaLow-and-LoRaWAN-use-case.git
```

2. On the Wi-Fi HaLow station gateway, navigate to the `station` directory and run the following command to bring up the UDP packet forwarder:

```bash
cd ./Wi-Fi-HaLow-and-LoRaWAN-use-case/station
docker-compose up
```

Wait until the UDP packet forwarder is deployed. Take note that in the `docker-compose.yml` file, the `SERVER_HOST` should be set to `192.168.200.1`, which is the IP address of the Wi-Fi HaLow AP.

3. For the Wi-Fi HaLow AP, go to the `AP` directory and deploy Chirpstack, similar to the steps above. Run the following command:

```bash
cd ./Wi-Fi-HaLow-and-LoRaWAN-use-case/AP
docker-compose up
```

4. Once the deployment is complete, you can log in to the Chirpstack's web page and add the gateway. The gateway EUI is configured as `AAAAAAAAAAAAAAAA` for testing purposes. If needed, you can modify it in the `docker-compose.yml` file located in the `station` directory.

By following these steps, you should have a functioning Wi-Fi HaLow AP and station with a UDP packet forwarder and Chirpstack deployed respectively.
