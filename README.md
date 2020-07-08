# RVMP (RTCM-VIA-MQTT-PROTOCOL)

RVMP (RTCM-VIA-MQTT-PROTOCOL) V0.2 (first very basic approach, but some features for future are in preperation). (GNU GPL V3)

In use by RVMT (RTCM Via MQTT Transmitter - https://github.com/hagre/RVMT_RTCM-VIA-MQTT-TRANSMITTER) using the MQTT msg/protocol (as a secure and opensource alternative to NTRIP) to get RTK correction data from BaseStations to Broker and to Rover GPS units.

Caster is on https://github.com/hagre/RVMT_RTCM-VIA-MQTT-CASTER

Basicly, this is a try to implement something like a NTRIP caster with MQTT features. There are a lot of similarities to NTRIP (V1.0/V2.0), because it is a known and usefull protocol with most needed features for this task.

Attension: Topics are casesensitive!!

# Basic Workflow Description: v.0.2
+ Basestation(s) are checking if a caster exists/is implemented or a simple base-client relay configuration is used
+ Basestation(s) is/are publishing to MQTT broker (RTK/Base/...) and/or (RTK/MNTP/... - simple relay config) and deliver basic information (needed for Mountpointlist and ..)
+ Basestationen are continuosly evaluating the RTCM input and presenting the avaliable feature (incl update if somthing new,..)
+ Basestationen have to present relevant MNTPList msg if in use as simple relay

+ Caster (if implemented) publish basic infos on RTK/Caster/... , to be recogniced by other users
+ Caster (if implemented) is subscribing to broker and receiving all msg from connected basestations.
+ Only caters have permission to read RTK/Base/...  input
+ Caster is keeping track of avaliable basstations (all, old and active ones).
+ Caster is creating an Mountpointlist of possible active stations and publishing it to the broker under RTK/MNTPList/...
+ Caster can activate RTCM transmission of connected and active basstation if needed
+ Caster are tranforming/copy all the necessary RTCM datar from the RTK/Base/... to the RTK/MNTP/... topic (readeable for all RTK users)
+ If a basestation is lost the last will show "off", the caster will remove it from the active Mountpointlist

+ Clients can connect to the broker (user/password protected) and post the user id, this will request the caster to resend the Mountpointlist
+ Clients can select to subscribe to the required RTCM msg from the desired MNTP (mountpoint) or all of them. RTK/MNTP/<NAME of MNTP>/RTCM/# 
+ If the Client is leaving, the lastwill "byebye" gets activated.
+ Caster can switch off publishing of the Basestations publishing RTCM (maintaining in STBY) if the last client has closed the subscription.


# More Details on the used Topics:

# Basestation (MNTP) publish:
Welcome msg: (transmitted when a new connection is established, or a feature is updated, .. )
+ RTK/Base/(NAME of BASE)/Status/FirmwareVer/ - e.g "2.2" - Version of transmitter firmware [QOS = 1, retain = true] [read acces only by caster app, read/write by Basestation ]
+ RTK/Base/(NAME of BASE)/Status/ProtocolVer/ - e.g "0.1" - Version of protocol [QOS = 1, retain = true] [read acces only by caster app, read/write by Basestation ]
+ RTK/Base/(NAME of BASE)/Status/RTCM/(TYPE of MSG)/ - e.g "1", "0.5" or "30" - time in seconds between update (~frequenz) [QOS = 1, retain = true] [read acces only by caster app, read/write by Basestation ]
+ RTK/Base/(NAME of BASE)/Status/Operation/ - "ON" full working RTCM, "STBY" receiving RTCM, but not transmitting over MQTT, "ERROR" having some trouble, "OFF" - not avaliable  [QOS = 1, retain = true, last_will = "OFF"] [read acces only by caster app, read/write by Basestation ]
+ RTK/Base/(NAME of BASE)/Status/Error/ - type of error e.g "NO_RTCM_INPUT"  [QOS = 1, retain = false] [read acces only by caster app, read/write by Basestation ] //further work required 
+ //planning to implement: RTK/Base/(NAME of BASE)/Status/Position/Lat/ - LatPosition [QOS = 1, retain = true] [read acces only by caster app, read/write by Basestation ] //further work required 
+ //planning to implement: RTK/Base/(NAME of BASE)/Status/Position/Long/ - LongPosition [QOS = 1, retain = true] [read acces only by caster app, read/write by Basestation ] //further work required 
+ //planning to try: RTK/MNTP/(NAME of BASE)/Command/Out/PW/ - e.g "Password" [QOS = 1, retain = false] [read acces only by caster app, read/write by Basestation ] //further work required 
+ //planning to try: RTK/MNTP/(NAME of BASE)/Command/Out/USER/ - e.g "Username" (? username == <NAME of MNTP>) [QOS = 1, retain = false] [read acces only by caster app, read/write by Basestation ] //further work required

for each RTCM msg type:
+ RTK/Base/NAME of BASE/RTCM/<TYPE of MSG>/ [retain = false] [read acces only by caster app, read/write by Basestation ]

# Basestation (MNTP) subscribe:
 RTK/Caster/Status/Operation/
+ RTK/Base/(NAME of BASE)/Command/In/RTCM/ 
+ RTK/Base/(NAME of BASE)/Command/In/GPS/ 
+ RTK/Base/(NAME of BASE)/Command/In/Transmitter/ 
+ //planning to try: RTK/Base/(NAME of BASE)Command/In/Serial/
+ //planning to try: RTK/Base/(NAME of BASE)/Serial/In/ 

# Caster subscribe:
+ RTK/Base/+/Status/Firmware/ 
+ RTK/Base/+/Status/Protocol/ 
+ RTK/Base/+/Status/RTCM/(TYPE of MSG)/ 
+ RTK/Base/+/Status/Operation/ 
+ RTK/Base/+/Status/Error/
+ RTK/Base/+/Status/Position/Lat/
+ RTK/Base/+/Status/Status/Position/Long/
+ //planning to try: RTK/MNTP/+/Command/Out/PW/
+ //planning to try: RTK/MNTP/+/Command/Out/USER/
+ RTK/Base/+/RTCM/#

some calculation in the caster ....  
- acvtivate or deactivate Basestations
- solve errors witch reboot if required
- copy RTCM from RTK/Base/(NAME of BASE)/... to RTK/MNTP/(NAME of MNTP)
- keep track of the Basestations
- construct and update Mountpointlist (all possible avaliable Basestations )
- keep track of the Basestations

# Caster publish:
+ RTK/Caster/Status/Operation/ - "ON" full working "OFF" - not avaliable  [QOS = 1, retain = true, last_will = "OFF"] [read acces by all RTK users, read/write by caster]
+ RTK/MNTP/(NAME of MNTP)/RTCM/(TYPE of MSG)/ [retain = false] [read acces by all RTK users, read/write by caster]
commands to Basestation
+ RTK/MNTP/(NAME of MNTP)/Command/In/RTCM/ - "ON" or "OFF" to activate/deactivate RTCM transmission if required [QOS = 1, retain = false] [read/write acces by caster app, read by correct Basestation ]
+ RTK/MNTP/(NAME of MNTP)/Command/In/GPS/ - "REBOOT" to force an GPS reboot [retain = false] [QOS = 1, read/write acces by caster app, read by correct Basestation ]
+ RTK/MNTP/(NAME of MNTP)/Command/In/Transmitter/ - "REBOOT" to force an transmitter reboot (e.g ESP32 modul) [QOS = 1, retain = false] [read/write acces by caster app, read by correct Basestation ]
+ RTK/MNTPList/(NAME of BASE)/Status/Protocol/ [QOS = 1, retain = true] [read/write acces by caster app, read by clients ]
+ RTK/MNTPList/(NAME of BASE)/Status/RTCM/(TYPE of MSG)/  [QOS = 1, retain = true] [read/write acces by caster app, read by clients ]
+ RTK/MNTPList/(NAME of BASE)/Status/Operation/  [QOS = 1, retain = true] [read/write acces by caster app, read by clients ]
+ RTK/MNTPList/(NAME of BASE)/Status/Position/Lat/ [QOS = 1, retain = true] [read/write acces by caster app, read by clients ]
+ RTK/MNTPList/(NAME of BASE)/Status/Position/Long/ [QOS = 1, retain = true] [read/write acces by caster app, read by clients ]

# Clients publish:
+ RTK/User/Subscribe/(NAME of USER)/ - "(NAME of MNTP)" to request MNTP transmission, caster will provide needet data if on the mountpointlist [QOS = 1, retain = false, last_will = "ByeBye"] [read/write acces by clients app, read by caster ]

# Clients subscribe:
initially and refresh if needed
+ RTK/MNTPList/# - get complete list
or 
+ RTK/MNTPList/(NAME of MNTP)/# - get single desired MNTP
continously 
+ RTK/MNTP/(NAME of MNTP)/Status/Operation/ - check operation of desired MNTP
+ RTK/MNTP/(NAME of MNTP)/RTCM/# - get all RTCM msg avaliable
or 
+ RTK/MNTP/(NAME of MNTP)/RTCM/(TYPE of MSG)/ - just for each desired msg type

# //MaintenanceClient subscribe:
+ //planning: RTK/Base/(NAME of BASE)/Serial/Out/

# //MaintenanceClient publish:
+ //planning: RTK/Base/(NAME of BASE)/Serial/In/ -  Serial transmission to GPS if required (config of chip) [retain = false] //serial transmission will be included in a later version
+ //planning to try: RTK/Base/(NAME of BASE)/Command/In/Serial/ - "ON" or "OFF" to activate/deactivate Serial transmission to GPS if required (config of chip) [retain = false] //serial transmission will be included in a later version


# Hints:
+ MNTP - Mountpoint
+ (NAME of MNTP)  == "(NAME of BASE)"- e.g. XYZ01  
+ (TYPE of MSG) - e.g. 1074


# further plans:
+add PW acces for Basestation (if required at all)
+add a special client ID  (user "BaseMaintenance") for mantenance my basestations with serial com
+serial com with base (RTK/Base/(NAME of BASE)/Command/In/Serial/ ...)
in far future
+ RTK/User/Subscribe/(NAME of USER)/Position/
+ and construckt/calculate virtual basestation on user request for user position - give virtual name MNTP and publish accordingly
