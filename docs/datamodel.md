আগের চ্যাপ্টারে আমরা Nautobot 3.0 ইনস্টল করলাম। নির্ভর কমিউনিকেশনের এনওসি টিম এখন একটা চালু নটোবট ইনস্ট্যান্স পেয়েছে। কিন্তু এখনও কোনো ডেটা নেই - খালি একটা সিস্টেম। ডেটা এন্ট্রি শুরু করার আগে বুঝতে হবে নটোবট কীভাবে নেটওয়ার্ক ডেটা স্টোর করে, কীভাবে বিভিন্ন জিনিস একে অপরের সাথে সম্পর্কিত।

এই চ্যাপ্টারে আমরা Nautobot 3.0 এর ডেটা মডেল বুঝব। এটা একটু থিওরি মনে হতে পারে, কিন্তু এই ফাউন্ডেশন না বুঝলে পরে ডেটা এন্ট্রি করতে গিয়ে সমস্যা হবে। নির্ভর কমিউনিকেশনের নেটওয়ার্ককে উদাহরণ হিসেবে ব্যবহার করে প্রতিটা কনসেপ্ট ব্যাখ্যা করব।

### Nautobot 3.0 এর নতুন আর্কিটেকচার

Nautobot 3.0 এ সবচেয়ে বড় পরিবর্তন হলো এর মডুলার আর্কিটেকচার। পুরো সিস্টেমকে কয়েকটা কোর অ্যাপ-এ ভাগ করা হয়েছে:

**কোর অ্যাপ্লিকেশন:**

- **Organization**: Tenants, Teams, Locations
- **DCIM** (Data Center Infrastructure Management): Devices, Racks, Cables
- **IPAM** (IP Address Management): IP Addresses, Prefixes, VLANs
- **Circuits**: Providers, Circuit Types, Circuits
- **Extras**: Roles, Status, Tags, Custom Fields

প্রতিটা অ্যাপ নিজের মতো কাজ করে, কিন্তু সবগুলো একসাথে মিলে একটা সম্পূর্ণ নেটওয়ার্ক সোর্স অফ ট্রুথ তৈরি করে।

চলুন একটা একটা করে দেখি।

---

### অর্গানাইজেশন অ্যাপ - নেটওয়ার্কের কাঠামো

অর্গানাইজেশন অ্যাপ হলো নটবটের ভিত্তি। এখানে আপনার নেটওয়ার্কের অর্গানাইজেশন অ্যাপ ডিফাইন করবেন।

#### লোকেশন - সবচেয়ে গুরুত্বপূর্ণ পরিবর্তন

**Nautobot 2.x এ ছিল:** সাইটস (একটা ফ্ল্যাট লিস্ট)

**Nautobot 3.0 এ এখন:** লোকেশন (হায়ারারকিকাল, নেস্টেবল)

নির্ভর কমিউনিকেশনের নেটওয়ার্ক ঢাকা নর্থ জোনে ছড়িয়ে আছে। তাদের এই স্ট্রাকচার:

```
Nirvor Communication Network
  └── Dhaka North Zone
       ├── Mirpur Cluster
       │    └── Mirpur POP
       │         ├── Rack A
       │         └── Rack B
       └── Uttara Cluster
            └── Uttara POP
                 └── Rack A
```

এই হায়ারারকি Nautobot 3.0 এ সহজেই রিপ্রেজেন্ট করা যায়।

**লোকেশন টাইপ:**

প্রথমে ডিফাইন করতে হয় কী ধরনের লোকেশন আছে:

```
Location Type: Zone
  - Description: Geographic coverage zone
  - Nestable: Yes
  - Parent: None

Location Type: Cluster
  - Description: Group of nearby POPs
  - Nestable: Yes
  - Parent: Zone

Location Type: POP
  - Description: Point of Presence
  - Nestable: Yes
  - Parent: Cluster

```

**অ্যাকচুয়াল লোকেশন:**

এরপর অ্যাকচুয়াল লোকেশন তৈরি করবেন:

```
Location: Dhaka North Zone
  - Type: Zone
  - Parent: None
  - Status: Active

Location: Mirpur Cluster
  - Type: Cluster
  - Parent: Dhaka North Zone
  - Status: Active

Location: Mirpur POP
  - Type: POP
  - Parent: Mirpur Cluster
  - Status: Active
  - Address: House 25, Road 10, Mirpur-12, Dhaka-1216
  - GPS: 23.8103, 90.3654

Rack (DCIM Object):

Rack: Rack A
  - Location: Mirpur POP
  - Status: Active
  - Height: 42U
```

