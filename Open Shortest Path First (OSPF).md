# Open Shortest Path First (OSPF)
**OSPF TABLES**
- Neighbor Table
- Topology Table
  - Link State Database (LSDB)
  - Link State Advertisemet (LSA)
- Routing Table

**OSPF PACKETS**
> *Hello*
>
> *Database Descriptor(DBD)*
>
> *Link State Request(LSR)*
>
> *Link State UPdate(LSU)*
>
> *LsACK*

**Neighbor Table**
- Directly connected OSPF routers
- state of Adjacency( Full,two way,DR)
- *show ip ospf neighbor*

**Topoly Table**
- Everything OSPF Know About
- Link State Database(LSDB)
  - Each entry in LSDB is known as: link state advertisement(LSA)
  - *show ip ospf database*

**Routing Tables**
- Routers Routing Table (not Solely a function of OSPF)
- OSPF will contribute its best routes to routing tables.

## OSPF PACKETS
**Hello** 
  - Perodically sent to **224.0.0.5** (Multicast address for all Ospf routers)
  - Discover other OSPF routers.
  - Includes info about sending router
  - Determines whether adjacency will form.

**Database Descriptor(DBD)**
  - Summary of LSA's in each routers LSDB.
  - Avoids sending full LSDB for each Neighbor.

**Link State Request(LSR)**
  - Sent to request full LSA's.

**Link State Update(LSU)**
  - Includes request LSA's
  - Also Sent For newly learned Networks.

**Link State Acknowledgement (LsACK)**
  - Sent to confirm reception of LSU.


## OSPF AREAS
- OSPF routers maintain identical LSDBs ( Changes anywhere propagated everywhere)
- Networks can be Segregated using Areas.
    - limits propagation to confined sections.
- Area Design Create a 0-tier hierarchy
    - Area 0: Top of Hierarchy , **BACKBONE AREA**
    - Area# : All other Areas: **1 to 4,294,967,295**
- Traffic between areas must traverse **Area 0**
    - Assures loop free area topologies
    - Hub and Spoke Desing.
 
      
## OSPF Types Of Routers
- Internal Routers
  - All interfaces in Single Area.

- Backbone Routers.
    - At least one interface in Area 0
  
- ABR-Area Border Routers.
    - Interfcaes in Area 0 and another Area.
    - Maintain a LSDB for each Area.
    - Summarize LSA's between Area.

- ASBR:Autonomus System Border Routers.
    -Redistributing foregin routes into OSPF.

|---------------------------------------------------------------------------------------------------------------------------------------------------------|

## HELLO PACKETS:
- Discover OSPF neighbors.
- Sent periodically to 224.0.0.5
  - Typically every 10 Seconds.
- Some Network don't support multicast
  - Router's peer ip must be maunually configured.
  - Hello packets sent unicat (tyically every 30 seconds)
- Content of hello determines if router will become neighbor

| Hello Packtes    | Descriptio |
|-------------------|------------|
| **Router ID**     | Identity of each routers (32bit)|
| **Hello Interval**| Frequency of periodic hello's |
| **Dead INterval** | Duration to remember Neighbor (Typically 4*Hello interval) |rea
| **Neighbor**      | Neighbor router's ID's seen on link. *validates two way reachability* |
| **Area ID**       | OSPF area interface belongs to  |
| **Authentication data** | Password restricated peering** |
| **Network mask**   |  Subnet mask for link |
| **Area Type**    | Normal,stub,NSSA  |
| **DR**            | IP address of DR |
| **BDR**          | IP address of BDR |
| **Priority**    | Influences DR/BDR election |



## AREA TYPE
- Normal Area : Default Area Type
- STUB AREA :
    - No redistributed routes
    - Replaced with default routes.
- NSSA: Not So Stubby Area
    - no redistrbution , except from local area.
    - optionall replaced with default routes.

## OSPF ADJACENCY

```mermaid
graph LR;
    DOWN --> INIT;
    INIT --> TwoWAY;
    TwoWAY -->EXSTART;
    EXSTART --> EXCHANGE;
    EXCHANGE --> LOADING;
    LOADING --> FULL;
    DOWN --> ATTEMPT;
    ATTEMPT --> INIT;
```

- DOWN
  - Initial state when ospf first configured
      - Technically , a non-state
  -Sending perodic Hellos to 224.0.0.5
      - Initally *Neighbor* field is *Empty*.

- INIT:
    - Received a hello packets.
    - Outbound hellos now include peer router ID.
 
- 2-Way:
    - Router see itself in Neighbor hello.
    - Router decide if adjacency will proceed.
 
- ExSTART:
    - Master/slave election.
    - Governs reliable DBD exchange.
    - Higher Router-id become master.
 
- ExChange:
    - Master/slave election is complete.
    - Slaves sends confirming DBD.
    - Peers exchange LSDB Summaries.
 
