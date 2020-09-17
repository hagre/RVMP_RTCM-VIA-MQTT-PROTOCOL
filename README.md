# RVMP (RTCM-VIA-MQTT-PROTOCOL)

RVMP (RTCM-VIA-MQTT-PROTOCOL) V0.3 (first very basic approach, but some features for future are in preperation). (GNU GPL V3)

In use by RVMT (RTCM Via MQTT Transmitter - https://github.com/hagre/RVMT_RTCM-VIA-MQTT-TRANSMITTER) using the MQTT msg/protocol (as a secure and opensource alternative to NTRIP) to get RTK correction data from BaseStations to Broker and to Rover GPS units.

Caster is/will be on https://github.com/hagre/RVMT_RTCM-VIA-MQTT-CASTER

Basicly, this is a try to implement something like a NTRIP caster with MQTT features. There are a lot of similarities to NTRIP (V1.0/V2.0), because it is a known and usefull protocol with most needed features for this task.

Attension: Topics are casesensitive!!

The basic system setup can be:

For most users with 1 Base and 1 or more Rovers -----------------> My focus until V2.0

1.) Base (Transmitter) - via MQTT - Broker/Server - via MQTT - Client/Receiver/Rover

For users with more than 1 Base and different "customers" with lots of rovers (accountmanagement) -----------------> Not until V2.0

2.) Base (Transmitter) - via MQTT - Broker/Server - via MQTT - Caster (Relaying/managing the Topics) - via MQTT - Client/Receiver/Rover
                                      

# Basic Workflow Description: V0.3
+ Basestation(s) is/are publishing to MQTT broker (RTK/Base/...) and deliver basic information 
+ Basestationen are continuosly evaluating the RTCM input and presenting the avaliable feature (incl update if something new,..)

+ Clients can connect to the broker (user/password protected and TLS)
+ Clients can select to subscribe to the required RTCM msg from the desired RTK/Base/... 
+ Clients can switch off publishing of the Basestations publishing RTCM (maintaining in STBY) if the last client has closed the subscription.


# More Details on the used Topics:

# Basestation publish:
Welcome msg: (transmitted when a new connection is established, or a feature is updated, .. )
+ RTK/Base/(NAME of BASE)/Status/FirmwareVer - e.g "2.2" - Version of transmitter firmware [QOS = 1, retain = true] [read acces by client app, read/write by Basestation ]
+ RTK/Base/(NAME of BASE)/Status/ProtocolVer - e.g "0.1" - Version of protocol [QOS = 1, retain = true] [read acces by clent, read/write by Basestation ]
+ RTK/Base/(NAME of BASE)/Status/RTCM/(TYPE of MSG) - e.g "1", "0.5" or "30" - time in seconds between update (~frequenz) [QOS = 1, retain = true] [read acces by clent, read/write by Basestation ]
+ RTK/Base/(NAME of BASE)/Status/Operation - "ONLINE" full working RTCM, "STBY" receiving RTCM, but not transmitting over MQTT, "ERROR" having some trouble (not reveiving RTCM) [QOS = 1, retain = true, last_will = "OFFLINE"] [read acces by clent, read/write by Basestation ]

for each RTCM msg type:
+ RTK/Base/NAME of BASE/RTCM/<TYPE of MSG> [retain = false] [read acces by client, read/write by Basestation ]

# Basestation subscribe:
+ RTK/Base/(NAME of BASE)/Command/In/Transmitter 

# Clients publish:
+ NIL

# Clients subscribe:
+ RTK/Base/(NAME of Base)/Status/Operation - check operation of desired Base
+ RTK/Base/(NAME of Base)/RTCM/# - get all RTCM msg avaliable
or 
+ RTK/Base/(NAME of Base)/RTCM/(TYPE of MSG) - just for each desired msg type

# //MaintenanceClient subscribe:
+ NIL

# //MaintenanceClient publish:
+ RTK/Base/(NAME of BASE)/Command/In/Transmitter  - "ON" or "OFF" to activate/deactivate RTCM transmission [retain = false] 

# Hints:
+ (NAME of BASE)- e.g. XYZ01  
+ (TYPE of MSG) - e.g. 1074


# further plans:
+ add PW acces for Basestation (if required at all)
+ add a special client ID  (user "BaseMaintenance") for mantenance my basestations with serial com

+ Caster setup

in far future
+ RTK/User/Subscribe/(NAME of USER)/Position/
+ and construckt/calculate virtual basestation on user request for user position - give virtual name MNTP and publish accordingly