**লোকেশন হায়ারারকি এর সুবিধা:**

১. **ফিল্টারিং সহজ:** "মিরপুর ক্লাস্টার এর সব ডিভাইস দেখাও" - এক ক্লিকে
২. **রিপোর্টিং:** জোন অনুযায়ী রিপোর্ট তৈরি করা সহজ
৩. **স্কেলেবিলিটি:** নতুন জোন/ক্লাস্টার যোগ করা সহজ
৪. **ক্ল্যারিটি:** নেটওয়ার্ক স্ট্রাকচার একদম পরিষ্কার

#### টেন্যান্ট - মাল্টি-টেন্যান্সি

যদি আপনার একাধিক ব্যবসা বা ব্র্যান্ড থাকে, তাহলে টেন্যান্ট ব্যবহার করবেন।

নির্ভর কমিউনিকেশনের দুটো ব্র্যান্ড আছে:
- Nirvor Communication (Residential)
- Nirvor Communication (Corporate)

```
Tenant: Nirvor Communication Residential
  - Description: Residential broadband services
  - Group: Consumer Services

Tenant: Nirvor Communication Business
  - Description: Corporate connectivity solutions
  - Group: Enterprise Services
```

প্রতিটা ডিভাইস, আইপি অ্যাড্রেস, সার্কিট-এ টেন্যান্ট অ্যাসাইন করা যায়। তাহলে আলাদা আলাদা করে ট্র্যাক করতে পারবেন।

**ছোট আইএসপি এর জন্য:** টেন্যান্ট সাধারণত লাগে না। সব একটা সিঙ্গেল অর্গানাইজেশন এর আন্ডারে রাখলেই চলে।

#### টিমস - কে কী দেখবে

বড় অর্গানাইজেশন এ বিভিন্ন টিম থাকে। Nautobot 3.0 এ টিমস দিয়ে সেটা ম্যানেজ করা যায়।

নির্ভর কমিউনিকেশনের টিম:

```
Team: NOC Team
  - Members: Jahangir, Asif, Rahim
  - Permissions: সব দেখতে এবং এডিট করতে পারবে

Team: Field Operations
  - Members: Karim, Salman, Rafiq
  - Permissions: শুধু devices এবং cables দেখতে/এডিট করতে পারবে

Team: Management
  - Members: CEO, CTO
  - Permissions: শুধু read-only access
```

---

### ডিসিআইএম অ্যাপ - ফিজিক্যাল ইনফ্রাস্ট্রাকচার

ডিসিআইএম (ডাটা সেন্টার ইনফ্রাস্ট্রাকচার ম্যানেজমেন্ট) হলো নটবটের সবচেয়ে বড় অংশ। এখানে আপনার সব ফিজিক্যাল নেটওয়ার্ক ইকুইপমেন্ট থাকবে।

#### ডিভাইস টাইপ - ব্লুপ্রিন্ট

কোনো ডিভাইস যোগ করার আগে তার "type" define করতে হয়। ডিভাইস টাইপ হলো একটা ব্লুপ্রিন্ট বা টেমপ্লেট।

নির্ভর কমিউনিকেশন MikroTik router ব্যবহার করে। তাদের মেইন মডেল হলো CCR2116-12G-4S+।

```
Device Type: MikroTik CCR2116-12G-4S+
  - Manufacturer: MikroTik
  - Model: CCR2116-12G-4S+
  - Part Number: CCR2116-12G-4S+
  - Height: 1U
  - Full Depth: Yes
  - Subdevice Role: None (এটা parent device)
  
  Interface Templates:
    - ether1 (Type: 1000BASE-T)
    - sfp-sfpplus1 (Type: 10GBASE-X-SFP+)
    - sfp-sfpplus2 (Type: 10GBASE-X-SFP+)
    - ... sfp-sfpplus12 পর্যন্ত
    - sfp28-1 (Type: 25GBASE-X-SFP28)
    - sfp28-2 (Type: 25GBASE-X-SFP28)
```

**ইন্টারফেস টেমপ্লেট এর গুরুত্ব:**

যখন এই ডিভাইস টাইপ এর একটা নতুন ডিভাইস তৈরি করবেন, তখন সব ইন্টারফেসেস অটোমেটিক্যালি তৈরি হয়ে যাবে। ম্যানুয়ালি একটা একটা করে করতে হবে না।

#### ম্যানুফ্যাকচারার

ডিভাইস টাইপ তৈরির আগে ম্যানুফ্যাকচারার তৈরি করতে হয়:

```
Manufacturer: MikroTik
  - Description: Latvian network equipment manufacturer
  - Website: https://mikrotik.com

Manufacturer: TP-Link
  - Description: Chinese networking equipment
  - Website: https://www.tp-link.com

Manufacturer: Cisco
  - Description: American networking giant
  - Website: https://www.cisco.com
```

#### ডিভাইস - আসল হার্ডওয়্যার

এখন অ্যাকচুয়াল ডিভাইস তৈরি করা যাবে।

নির্ভর কমিউনিকেশনের মিরপুর পপ  এ একটা কোর রাউটার আছে:

```
Device: R-DN-MIR-CORE-01
  - Device Type: MikroTik CCR2116-12G-4S+
  - Role: Core Router (এটা Extras থেকে আসবে, পরে দেখব)
  - Location: Mirpur POP
  - Rack: Rack A
  - Position: 25U
  - Face: Front
  - Status: Active
  - Serial Number: ABC1234MIR001
  - Asset Tag: NIRVOR-RTR-001
  - Comments: Primary core router for Mirpur POP
```

**ডিভাইস নেমিং কনভেনশন:**

নির্ভর কমিউনিকেশনের কনভেনশন:
```
Format: [Type]-[Zone]-[Site]-[Role]-[Number]

R = Router
DN = Dhaka North
MIR = Mirpur
CORE = Core Router
01 = First device

উদাহরণ:
R-DN-MIR-CORE-01 = Router, Dhaka North, Mirpur, Core, #1
SW-DN-UTT-DIST-01 = Switch, Dhaka North, Uttara, Distribution, #1
```

#### ইন্টারফেস - কানেকশন পয়েন্ট

ডিভাইস তৈরি হলে তার ইন্টারফেসেস অটোমেটিক্যালি তৈরি হয়েছে (ডিভাইস টাইপ এর টেমপ্লেট থেকে)। কিন্তু এগুলোতে আরো ডিটেইলস যোগ করতে হয়।

মিরপুর কোর রাউটার এর আপলিংক ইন্টারফেস:

```
Interface: sfp-sfpplus1
  - Device: R-DN-MIR-CORE-01
  - Type: 10GBASE-X-SFP+
  - Enabled: Yes
  - MTU: 1500
  - Speed: 10 Gbps
  - Description: "Uplink to BTCL"
  - Mode: Access (Trunk নয়)
```

#### কেবল - ফিজিক্যাল কানেকশন

দুটো ডিভাইস একসাথে কানেক্ট করতে কেবল অবজেক্ট তৈরি করতে হয়।

মিরপুর কোর রাউটার থেকে ডিস্ট্রিবিউশন সুইচ এ একটা ফাইবার কেবল:

```
Cable: CORE-TO-DIST-01
  - Termination A:
    * Device: R-DN-MIR-CORE-01
    * Interface: sfp-sfpplus2
  
  - Termination Z:
    * Device: SW-DN-MIR-DIST-01
    * Interface: sfp1
  
  - Type: Single-Mode Fiber (SMF)
  - Status: Connected
  - Color: Blue
  - Length: 5 m
  - Label: CORE-TO-DIST-01
```

**কেবল ট্রেস ফিচার:**

কেবল তৈরি করার পর নটবটের "Trace" ফিচার দিয়ে পুরো ফিজিক্যাল পাথ ফলো করতে পারবেন। যেমন একটা কাস্টমার এর কানেকশন কোন কোন ডিভাইস দিয়ে গেছে - সব দেখা যায়।

#### র‍্যাক - সংগঠিত রাখা

র‍্যাক হলো একটা ফিজিক্যাল এনক্লোজার যেখানে ডিভাইস মাউন্ট করা থাকে।

```
Rack: Rack A
  - Location: Mirpur POP
  - Status: Active
  - Type: 4-post cabinet
  - Width: 19 inches
  - Height: 42U
  - Description: Primary equipment rack
  - Outer Width: 600 mm
  - Outer Depth: 1000 mm
```

**র‍্যাক এলিভেশন:**

নটবটে একটা ভিজ্যুয়াল র‍্যাক এলিভেশন দেখা যায়। কোন U পজিশন এ কোন ডিভাইস আছে - গ্রাফিক্যালি দেখায়।

```
42U ┌──────────────┐
    │              │  (Empty)
    │              │
25U ├──────────────┤
    │ R-DN-MIR-... │  MikroTik CCR2116 (1U)
24U ├──────────────┤
    │              │
20U ├──────────────┤
    │ SW-DN-MIR-...│  TP-Link Switch (1U)
19U ├──────────────┤
    ...
```

---