- Loading:
    - peer Know LSA's in Neighbor LSDB.
    - peer begin requesting full LSA's
    - LSR,LSU,LsACK.
 
  -FULL:
    - LSDB'S are syncronized.
    - Adjacency Complete.


## DR and BDR ELECTION:
- Elected using interface priority number.
    - Range [0-255]
    - Default 1
    - Highest router-id breaks ties.
 
```mermaid
graph TD;
  R1 --> | ip:10.10.10.1, 1.1.1.1, priority 1 | SW;
  R2 --> | ip:10.10.10.2, 2.2.2.2, priority 1 | SW;
  R3 --> | ip:10.10.10.3, 3.3.3.3, priority 1 | SW;
  R4 --> | ip:10.10.10.4, 4.4.4.4, priority 1 | SW;
```
- R1 initial hello packets.
    - Router id: 1.1.1.1
    - priority: 1
    - DR ip: 0.0.0.0
    - BDR ip : 0.0.0.0
- Checking if DR already exists.
- Duration: wait timer (same as dead interval 40sec)
- if no other DR found , R1 elects itself DR.
    - router id 1.1.1.1
    - priority 1
    - DR: 10.10.10.1
    - BDR: 0.0.0.0
 
-R2 Initiat Hello packets
  - Router ID: 2.2.2.2 
  - Priority : 1
  - DR: 0.0.0.0
  - BDR: 0.0.0.0
- Checking if DR already exists.
- Receives Hello packet from R1.
  - R2 has same priority , but better router id.
  - Does not preemot current DR.
- R2 Elects itself BDR.
  - router id: 2.2.2.2
  - priority: 1
  - DR: 10.10.10.1
  - BDR: 10.10.10.2
 
- R1 hello packets.
    - router id: 1.1.1.1
    - priority: 1
    - DR: 10.10.10.1
    - BDR: 10.10.10.2
 
- R3/R4 see DR/BDR already elected. Become something other than DR i.e DROTHER.
- Each router syncs LSDB with DR/BDR.
- DR/BDR requires full adjacency.
- No LSDB sync with other DROTHERS.
    - ***DR/BDR status and priority is per interfaces , not peer router***
- ***R1 is DR because it was enabled first***
- ***Priority can be influence for DR election***
    - ***0-255-default:1-Higher is better***
    - ***Ties broken by router id***
    - ***0 never become DR/BDR, interface is always DROTHER***
- ***Command: ip ospf priority [0-255]***

- How are routing updates propagated?
    - 224.0.0.5- All OSPF routers.
    - 224.0.0.6- Only DR/BDR
 
- When DR has routing updates:
    - DR sends LSU to 224.0.0.5 ( to all routers)
    - BDR send LsACK 224.0.0.5 ( to all routers)
    - DROTHER's send LsACK to 224.0.0.6 ( only to DR/BDR)
      
- When BDR has routing updates:
    - BDR sends LSU to 224.0.0.5
    - DR sends LsACK to 224.0.0.5
    - DROTHER's sends LsACk to 224.0.0.6
 
- When DROTHER's has routing updates:
    - DROTHERS sends LSU to 224.0.0.6
    - DR sends LSU to 224.0.0.5
    - BDR send LsACK to 224.0.0.5
    - Remaining DROTHERs send LsACk to 224.0.0.6.


 ## OSPF COST CALCULATION:

**COST= Refernce Bandwith / Interface Bandwidth  (Default: 100MPBS)**

