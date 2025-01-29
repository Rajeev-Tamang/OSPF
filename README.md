# OSPF
**OSPF TABLES**
>
> Neighbor Table
>
> Topology Table
- Link State Database (LSDB)
- Link State Advertisemet (LSA)
>
> Routing Table

**OSPF PACKETS**
> Hello
>
> Database Descriptor(DBD)
>
> Link State Request(LSR)
>
> Link State UPdate(LSU)
>
> LsACK

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
**Hello** **.**