### আইপিএএম অ্যাপ - আইপি অ্যাড্রেস ম্যানেজমেন্ট

নেটওয়ার্কের লজিক্যাল দিক - আইপি অ্যাড্রেস, সাবনেটস, ভি-লেন।

#### আইপি নেমস্পেস - আইসোলেটেড আইপি স্পেস

Nautobot 3.0 এ নেমস্পেস একটা নতুন ফিচার। এটা দিয়ে একই আইপি রেঞ্জ বিভিন্ন কনটেক্সট এ ব্যবহার করা যায়।

**নির্ভর কমিউনিকেশনের ক্ষেত্রে:**

তাদের শুধু একটা নেমস্পেস দরকার:

```
Namespace: Global
  - Description: Primary IP namespace for Nirvor Communication
```

**কখন একাধিক নেমস্পেস লাগে?**

ধরুন আপনার দুটো সম্পূর্ণ আলাদা নেটওয়ার্ক আছে যেগুলো কখনোই ইন্টারকানেক্ট হবে না। তখন উভয়েই 10.0.0.0/8 ব্যবহার করতে পারবেন আলাদা নেমস্পেস এ রেখে।

#### প্রিফিক্স - আইপি রেঞ্জ

প্রিফিক্স হলো একটা আইপি সাবনেট।

নির্ভর কমিউনিকেশনের কাছে বিটিসিএল থেকে পাওয়া একটা /22 পাবলিক আইপি ব্লক আছে:

```
Prefix: 103.125.40.0/22
  - Type: Container (এর ভেতরে আরো ছোট prefixes আছে)
  - Status: Active
  - Namespace: Global
  - Description: Assigned public IP block from BTCL
  - Location: (খালি - কারণ এটা overall block)
```

**হায়ারারকিকাল প্রিফিক্স:**

এই /22 ব্লক কে ভাগ করা হয়েছে:

```
Parent: 103.125.40.0/22 (Container)
  ├── 103.125.40.0/24 (Network)
  │   - Location: Mirpur POP
  │   - VLAN: RESIDENTIAL_MIR
  │   - Description: Residential customers - Mirpur
  │
  ├── 103.125.41.0/24 (Network)
  │   - Location: Uttara POP
  │   - VLAN: RESIDENTIAL_UTT
  │   - Description: Residential customers - Uttara
  │
  ├── 103.125.42.0/25 (Network)
  │   - VLAN: CORPORATE
  │   - Description: Corporate clients
  │
  └── 103.125.42.128/25 (Network)
      - Description: Infrastructure IPs
```

**প্রিফিক্স টাইপ:**

- **কন্টেইনার:** শুধু অর্গানাইজ করার জন্য, এর ভেতরে চাইল্ড প্রিফিক্স আছে
- **নেটওয়ার্ক:** আসল ইউজেবল নেটওয়ার্ক, এখান থেকে আইপি অ্যালোকেট হয়
- **পুল:** ডাইনামিক অ্যালোকেশন এর জন্য (যেমন ডিএইচসিপি পুল)

#### আইপি অ্যাড্রেস - ইন্ডিভিজুয়াল আইপি

ইন্ডিভিজুয়াল আইপি অ্যাড্রেসেস অ্যাসাইন করা হয় ডিভাইস  এর ইন্টারফেস এ।

ম্যানেজমেন্ট আইপি স্পেস:

```
Prefix: 10.10.0.0/16
  - Type: Container
  - Status: Active
  - Namespace: Global
  - Description: Management IP space
```
আপলিংক অ্যান্ড ইনফ্রাস্ট্রাকচার আইপি:
```
Prefix: 103.125.42.128/25
  - Type: Container
  - Status: Active
  - Namespace: Global
  - Description: Uplink & Infrastructure IPs
```
Mirpur core router এর লুপব্যাক আইপি:
```
IP Address: 10.10.1.1/32
  - Status: Active
  - Role: Loopback
  - Namespace: Global
  - Parent Prefix: 10.10.0.0/16
  - Assigned to:
    * Device: R-DN-MIR-CORE-01
    * Interface: lo0
  - DNS Name: r-mir-core-01.nirvor.bd
  - Description: Management loopback
```

আপলিংক আইপি:

```
IP Address: 103.125.42.130/30
  - Status: Active
  - Role: Network
  - Namespace: Global
  - Parent Prefix: 103.125.42.128/25
  - Assigned to:
    * Device: R-DN-MIR-CORE-01
    * Interface: sfp-sfpplus1
  - DNS Name: r-mir-uplink.nirvor.bd
  - Description: BTCL uplink connection
```
#### সামারি