## OSPF LSA TYPE:
![OSPF LSAs](https://github.com/Rajeev-Tamang/OSPF/blob/main/OSPF%20LSAs.jpg)
  ## OSPF LSA TYPE

**Router LSA**
  - Router identifies itself and its link and each link costs
  - Ip networks/subnets masks / cost for each router link
  - used to build  topology map of local area
  - 
    ***NOTE: Type 1 i.e Router LSA vanhe ko within local Area hunxa , to verify we can see the LSDB with show ip ospf database*** 


  **Network LSA TYPE 2 LSA**
    - send by DR, when multiple routers connected to the same multi-access link
    - DR interface ip, DR router id, network mask and router-id of all attached routers.
    
  ***NOTE: type 2 LSA hami teti belha dekxam jun belha euta area ma multi-acess link use hunxa jastai switch haru use vako thau ma jun thau ma DR/BR election hunxa***


**Summary LSA - TYPE 3 LSA**
  - Contain IP networks from foregin areas.
  - sent by ABR
  - summarizes type 1 , type 2 and type 3 LSA's.
  - Each type 3 LSA includes one ip subnets.Type 3 LSA's can create ip summarization booundries.
  - **ABR generates a type 3 LSAs for each ip network in a foregin Area.**
  - **ABR generate type 3 LSAs ineach direction, for each Area they border**
  - **ABRs Generate type 3 LSAs from oter type 3 LSAs from foregin Areas.**
 
    ***NOTE:type 3 LSA vanna le euta area le aafno type 1,2 and 3 LSA lai summarize garhera aarko area lai dinxa , then hami LSDB ma type 3 LSA dekxam and yo kam ABR le Garxa***


**TYPE 5 and TYPE 4 LSAs [if we understand type 5 LSA then it will be easy to understand type 4 LSA]**
  - Contain an IP subnet redistributed into OSPF.
  - Sent by ASBR.
  - Forwarded unchanged throughout OSPF domain.


   **TYPE 4-ASBR SUMMARY LSA**
   - Include instruction to reach an ASBR in a foregin area-Generated by ABR in all areas, except area ASBR resides within.

  ***NOTE:OSPF ma euta router taba ASBR hunxa jaba tesma ospf route bahek aarhu jasti rip,isis,connected,etc route haru redistribute hunxa , jun area ma ASBR hunxa tiya hami type
  5 LSA dekxam and type type 5 LSA ABR router le aarhu area ma forward gardida without changes jasto ko testai forward gardinxa and so aarhu area lai how to reach that route vanne 
  tha hunna so ABR le Type 4 LSA ni send garxa aarhu area ma with Type 5 LSA and type 4 LSA le as a helper LSA kam garxa***

    
## OSPF NETWORK TYPE
  **POINT TO POINT**
```mermaid
graph LR;
    R1 --> |PointToPOint|R2;
```
  **BROADCAST and NBMA**
```mermaid
graph TD;
    R1["R1"] --> SW;
    R2 --> SW;
    R3 --> SW;
    R4 --> SW;
```

  **POINT TO MULTIPOINT AND P2MP-NB**
```mermaid
graph TD;
    R1["R1 (Hub)"] --> Cloud["☁ Cloud ☁"]
    Cloud --> R2["R2"]
    Cloud --> R3["R3"]    
```

|  Network Type  | P2P  | BROADCAST  | NBMA |  P2MP  | P2MP-NB  |
|----------------|------|-----------|-----|-------|-------------|
| **Max router per link** | 2  | &infin; | &infin; | &infin; | &infin; |
| **Full mesh connectivity** | ✅ | ✅ | ✅ | ❌ | ❌ |
| **DR/BDR** | ❌  | ✅  | ✅  |  ❌  | ❌  |
| **Hello/Dead interval timer** |  10/40  | 10/40  | 30/120  | 30/120  | 30/120  |
| **Auto Neighbor Discovery** | ✅  | ✅  |  ❌  | ✅  | ❌  |
| **Periodic Hello sent to** |  224.0.0.5 | 224.0.0.5 | Neighbor ID | 224.0.0.5 | Neighbor ID |
| **Neighbor Communication sent to** | 224.0.0.5 | unicast | unicast | unicast | unicast | 
| **LSA sent to** | 224.0.0.5 | multicast DR/BDR | unicast DR/BDR | unicast | unicast |
| **Next HOP IP** | Next peer | originator | originator | HUB | HUB |


## OSPF AREA TYPES

![OSPF AREA TYPE](https://github.com/Rajeev-Tamang/OSPF/blob/main/OSPF%20AREA%20TYPE.jpg)

***Befor understanding the OSPF Area, you should better understand the LSA TYPES.***

**1.NORMAL AREA**
    - LSA |1|2|3|4|5|
    - It is default area where all type of LSA are allowed.

**2.STUB ARE**
  - LSA |1|2|3|
  - This is a area where only type 1 , 2 and 3 are allowed. ALSO default route via type 3 appers.
  - ***NOTE-jun area ma ASBR hudaina tiyo area lai matra hamri stub area banhauna sakxam***
  - ***to implement ABR and tiyo area vari we need to add command [area 44 stub] 44 is area number, change according to your topology***

***3.TOTALLY STUB AREA**
- LSA |1|2|
- This is area where type 1 and 2 are only allowed , it is an additioal features of stub area, and changes need to be done in ABR only , not on all router within an area.
- **ABR(config): area 44 no-summary**
- ***NOTE:hamle area lai stub banhaune bitikai euta default route aauxa so hamlai type 3 LSA ni hamro routing table within that area khashai vannu parda xaidaina as sabbai ABR batai janhe ho , so tesko laghi hami totally stubby area  useful hunxa***


**4.NSSA**
- It only exists in ASBR area, it is similar like STUB area where only 1,2,3 are allowed but in NSSA area there is TYPE 7 LSA.
- Redistribution route via Type 7 .
- ABR convert the type 7 to TYPE 5 when advertising to other area.
- **COMMAND: area 55 nssa default-information-originate**
- ***NOTE:For better understaing figure ma vako explanaton paddhnu***

**5.TOTALLY NSSA**
  - LSAs |1|2|7|
  - Redistribution via type 7
  - default route via type 3.

**Opaque LSA's**
  - Carry non-ospf information via OSPF.
  - LSA TYPE 9 -local link scope
  - LSA TYPE 10- Local area scope
  - LSA TYPE 11- OSPF Domain SCOPE.
  - BY default Opaque LSA Are allowed in STUB and NSSA.
  - **COMMAND: stub no-ext-capabilty- disable opaque LSA
