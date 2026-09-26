Cisco-Packet-Tracer-Standard-ACL-Lab
A hands-on network security lab configuring and troubleshooting **Standard Access Control Lists (ACLs)** in Cisco IOS.

---

## 📌 Topology Overview

* **LAN Subnet:** `10.0.0.0/8`
  * **PC0:** `10.0.0.2` (Target to be isolated)
  * **PC1:** `10.0.0.3` (Legitimate user / Permitted)
  * **Default Gateway:** `10.0.0.1` (Router `Gig0/0/0`)
* **DMZ / Server Subnet:** `20.0.0.0/8`
  * **Web/DNS Server:** `20.0.0.2`
  * **Gateway:** `20.0.0.1` (Router `Gig0/0/1`)

---

## 🎯 Lab Objectives

1. Block **PC0 (`10.0.0.2`)** from reaching the Server subnet (`20.0.0.0/8`).
2. Allow **PC1 (`10.0.0.3`)** and all other endpoints to communicate freely.
3. Understand the danger of the **Implicit Deny** rule.
4. Correctly apply the ACL using the proper interface direction (`in` vs `out`).

---

## ⚙️ Router Configuration
```cisco
Router> enable
Router# configure terminal

! --- Step 1: Create Standard ACL Number 1 ---
! Block the specific host
Router(config)# access-list 1 deny host 10.0.0.2

! Prevent total blackout by permitting all other traffic
! (Overrides the invisible implicit deny any)
Router(config)# access-list 1 permit any

! --- Step 2: Apply ACL to Interface ---
! Option A: Inbound on LAN Gateway (Recommended for Standard ACL)
Router(config)# interface gigabitEthernet 0/0/0
Router(config-if)# ip access-group 1 in
Router(config-if)# exit

! Alternative Option B: Outbound on Server-facing interface
! Router(config)# interface gigabitEthernet 0/0/1
! Router(config-if)# ip access-group 1 out

🧠 Key Takeaways & The Brutal Truths
1. The Invisible Trap: Implicit Deny

Every Cisco ACL ends with an unwritten deny any rule.

    If you write only access-list 1 deny host 10.0.0.2, the router drops 10.0.0.2 on line 1, and drops everyone else on the invisible line 2.
    Adding access-list 1 permit any ensures only the targeted host is blocked while others remain unaffected.

2. Standard vs Extended ACL Placement Rule

    Standard ACL (1–99): Filters only by Source IP. Placement rule: apply as close to the destination as possible (or inbound on host interface if isolating from entire router transit).
    Extended ACL (100–199): Filters by source, destination, protocol, and port. Placement rule: apply as close to the source as possible.

3. Directionality (in vs out)

    in: Packet enters the router interface before routing lookup.
    out: Packet has already been routed and is exiting through the interface wire.

🧪 Verification & Proof
From PC0 (10.0.0.2):

        

cmd

PC> ping 20.0.0.2
Pinging 20.0.0.2 with 32 bytes of data:
Request timed out.
Request timed out.

(Result: Traffic Dropped as intended)
From PC1 (10.0.0.3):

        

cmd

PC> ping 20.0.0.2
Pinging 20.0.0.2 with 32 bytes of data:
Reply from 20.0.0.2: bytes=32 time<1ms TTL=127
Reply from 20.0.0.2: bytes=32 time<1ms TTL=127

(Result: Success / Reachable)
From Router CLI (Packet Kill-Count):

        

cisco
Router# show access-lists 1
Standard IP access list 1
10 deny 10.0.0.2 (4 matches)
20 permit any (4 matches)

👨‍💻 Author

EHSAN SALEHI 🚀