নেমস্পেস Nautobot 3.0 এর IPAM মডেলের একটি গুরুত্বপূর্ণ অংশ। এটি ওভারল্যাপিং আইপি ডিজাইন, মাল্টি-টেন্যান্ট আর্কিটেকচার এবং ভিআরএফ-বেসড নেটওয়ার্ক কে পরিষ্কারভাবে ম্যানেজ করতে সাহায্য করে।

#### ভি-ল্যান - লেয়ার ২ সেগমেন্টেশন

ভি-ল্যান দিয়ে একই ফিজিক্যাল নেটওয়ার্ক কে লজিক্যাল ভাগে আলাদা করা যায়।

নির্ভর কমিউনিকেশনের ভি-ল্যান স্ট্রাকচার:

```
VLAN 10: MGMT_MIRPUR
  - Location: Mirpur POP
  - Status: Active
  - Description: Management network for Mirpur devices

VLAN 20: UPLINK_MIRPUR
  - Location: Mirpur POP
  - Status: Active
  - Description: Uplink to BTCL

VLAN 100: RESIDENTIAL_MIR
  - Location: Mirpur POP
  - Status: Active
  - Description: Residential customer traffic - Mirpur

VLAN 200: CORPORATE
  - Location: Global
  - Status: Active
  - Description: Corporate clients across all sites
```

**ভি-ল্যান গ্রুপ:**

যদি একই ভি-ল্যান আইডি বিভিন্ন সাইটে ভিন্ন পারপাস এ ব্যবহার করতে চান, তাহলে ভি-ল্যান গ্রুপ ব্যবহার করুন।

---

### সার্কিটস অ্যাপ - আপস্ট্রিম কানেক্টিভিটি

যেকোনো আইএসপি এর নিজের আপলিংক প্রোভাইডার থাকে। নির্ভর কমিউনিকেশন বিটিসিএল থেকে ইন্টারনেট নেয়।

#### প্রোভাইডার

প্রথমে প্রোভাইডার ডিফাইন করুন:

```
Provider: BTCL
  - ASN: 17494
  - Account Number: NIRVOR-BTCL-2023-MIR
  - Portal URL: https://isp.btcl.gov.bd
  - NOC Contact: +880 2-9555555
  - Admin Contact: noc@btcl.gov.bd
  - Comments: Primary upstream provider
```

নির্ভর কমিউনিকেশনের একটা সেকেন্ডারি প্রোভাইডার ও আছে:

```
Provider: Summit Communications
  - ASN: 45928
  - Account Number: SKY-SUMMIT-2024
  - Portal URL: https://portal.summitcommunications.net
  - Comments: Secondary/backup provider
```

#### সার্কিট টাইপ

সার্কিট এর ধরন ডিফাইন করুন:

```
Circuit Type: Fiber Optic
  - Description: Fiber optic connectivity

Circuit Type: Microwave
  - Description: Point-to-point microwave link

Circuit Type: IPLC
  - Description: International Private Leased Circuit
```

#### সার্কিট - অ্যাকচুয়াল কানেকশন

এখন অ্যাকচুয়াল সার্কিট:

```
Circuit: BTCL-MIR-001
  - Provider: BTCL
  - Type: Fiber Optic
  - Status: Active
  - Install Date: 2023-03-15
  - Commit Rate: 5000 Mbps (5 Gbps)
  - Description: Primary 5Gbps uplink from Mirpur POP
  - Comments:
    * Contract Start: 2023-03-01
    * Contract Duration: 2 years
    * Monthly Cost: BDT 400,000
    * Renewal Date: 2025-03-01
```

**সার্কিট টার্মিনেশন:**

প্রতিটা সার্কিট এর দুই প্রান্ত থাকে - A এবং Z।

```
Termination A (Provider Side):
  - Side: A
  - Port Speed: 5 Gbps
  - Upstream Speed: 5000 Mbps
  - Cross Connect ID: BTCL-XC-MIR-12345

Termination Z (Nirvor Communication Side):
  - Side: Z
  - Location: Mirpur POP
  - Device: R-DN-MIR-CORE-01
  - Interface: sfp-sfpplus1
  - Port Speed: 5 Gbps
```

এভাবে নটোবট জানে ঠিক কোন ইন্টারফেস এ আপলিংক আছে।

---

### এক্সট্রাস অ্যাপ - ফ্লেক্সিবিলিটি এবং কাস্টমাইজেশন

এক্সট্রাস অ্যাপ হলো নটবটের "toolbox"। এখানে এমন জিনিস আছে যা সব অ্যাপ এ ব্যবহার হয়।

#### রোলস - ইউনিফাইড সিস্টেম (Nautobot 3.0 এর বড় পরিবর্তন)

**Nautobot 2.x এ ছিল:**
- Device Roles (আলাদা)
- Rack Roles (আলাদা)
- IPAM Roles (আলাদা)

**Nautobot 3.0 এ এখন:**
- একটা unified Roles system
- Content Types দিয়ে নির্ধারণ করবেন কোথায় ব্যবহার হবে

নির্ভর কমিউনিকেশন এর ডিভাইস রোল:

```
Role: Core Router
  - Content Types: dcim.device
  - Color: f44336 (Red)
  - Weight: 1000 (যত বেশি, তত গুরুত্বপূর্ণ)
  - Description: Core routing equipment

Role: Distribution Switch
  - Content Types: dcim.device
  - Color: ff9800 (Orange)
  - Weight: 800
  - Description: Distribution layer switches

Role: Access Switch
  - Content Types: dcim.device
  - Color: 4caf50 (Green)
  - Weight: 600
  - Description: Access layer switches
```

**একই রোল একাধিক জায়গায়:**

```
Role: Critical Infrastructure
  - Content Types: dcim.device, dcim.rack, circuits.circuit
  - Color: 000000 (Black)
  - Weight: 2000
  - Description: Mission-critical components
```

এই রোল এখন ডিভাইস, র‍্যাক, এবং সার্কিট তিনটাতেই ব্যবহার করা যাবে।

#### স্ট্যাটাস - লাইফসাইকেল ম্যানেজমেন্ট

প্রতিটা অবজেক্ট এর একটা স্ট্যাটাস থাকে। Nautobot 3.0 এ কাস্টম স্ট্যাটাস তৈরি করা যায়।

**ডিফল্ট স্ট্যাটাস:**
- Active
- Planned
- Staged
- Failed
- Inventory
- Decommissioned
- Offline
- Retired

নির্ভর কমিউনিকেশন একটা কাস্টম স্ট্যাটাস যোগ করেছে:

```
Status: Under Maintenance
  - Content Types: dcim.device
  - Color: ffeb3b (Yellow)
  - Description: Device temporarily offline for maintenance
```

#### ট্যাগ - ক্যাটেগরাইজেশন

ট্যাগ দিয়ে যেকোনো অবজেক্ট ক্যাটেগরাইজ করা যায়।

নির্ভর কমিউনিকেশনের কিছু ট্যাগ:

```
Tag: production
  - Color: 4caf50 (Green)
  - Description: Production/live equipment

Tag: backup
  - Color: 2196f3 (Blue)
  - Description: Backup/standby equipment

Tag: end-of-life
  - Color: f44336 (Red)
  - Description: Equipment approaching end of life

Tag: critical
  - Color: 000000 (Black)
  - Description: Critical infrastructure
```

**ট্যাগ এর ব্যবহার:**

```
Device: R-DN-MIR-CORE-01
  - Tags: production, critical
  
Device: R-DN-MIR-CORE-02
  - Tags: backup
```

এখন filter করতে পারবেন: "Show me all devices with 'critical' tag"

#### কাস্টম ফিল্ড - নিজের মতো ফিল্ড

নটবটে যদি কোনো বিল্ট-ইন ফিল্ড না থাকে, কাস্টম ফিল্ড তৈরি করুন।

নির্ভর কমিউনিকেশনের দরকার ওয়ারেন্টি ট্র্যাকিং:

```
Custom Field: Warranty Expiry Date
  - Content Types: dcim.device
  - Type: Date
  - Label: Warranty Expiry Date
  - Description: Device warranty expiration date
  - Required: No
  - Default: (empty)
```

এখন প্রতিটা ডিভাইস এ এই ফিল্ড দেখাবে:

```
Device: R-DN-MIR-CORE-01
  - Warranty Expiry Date: 2027-01-15
```

আরেকটা উদাহরণ - বিল্ডিং ট্র্যাকিং:

```
Custom Field: Building Name
  - Content Types: dcim.device
  - Type: Text
  - Label: Building Name
  - Description: Building where device is located
```

```
Device: SW-DN-MIR-ACC-01
  - Building Name: Building A, Sector 12
```

#### কাস্টম রিলেশনশিপ (Nautobot 3.0 নতুন)

এটা একটা শক্তিশালী নতুন ফিচার। দুটো যেকোনো অবজেক্ট এর মধ্যে কাস্টম রিলেশনশিপ তৈরি করা যায়।

**উদাহরণ:** ডিভাইস এবং কাস্টমার এর মধ্যে রিলেশনশিপ

```
Relationship: Device Serves Customer
  - Source Type: dcim.device
  - Destination Type: tenancy.tenant
  - Type: Many to Many
  - Description: Which devices serve which customers
```

এখন একটা ডিভাইস এর সাথে মাল্টিপল কাস্টমার লিংক করা যাবে:

```
Device: SW-DN-MIR-ACC-01
  - Serves Customers:
    * ABC Corporation
    * XYZ Limited
    * DEF Enterprises
```

---

### ডেটা মডেলের রিলেশনশিপ - সব কীভাবে যুক্ত

এখন দেখি কীভাবে সব একসাথে কাজ করে। নির্ভর কমিউনিকেশনের Mirpur POP এর একটা সম্পূর্ণ উদাহরণ:

#### উদাহরণ: মিরপুর পপ এর সম্পূর্ণ ডেটা স্ট্রাকচার

**1. লোকেশন হায়ারারকি:**

```
Dhaka North Zone (Location Type: Zone)
  └── Mirpur Cluster (Location Type: Cluster)
       └── Mirpur POP (Location Type: POP)
            ├── Rack A (Location Type: Rack)
            └── Rack B (Location Type: Rack)
```

**2. মিরপুর পপ-তে ডিভাইস:**

```
R-DN-MIR-CORE-01
  - Type: MikroTik CCR2116
  - Role: Core Router
  - Location: Mirpur POP → Rack A → Position 25U
  - Status: Active
  - Tags: production, critical
  - Custom Fields:
    * Warranty Expiry: 2027-01-15
  
SW-DN-MIR-DIST-01
  - Type: TP-Link TL-SX3016F
  - Role: Distribution Switch
  - Location: Mirpur POP → Rack A → Position 20U
  - Status: Active
  - Tags: production

SW-DN-MIR-ACC-01
  - Type: TP-Link TL-SG3428MP
  - Role: Access Switch
  - Location: Mirpur POP → Rack B → Position 15U
  - Status: Active
  - Tags: production
  - Custom Fields:
    * Building Name: Building A, Sector 12
```

**3. কানেক্টিং কেবল:**

```
Cable: CORE-TO-DIST-01
  - From: R-DN-MIR-CORE-01 → sfp-sfpplus2
  - To: SW-DN-MIR-DIST-01 → sfp1
  - Type: SMF
  - Length: 5 m

Cable: DIST-TO-ACC-01
  - From: SW-DN-MIR-DIST-01 → ether1
  - To: SW-DN-MIR-ACC-01 → ether24
  - Type: Cat6
  - Length: 20 m
```

**4. আইপি অ্যাড্রেস:**

```
10.10.1.1/32
  - Assigned to: R-DN-MIR-CORE-01 → lo0
  - Role: Loopback
  - DNS: r-mir-core-01.nirvor.bd

103.125.42.130/30
  - Assigned to: R-DN-MIR-CORE-01 → sfp-sfpplus1
  - Role: Network
  - DNS: r-mir-uplink.nirvor.bd
  - Description: BTCL uplink

10.10.10.11/24
  - Assigned to: SW-DN-MIR-DIST-01 → vlan10
  - VLAN: MGMT_MIRPUR
  - DNS: sw-mir-dist-01.nirvor.bd
```

**5. ভি-ল্যান:**

```
VLAN 10 - MGMT_MIRPUR
  - Location: Mirpur POP
  - Prefixes: 10.10.10.0/24

VLAN 100 - RESIDENTIAL_MIR
  - Location: Mirpur POP
  - Prefixes: 103.125.40.0/24
```

**6. সার্কিট:**

```
Circuit: BTCL-MIR-001
  - Provider: BTCL
  - Type: Fiber Optic
  - Commit Rate: 5 Gbps
  - Termination Z:
    * Device: R-DN-MIR-CORE-01
    * Interface: sfp-sfpplus1
```

#### ডেটা ফ্লো ভিজুয়ালাইজেশন

```
BTCL Provider
    |
    | Circuit: BTCL-MIR-001 (5 Gbps)
    ↓
[R-DN-MIR-CORE-01] sfp-sfpplus1 (103.125.42.130/30)
    | lo0 (10.10.1.1/32)
    |
    | Cable: CORE-TO-DIST-01 (SMF, 5 m)
    | sfp-sfpplus2 ↔ sfp1
    ↓
[SW-DN-MIR-DIST-01] vlan10 (10.10.10.11/24)
    |
    | Cable: DIST-TO-ACC-01 (Cat6, 20 m)
    | ether1 ↔ ether24
    ↓
[SW-DN-MIR-ACC-01]
    | ether1-23: Customer connections
    |
    └→ Building A customers (VLAN 100)
```

এই পুরো স্ট্রাকচার নটবটের ডেটাবেসে স্টোর করা। এখন যদি কেউ জিজ্ঞাসা করে:

**প্রশ্ন:** "মিরপুর পপে মোট কয়টা ডিভাইস?"
**উত্তর:** নটবটে filter করুন Location = 'Mirpur POP'

**প্রশ্ন:** "BTCL uplink কোন ডিভাইসের কোন পোর্টে?"
**উত্তর:** Circuit BTCL-MIR-001 এর Termination Z দেখুন

**প্রশ্ন:** "Building A এর কাস্টমাররা কোন সুইচে আছে?"
**উত্তর:** Custom Field "Building Name" = Building A filter করুন

**প্রশ্ন:** "কোর রাউটার থেকে এক্সেস সুইচ পর্যন্ত physical path কী?"
**উত্তর:** Cable Trace ফিচার ব্যবহার করুন

---

### ডেটা ইন্টিগ্রিটি - নটোবট কীভাবে নিশ্চিত করে

নটোবট অনেক ভ্যালিডেশন করে যাতে ভুল ডেটা ঢুকতে না পারে।

#### রেফারেনশিয়াল ইন্টেগ্রিটি

যদি একটা লোকেশন মুছতে চান যেখানে ডিভাইস আছে, নটোবট বলবে:

```
Error: Cannot delete location "Mirpur POP" because:
  - 3 devices are assigned to this location
  - 2 racks are in this location

Delete or reassign these first.
```

#### ইউনিক কনস্ট্রেইন্ট

একই নামে দুটো ডিভাইস তৈরি করা যাবে না:

```
Error: Device with name "R-DN-MIR-CORE-01" already exists.
```

একই আইপি অ্যাড্রেস দুবার অ্যাসাইন করা যাবে না:

```
Error: IP address "10.10.1.1/32" is already assigned to:
  - Device: R-DN-MIR-CORE-01
  - Interface: lo0
```

#### স্ট্যাটাস ভ্যালিডেশন

অফলাইন ডিভাইস এ নতুন আইপি অ্যাসাইন করতে গেলে ওয়ার্নিং:

```
Warning: This device status is "Offline". 
Are you sure you want to assign an IP?
```

#### কাস্টম ভ্যালিডেশন

কাস্টম ভ্যালিডেশন রুল তৈরি করা যায় প্লাগইন দিয়ে। যেমন:

```
Rule: Device নামে অবশ্যই site code থাকতে হবে
  - Valid: R-DN-MIR-CORE-01 (DN আছে)
  - Invalid: R-MIR-CORE-01 (DN নেই)
```

---

### গ্রাফকিউএল এপিআই - অ্যাডভান্সড কোয়েরিং

Nautobot 3.0 এ গ্রাফকিউএল এপিআই আছে যা রেস্ট এপিআই থেকে শক্তিশালী।

**রেস্ট এপিআই তে:**

Mirpur POP এর সব ডিভাইস এবং তাদের আইপি অ্যাড্রেস নিতে:

```
GET /api/dcim/devices/?location=mirpur-pop
  - Response: Devices list
  
Then for each device:
GET /api/ipam/ip-addresses/?device=<device-id>
  - Response: IPs for that device
```

অনেকগুলো রিকোয়েস্ট লাগে।

**গ্রাফকিউএল এ:**

একটা রিকোয়েস্ট এ সব:

```graphql
query {
  devices(location: "mirpur-pop") {
    name
    device_type {
      model
    }
    interfaces {
      name
      ip_addresses {
        address
        dns_name
      }
    }
  }
}
```

রেসপন্স এ পুরো নেস্টেড ডাটা একসাথে আসবে। অনেক দ্রুত।

---

নির্ভর কমিউনিকেশন এখন বুঝে গেছে তাদের নেটওয়ার্ক কীভাবে নটবটে রিপ্রেজেন্ট হবে। পরের চ্যাপ্টারে আমরা দেখব কীভাবে ইউআই দিয়ে অ্যাকচুয়াল ডেটা এন্ট্রি করতে হয়। এভাবে একটু একটু করে তারা পুরো নেটওয়ার্ক নটবটে ডকুমেন্ট করবে। শুরুতে সময় লাগবে, কিন্তু একবার হয়ে গেলে পরে সব সহজ হয়ে যায়। পাশাপাশি, থিওরি থেকে প্র্যাকটিসে পুরোপুরি নামা যাবে।
