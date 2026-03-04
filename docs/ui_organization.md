আগের চ্যাপ্টারে আমরা Nautobot 3.0 এর ডেটা মডেল বুঝলাম - Organization, DCIM, IPAM, Circuits, Extras। এখন সময় এসেছে হাতে-কলমে কাজ শুরু করার। এই চ্যাপ্টারে আমরা দেখব কীভাবে নটবটের ওয়েব UI ব্যবহার করে actual ডেটা এন্ট্রি করতে হয়।

নির্ভর কমিউনিকেশনের NOC টিম এখন তাদের মিরপুর এবং উত্তরা পপের সম্পূর্ণ নেটওয়ার্ক নটোবট ডকুমেন্ট করবে। তাদের সাথে একসাথে আমরাও শিখব। শুরু করা যাক।

### Nautobot 3.0 ইউআই ওভারভিউ

প্রথমে একবার নটোবট ইউআই এর সাথে পরিচিত হয়ে নিই।

#### লগইন এবং ড্যাশবোর্ড

ব্রাউজারে যান:

```
https://nautobot.nirvor.bd:8080
```

লগইন করুন:
```
Username: admin
Password: Nirvor@Admin2025!
```

লগইন করার পরে আপনি ড্যাশবোর্ডে আসবেন। প্রথমবার লগইন করলে ড্যাশবোর্ড খালি দেখাবে - এটা স্বাভাবিক। এখনও কোনো ডেটা নেই।

#### নেভিগেশন বার

উপরে একটা নেভিগেশন বার দেখবেন। এখানে মূল মেনুগুলো:

**Organization:** Locations, Teams, Tenants, Contacts

**Devices:** Device Types, Manufacturers, Devices, Racks, Cables, Device Redundancy Groups

**IPAM:** Namespaces, Prefixes, IP Addresses, VLANs, VLAN Groups, RIRs

**Circuits:** Providers, Circuit Types, Circuits

**Power:** Power Feeds, Power Panels (আমরা এগুলো ব্যবহার করব না এই বইতে)

**Extras:** Roles, Statuses, Tags, Custom Fields, Relationships, Jobs, Git Repositories

**Plugins:** (যদি কোনো প্লাগইন ইনস্টল করা থাকে)

#### সার্চ ফিচার

ডান দিকে উপরের কোণায় একটা সার্চ বক্স দেখবেন। এটা গ্লোবাল সার্চ - যেকোনো কিছু এখানে টাইপ করলে নটোবট সব জায়গায় খুঁজবে।

টেস্ট করুন - টাইপ করুন "mirpur" (এখনও কিছু পাবেন না কারণ ডেটা নেই)।

#### ইউজার মেনু

একদম ডান দিকে উপরের কোণায় আপনার ইউজারনেম দেখবেন। ক্লিক করলে একটা ড্রপডাউন:

- **Profile:** আপনার প্রোফাইল দেখুন/এডিট করুন
- **Preferences:** ইউআই সেটিংস (items per page, theme ইত্যাদি)
- **API Tokens:** এপিআই টোকেন ম্যানেজ করুন
- **Change Password:** পাসওয়ার্ড চেঞ্জ করুন
- **Logout:** লগআউট

---

### লোকেশন হায়ারার্কি তৈরি করা

প্রথম কাজ হলো লোকেশন হায়ারারকি সেটআপ করা। এটা Nautobot 3.0 এর সবচেয়ে ইম্পর্ট্যান্ট ফিচার।

#### স্টেপ ১: লোকেশন টাইপ ডিফাইন করা

**Organization → Location Types** যান।

খালি পেজ দেখাবে। ডান দিকে উপরে **+ অ্যাড** বাটনে ক্লিক করুন।

**প্রথম Location Type: Zone**

```
Name: Zone
Description: Geographic coverage zone
Nestable: ✓ (চেক করুন)
Content Types: (খালি রাখুন - সব কিছুতে ইউজ হতে পারে)
```

**ক্রিয়েট** ক্লিক করুন।

এখন আরেকটা - **+ অ্যাড** আবার ক্লিক করুন।

**দ্বিতীয় Location Type: Cluster**

```
Name: Cluster
Description: Group of nearby POPs for operational management
Nestable: ✓
Parent Location Type: Zone (এটা ড্রপডাউন থেকে সিলেক্ট করুন)
```

**ক্রিয়েট** করুন।

**তৃতীয় Location Type: POP**

```
Name: POP
Description: Point of Presence - ISP network node
Nestable: ✓
Parent Location Type: Cluster
```

**চতুর্থ Location Type: Rack**

```
Name: Rack
Description: Equipment rack
Nestable: ✗ (আনচেক রাখুন - এর নিচে আর কিছু হবে না)
Parent Location Type: POP
```

এখন আপনার চারটা লোকেশন টাইপস তৈরি হয়েছে। **Organization → Location Types** এ গিয়ে দেখুন - সব লিস্টেড দেখাবে।

#### স্টেপ ২: অ্যাকচুয়াল লোকেশন  তৈরি করা

এখন রিয়েল লোকেশন তৈরি করব।

**Organization → Locations** যান। **+ অ্যাড** ক্লিক করুন।

**Top Level Location: Dhaka North Zone**

```
Name: Dhaka North Zone
Location Type: Zone (ড্রপডাউন থেকে সিলেক্ট করুন)
Status: Active (ড্রপডাউন থেকে)
Parent Location: (খালি রাখুন - এটা top level)
Tenant: (খালি রাখুন)
Description: Northern Dhaka metropolitan area coverage
```

**ক্রিয়েট** ক্লিক করুন।

এখন ভেতরে ঢুকুন - **Mirpur Cluster** তৈরি করব।

আবার **Organization → Locations → + অ্যাড** যান।

```
Name: Mirpur Cluster
Location Type: Cluster
Status: Active
Parent Location: Dhaka North Zone (এটা খুব ইম্পর্ট্যান্ট!)
Description: Mirpur and Kalyanpur area operations cluster
```

**ক্রিয়েট** করুন।

এবার Mirpur POP:

```
Name: Mirpur POP
Location Type: POP
Status: Active
Parent Location: Mirpur Cluster
Facility: Mirpur-12 Data Center
Physical Address: House 25, Road 10, Mirpur-12, Dhaka-1216, Bangladesh
Shipping Address: (Same as physical address)
Latitude: 23.8103
Longitude: 90.3654
Contact Name: Jahangir Khan
Contact Phone: +880 1712-345678
Contact Email: jahangir@nirvor.bd
Time Zone: Asia/Dhaka
Description: Primary POP serving Mirpur area. Approximately 4000 customers.
Comments: 
  - Established: January 2023
  - Building: 3-story commercial building
  - Power: Dual utility feed + 50 KVA generator backup
  - Cooling: 2x 5-ton AC units
```

**ক্রিয়েট** করুন।

**জিপিএস কোঅর্ডিনেট কীভাবে পাবেন?**

Google Maps এ যান → আপনার লোকেশন সার্চ করুন → রাইট ক্লিক → "What's here?" → নিচে কোঅর্ডিনেট দেখাবে → কপি করুন।

এখন একইভাবে Uttara Cluster এবং Uttara POP তৈরি করুন:

**Uttara Cluster:**

```
Name: Uttara Cluster
Location Type: Cluster
Status: Active
Parent Location: Dhaka North Zone
Description: Uttara and Baridhara area operations cluster
```

**Uttara POP:**

```
Name: Uttara POP
Location Type: POP
Status: Active
Parent Location: Uttara Cluster
Facility: Uttara Sector-7 Office
Physical Address: Plot 15, Road 12, Sector-7, Uttara, Dhaka-1230, Bangladesh
Latitude: 23.8759
Longitude: 90.3795
Contact Name: Asif Rahman
Contact Phone: +880 1811-234567
Contact Email: asif@nirvor.bd
Time Zone: Asia/Dhaka
Description: Secondary POP serving Uttara area. Approximately 4000 customers.
```

এখন **Organization → Locations** এ যান। আপনার লোকেশন হায়ারার্কি দেখুন:

```
Dhaka North Zone
  ├── Mirpur Cluster
  │    └── Mirpur POP
  └── Uttara Cluster
       └── Uttara POP
```

চমৎকার! লোকেশন স্ট্রাকচার রেডি।

---

### ইউনিফাইড রোল সেটআপ

Nautobot 3.0 এর একটা বড় চেঞ্জ হলো ইউনিফাইড রোল। এখন **Extras → Roles** এ সব রোল থাকবে।

#### ডিভাইস রোল তৈরি করা

**Extras → Roles** যান। **+ অ্যাড** ক্লিক করুন।

**প্রথম Role: Core Router**

```
Name: Core Router
Content Types: dcim | device (এটা খুব ইম্পর্ট্যান্ট - এই dropdown থেকে সিলেক্ট করুন)
Color: f44336 (লাল রঙের হেক্স কোড)
Weight: 1000 (যত বেশি, তত important)
Description: Core routing equipment at the heart of the network
```

**ক্রিয়েট** করুন।

কালার কীভাবে বেছে নেবেন? কালার পিকার দেখাবে অথবা হেক্স কোড সরাসরি দিতে পারেন:
- Red: f44336
- Orange: ff9800
- Green: 4caf50
- Blue: 2196f3
- Purple: 9c27b0

এখন আরো কয়েকটা রোল তৈরি করুন:

**জিপিএস কোঅর্ডিনেটস:**

```
Name: Distribution Switch
Content Types: dcim | device
Color: ff9800 (Orange)
Weight: 800
Description: Distribution layer switches aggregating access switches
```

**অ্যাক্সেস সুইচ:**

```
Name: Access Switch
Content Types: dcim | device
Color: 4caf50 (Green)
Weight: 600
Description: Access layer switches connecting to customers
```

**ফায়ারওয়াল:**

```
Name: Firewall
Content Types: dcim | device
Color: 9c27b0 (Purple)
Weight: 950
Description: Security devices and firewalls
```

এখন **Extras → Roles** এ গিয়ে দেখুন - চারটা রোল দেখাবে কালারফুল ব্যাজ সহ।

---

### ম্যানুফ্যাকচারার এবং ডিভাইস টাইপ

ডিভাইস যোগ করার আগে তাদের টাইপ ডিফাইন করতে হবে।

#### ম্যানুফ্যাকচারার যোগ করা

**Devices → Manufacturers** যান। **+ অ্যাড** ক্লিক করুন।

**MikroTik:**

```
Name: MikroTik
Description: Latvian network equipment manufacturer specializing in routers and wireless systems
```

**ক্রিয়েট** করুন।

একইভাবে আরো:

**TP-Link:**

```
Name: TP-Link
Description: Chinese networking equipment manufacturer
```

**Cisco:**

```
Name: Cisco
Description: American multinational technology company - networking equipment
```

#### ডিভাইস টাইপ তৈরি করা

এখন ডিভাইস টাইপ বানাবেন। এটা একটু সময় নেয় কারণ ইন্টারফেস টেমপ্লেট যোগ করতে হয়।

**Devices → Device Types** যান। **+ অ্যাড** ক্লিক করুন।

**MikroTik CCR2116:**

```
Manufacturer: MikroTik (ড্রপডাউন থেকে)
Model: CCR2116-12G-4S+
Part Number: CCR2116-12G-4S+
Height (U): 1
Is Full Depth: ✓
Subdevice Role: None
Description: Cloud Core Router with 1x 1G, 12x 10G SFP+, 2x 25G SFP28 ports
Comments: Typical use - Core router for small to medium ISPs
```

**ক্রিয়েট** ক্লিক করুন।

এখন ইন্টারফেস টেমপ্লেট যোগ করতে হবে। Device Type এর ডিটেইল পেজে আসবেন।

**Interface Templates** ট্যাবে ক্লিক করুন। **+ অ্যাড Interface Templates** বাটনে ক্লিক করুন।

একসাথে একাধিক ইন্টারফেস যোগ করতে পারবেন:

**ম্যানেজমেন্ট ইন্টারফেস:**

```
Name: ether1
Type: 1000BASE-T (1GE)
Management Only: ✓ (চেক করুন - এটা ম্যানেজমেন্ট পোর্ট)
```

**ক্রিয়েট অ্যাড অ্যানাদার** ক্লিক করুন।

**SFP+ Ports (একটা একটা করে বা প্যাটার্ন ব্যবহার করে):**

Nautobot smart - আপনি প্যাটার্ন ব্যবহার করতে পারেন:

```
Name: sfp-sfpplus[1-12]
Type: 10GBASE-X-SFP+
Management Only: (আনচেক)
```

এটা করলে sfp-sfpplus1 থেকে sfp-sfpplus12 পর্যন্ত ১২টা ইন্টারফেস একসাথে তৈরি হবে।

**ক্রিয়েট  অ্যাড অ্যানাদার** ক্লিক করুন।

**SFP28 Ports:**

```
Name: sfp28-[1-2]
Type: 25GBASE-X-SFP28
Management Only: (আনচেক)
```

এখন **ক্রিয়েট** ক্লিক করুন (শেষ যেহেতু)।

ইন্টারফেস টেমপ্লেট ট্যাবে গিয়ে দেখুন - ১৫টা ইন্টারফেস দেখাবে (১ + ১২ + ২)।

চমৎকার! এখন যখনই এই ডিভাইস টাইপের একটা নতুন ডিভাইস তৈরি করবেন, সব ইন্টারফেস অটোমেটিক্যালি কপি হয়ে যাবে।

একইভাবে আরো দুটো ডিভাইস টাইপ তৈরি করুন:

**TP-Link TL-SX3016F:**

```
Manufacturer: TP-Link
Model: TL-SX3016F
Height (U): 1
Is Full Depth: ✓
Description: TP-Link TL-SX3016F JetStream 16-Port 10GE SFP+ L2+ Managed Switch with 4 SFP slots

Interface Templates:
  - ether[1-24]: Type 1000BASE-T
  - sfp[1-4]: Type 1000BASE-X-SFP
```

**TP-Link TL-SG1024D:**

```
Manufacturer: TP-Link
Model: TL-SG3428MP
Height (U): 1
Is Full Depth: ✓
Description: TP-Link TL-SG3428MP JetStream 28-Port Gigabit L2+ Managed Switch with 24-Port PoE+

Interface Templates:
  - ether[1-24]: Type 1000BASE-T
```

---

### র‍্যাক তৈরি করা

এখন র‍্যাক যোগ করব যেখানে ডিভাইস মাউন্ট করা হবে।

**Devices → Racks** যান। **+ অ্যাড** ক্লিক করুন।

**Mirpur POP Rack A:**

```
Name: Rack A
Location: Mirpur POP (ড্রপডাউন থেকে সিলেক্ট করুন - এটা hierarchy তে দেখাবে)
Status: Active
Facility ID: MIR-RACK-A
Type: 4-post cabinet
Width: 19 inches
Height (U): 42
Outer Width: 600 (mm)
Outer Depth: 1000 (mm)
Description: Primary equipment rack in Mirpur POP
Comments: Rack installed January 2023. APC NetShelter SV 42U.
```

**ক্রিয়েট and অ্যাড অ্যানাদার** ক্লিক করুন (যদি আরো র‍্যাক যোগ করতে চান)।

**Mirpur POP Rack B:**

```
Name: Rack B
Location: Mirpur POP
Status: Active
Facility ID: MIR-RACK-B
Type: 4-post cabinet
Width: 19 inches
Height (U): 42
Description: Secondary equipment rack in Mirpur POP
```

**ক্রিয়েট** ক্লিক করুন।

একইভাবে Uttara POP এর জন্য:

**Uttara POP Rack A:**

```
Name: Rack A
Location: Uttara POP
Status: Active
Facility ID: UTT-RACK-A
Type: 4-post cabinet
Width: 19 inches
Height (U): 42
Description: Primary equipment rack in Uttara POP
```

এখন **Devices → Racks** লিস্টে গিয়ে দেখুন - তিনটা র‍্যাক দেখাবে।

---

### প্রথম ডিভাইস এন্ট্রি - মিরপুর কোর রাউটার

এখন সবচেয়ে এক্সাইটিং অংশ - প্রথম ডিভাইস যোগ করা!

**Devices → Devices** যান। **+ অ্যাড** ক্লিক করুন।

একটা বড় ফর্ম আসবে। ঘাবড়াবেন না - ধাপে ধাপে পূরণ করব।

```
Name: R-DN-MIR-CORE-01
Device Type: MikroTik CCR2116-12G-4S+ (টাইপ করলে autocomplete আসবে)
Role: Core Router
Location: Mirpur POP (Hierarchy দেখাবে: Dhaka North Zone > Mirpur Cluster > Mirpur POP)
```

এই চারটা ফিল্ড সবচেয়ে ইম্পর্ট্যান্ট। এগুলো রিকোয়ার্ড।

এখন অপশনাল কিন্তু রেকমেন্ডেড ফিল্ডগুলো:

```
Rack: Rack A
Position: 25 (মানে র‍্যাকের ২৫U পজিশনে)
Face: Front

Status: Active

Tenant: (খালি রাখুন - Nirvor Communication একটাই organization)
Platform: (খালি রাখুন - এটা ওএস প্ল্যাটফর্ম এর জন্য, অপশনাল)

Serial Number: ABC1234MIR001
Asset Tag: SKY-RTR-001

Description: Primary core router for Mirpur POP serving approximately 4000 customers

Comments:
  - Purchased: December 2022
  - Vendor: TechSource Bangladesh
  - Purchase Order: PO-2022-12-015
  - Warranty: 3 years (expires Dec 2025)
  - Installation Date: January 15, 2023
```

সব পূরণ হয়ে গেলে **ক্রিয়েট** ক্লিক করুন।

আপনার প্রথম ডিভাইস তৈরি হয়ে গেছে!

ডিভাইস ডিটেইল পেজে চলে যাবেন। এখানে দেখবেন:

- **Device Info:** নাম, টাইপ, লোকেশন, স্ট্যাটাস
- **Rack Position:** একটা ভিজ্যুয়াল ডায়াগ্রাম যেখানে র‍্যাকে ডিভাইসের পজিশন দেখাচ্ছে
- **Interfaces:** সব ইন্টারফেসের লিস্ট (১৫টা - ডিভাইস টাইপ থেকে কপি হয়েছে)

**Interfaces** ট্যাবে ক্লিক করুন। দেখবেন:

```
ether1
sfp-sfpplus1
sfp-sfpplus2
...
sfp-sfpplus12
sfp28-1
sfp28-2
```

চমৎকার! সব ইন্টারফেস অটোমেটিক্যালি এসেছে।

---

### ইন্টারফেস কনফিগারেশন - ডিটেইলস যোগ করা

এখন ইন্টারফেসগুলোতে কিছু ডিটেইলস যোগ করব।

ether1 ইন্টারফেসে ক্লিক করুন। একটা ডিটেইল পেজ আসবে। **এডিট** বাটনে ক্লিক করুন।

```
Name: ether1 (ইতিমধ্যে আছে)
Type: 1000BASE-T (ইতিমধ্যে সেট)

Label: MGMT (optional - physical label)
Enabled: ✓ (চেক করুন - এই পোর্ট চালু আছে)

Speed: 1000000 (1 Gbps - Kbps এ)
Duplex: Full

MTU: 1500

Mode: Access (না Tagged)

Description: Management interface - connected to management VLAN
```

**আপডেট** ক্লিক করুন।

একইভাবে আপলিঙ্ক ইন্টারফেস:

sfp-sfpplus1 এ যান → **এডিট** ক্লিক করুন:

```
Label: UPLINK-BTCL
Enabled: ✓
Speed: 10000000 (10 Gbps)
MTU: 1500
Description: Primary uplink to BTCL - 5 Gbps circuit
```

**আপডেট** করুন।

আপনি চাইলে সব ইন্টারফেসে ডিসক্রিপশন দিতে পারেন। তবে শুধু যেগুলো ব্যবহার হচ্ছে সেগুলোতে দিলেই চলবে।

---

### আরো ডিভাইস যোগ করা - বাল্ক এর জন্য প্রস্তুতি

এখন আরো কিছু ডিভাইস যোগ করব। জিপিএস কোঅর্ডিনেটস এবং অ্যাক্সেস সুইচ।

**জিপিএস কোঅর্ডিনেটস - মিরপুর:**

**Devices → Devices → + অ্যাড**

```
Name: SW-DN-MIR-DIST-01
Device Type: TP-Link TL-SX3016F
Role: Distribution Switch
Location: Mirpur POP
Rack: Rack A
Position: 20
Face: Front
Status: Active
Serial Number: TPL1234DIST01
Asset Tag: SKY-SW-001
Description: Primary distribution switch for Mirpur POP
```

**ক্রিয়েট** করুন।

**অ্যাক্সেস সুইচ - একটা একটা করে (পরে সিএসভি দেখাব):**

প্রথম অ্যাক্সেস সুইচ:

```
Name: SW-DN-MIR-ACC-01
Device Type: TP-Link TL-SG1024D
Role: Access Switch
Location: Mirpur POP
Rack: Rack B
Position: 15
Status: Active
Serial Number: TPLACC001
Asset Tag: SKY-SW-011
Description: Building A access switch
```

দ্বিতীয় অ্যাক্সেস সুইচ:

```
Name: SW-DN-MIR-ACC-02
Device Type: TP-Link TL-SG1024D
Role: Access Switch
Location: Mirpur POP
Rack: Rack B
Position: 14
Status: Active
Serial Number: TPLACC002
Asset Tag: SKY-SW-012
Description: Building B access switch
```

এভাবে একটা একটা করে করা বিরক্তিকর। দশটা অ্যাক্সেস সুইচ থাকলে তো পুরো দিন চলে যাবে!

---

### সিএসভি বাল্ক ইমপোর্ট - দ্রুত ডেটা এন্ট্রি

নটবটে সিএসভি দিয়ে বাল্ক ইমপোর্ট  করার সুবিধা আছে। চলুন দেখি কীভাবে।

#### সিএসভি টেমপ্লেট ডাউনলোড করা

**Devices → Devices** লিস্ট পেজে যান।

উপরে ডান দিকে **Import** বাটন দেখবেন। ক্লিক করুন।

একটা পেজ আসবে যেখানে সিএসভি ফরম্যাট দেখানো আছে। নিচে **CSV Field Mapping** সেকশনে সব ফিল্ডের লিস্ট আছে।

#### সিএসভি ফাইল তৈরি করা

এক্সেল বা গুগল শিটস এ একটা নতুন ফাইল তৈরি করুন।

প্রথম রো তে হেডার:

```csv
name,device_type,role,location,rack,position,status,serial,asset_tag,description
```

এখন ডেটা রো:

```csv
name,device_type,role,location,rack,position,status,serial,asset_tag,description
SW-DN-MIR-ACC-03,TP-Link TL-SG1024D,Access Switch,Mirpur POP,Rack B,13,Active,TPLACC003,SKY-SW-013,Building C access switch
SW-DN-MIR-ACC-04,TP-Link TL-SG1024D,Access Switch,Mirpur POP,Rack B,12,Active,TPLACC004,SKY-SW-014,Building D access switch
SW-DN-MIR-ACC-05,TP-Link TL-SG1024D,Access Switch,Mirpur POP,Rack B,11,Active,TPLACC005,SKY-SW-015,Building E access switch
```

**গুরুত্বপূর্ণ নোট:**

- ডিভাইস টাইপ, রোল, লোকেশন, র্যাক এর নাম হুবহু নটবটে যেভাবে লেখা আছে সেভাবে লিখতে হবে
- স্ট্যাটাস ও এক্স্যাক্ট মিলতে হবে (অ্যাক্টিভ, প্ল্যানড, অফলাইন ইত্যাদি)
- পজিশন একটা নাম্বার
- সিরিয়াল এবং অ্যাসেট ট্যাগ ইউনিক হতে হবে

ফাইল সিএসভি ফরম্যাটে সেভ করুন (`devices.csv`)।

#### সিএসভি ইমপোর্ট করা

**Devices → Devices → Import** যান।

**CSV Data** বক্সে আপনার সিএসভি ফাইলের কন্টেন্ট পেস্ট করুন। অথবা **Browse** বাটনে ক্লিক করে ফাইল আপলোড করুন।

**সাবমিট** ক্লিক করুন।

নটোবট সিএসভি পার্স করবে এবং একটা প্রিভিউ দেখাবে:

```
Found 3 objects to import:
  - SW-DN-MIR-ACC-03
  - SW-DN-MIR-ACC-04
  - SW-DN-MIR-ACC-05
```

যদি কোনো এরর থাকে (যেমন Device Type নাম ভুল), তাহলে সেটা লাল করে দেখাবে।

সব ঠিক থাকলে **Confirm Import** বাটনে ক্লিক করুন।

তিনটা ডিভাইস একসাথে তৈরি হয়ে যাবে!

**Devices → Devices** লিস্টে গিয়ে দেখুন - এখন আপনার মোট ৭টা ডিভাইস আছে:
- ১টা Router
- ১টা Distribution Switch
- ৫টা Access Switch

---

### আইপি অ্যাড্রেস অ্যাসাইনমেন্ট - লজিকাল কানেকশন

এখন ডিভাইসগুলোতে আইপি অ্যাড্রেস অ্যাসাইন করব।

#### প্রথমে IP Namespace তৈরি

**IPAM → Namespaces** যান। **+ অ্যাড** ক্লিক করুন।

```
Name: Global
Description: Primary IP namespace for Nirvor Communication
```

**ক্রিয়েট** করুন।

#### IP Prefixes তৈরি করা

IP addresses assign করার আগে prefixes তৈরি করতে হবে।

**IPAM → Prefixes** যান। **+ অ্যাড** ক্লিক করুন।

**প্যারেন্ট প্রিফিক্স - প্রাইভেট ম্যানেজমেন্ট নেটওয়ার্ক:**

```
Prefix: 10.10.0.0/16
Type: Container
Status: Active
Namespace: Global
Description: Internal management network for all Nirvor Communication equipment
```

**ক্রিয়েট and অ্যাড অ্যানাদার** করুন।

**Child Prefix - Mirpur Management:**

```
Prefix: 10.10.10.0/24
Type: Network
Status: Active
Namespace: Global
Parent: 10.10.0.0/16 (ড্রপডাউন থেকে সিলেক্ট করুন)
Location: Mirpur POP
Description: Management IPs for Mirpur POP devices
```

**ক্রিয়েট** করুন।

#### IP Addresses তৈরি এবং Interface এ Assign করা

**IPAM → IP Addresses** যান। **+ অ্যাড** ক্লিক করুন।

**Core Router Loopback:**

```
IP Address: 10.10.1.1/32
Type: Host
Status: Active
Role: Loopback (ড্রপডাউন থেকে)
Namespace: Global
Parent Prefix: (খালি রাখুন - auto-detect হবে)

Assigned Object Type: dcim > interface
Assigned Object: 
  - Device: R-DN-MIR-CORE-01
  - Interface: ether1 (ড্রপডাউনে দেখাবে)

DNS Name: r-mir-core-01.nirvor.bd
Description: Management loopback for Mirpur core router
```

**ক্রিয়েট** করুন।

লক্ষ্য করুন - Assigned Object টা একটু ট্রিকি। প্রথমে Type সিলেক্ট করতে হবে `dcim > interface`, তারপর দুটো ড্রপডাউন আসবে - Device এবং Interface।

**জিপিএস কোঅর্ডিনেটস ম্যানেজমেন্ট আইপি:**

```
IP Address: 10.10.10.11/24
Type: Host
Status: Active
Namespace: Global

Assigned Object Type: dcim > interface
Assigned Object:
  - Device: SW-DN-MIR-DIST-01
  - Interface: (এখানে একটু সমস্যা - TP-Link switch এ VLAN interface নেই)
```

একটু দাঁড়ান - TP-Link TL-SX3016F এ management VLAN interface যোগ করা হয়নি। চলুন যোগ করি।

**Devices → Devices → SW-DN-MIR-DIST-01** এ যান।

**Interfaces** ট্যাবে → **+ অ্যাড Interfaces** ক্লিক করুন।

```
Name: vlan10
Type: Virtual
Enabled: ✓
Description: Management VLAN interface
```

**ক্রিয়েট** করুন।

এখন আবার আইপি অ্যাড্রেস অ্যাসাইনমেন্ট এ ফিরে যান:

```
IP Address: 10.10.10.11/24
Assigned Object:
  - Device: SW-DN-MIR-DIST-01
  - Interface: vlan10

DNS Name: sw-mir-dist-01.nirvor.bd
```

**ক্রিয়েট** করুন।

একইভাবে অ্যাক্সেস সুইচ এ:

```
10.10.10.21/24 → SW-DN-MIR-ACC-01 (vlan10 interface তৈরি করে)
10.10.10.22/24 → SW-DN-MIR-ACC-02
10.10.10.23/24 → SW-DN-MIR-ACC-03
...
```

এগুলো CSV দিয়েও করা যায়, কিন্তু Interface assignment একটু tricky CSV এ।

#### VLAN Management - লেয়ার ২ সেগমেন্টেশন

নির্ভর কমিউনিকেশনের নেটওয়ার্কে বিভিন্ন ধরনের ট্রাফিক আছে - ম্যানেজমেন্ট, কাস্টমার, আপলিংক। এগুলো আলাদা করার জন্য ভি-ল্যান ব্যবহার করা হয়।

##### ভি-ল্যান গ্রুপ তৈরি করা (অপশনাল কিন্তু রেকমেন্ডেড)

ভি-ল্যান গ্রুপ দিয়ে ভি-ল্যান গ্রুপস অর্গানাইজ করা যায়। বিশেষ করে যখন একই ভি-ল্যান গ্রুপস আইডি বিভিন্ন সাইটে ভিন্ন পারপাস এ ব্যবহার হয়।

**IPAM → VLAN Groups** যান। **+ অ্যাড** ক্লিক করুন।

```
Name: Mirpur POP VLANs
Location: Mirpur POP
Description: VLAN assignments for Mirpur Point of Presence
```

**ক্রিয়েট** করুন।

একইভাবে Uttara POP এর জন্য:

```
Name: Uttara POP VLANs
Location: Uttara POP
Description: VLAN assignments for Uttara Point of Presence
```

#### VLANs তৈরি করা

এখন অ্যাকচুয়াল ভি-ল্যান তৈরি করব।

**IPAM → VLANs** যান। **+ অ্যাড** ক্লিক করুন।

**VLAN 10 - Management (Mirpur):**

```
VLAN ID: 10
Name: MGMT_MIRPUR
Status: Active
Group: Mirpur POP VLANs (ড্রপডাউন থেকে)
Location: Mirpur POP
Tenant: (খালি)
Description: Management VLAN for network devices in Mirpur POP
```

**ক্রিয়েট and অ্যাড অ্যানাদার** করুন।

**VLAN 20 - Uplink (Mirpur):**

```
VLAN ID: 20
Name: UPLINK_MIRPUR
Status: Active
Group: Mirpur POP VLANs
Location: Mirpur POP
Description: Uplink traffic to BTCL from Mirpur
```

**ক্রিয়েট and অ্যাড অ্যানাদার** করুন।

**VLAN 100 - Residential (Mirpur):**

```
VLAN ID: 100
Name: RESIDENTIAL_MIR
Status: Active
Group: Mirpur POP VLANs
Location: Mirpur POP
Description: Residential customer traffic - Mirpur area
```

**ক্রিয়েট** করুন।

একইভাবে Uttara POP এর জন্য VLANs তৈরি করুন:

```
VLAN 10: MGMT_UTTARA (Group: Uttara POP VLANs, Location: Uttara POP)
VLAN 21: UPLINK_UTTARA (Group: Uttara POP VLANs)
VLAN 101: RESIDENTIAL_UTT (Group: Uttara POP VLANs)
```

**সাইট-অ্যাগনস্টিক ভি-ল্যান (সব সাইটে একই):**

কিছু ভি-ল্যান সব সাইটে একই পারপাস এ ব্যবহার হয়:

```
VLAN ID: 200
Name: CORPORATE
Status: Active
Group: (খালি - কোনো specific group এ নেই)
Location: (খালি - সব সাইটে ব্যবহার হয়)
Description: Corporate client connections across all POPs
```

এখন **IPAM → VLANs** লিস্টে গিয়ে দেখুন - আপনার সব VLANs দেখাবে VLAN ID, Name, Location সহ।

#### VLAN এবং Prefix এর সম্পর্ক

এখন ভি-ল্যান এর সাথে আইপি প্রিফিক্স লিঙ্ক করব।

**IPAM → Prefixes** এ যান। আগে তৈরি করা 10.10.10.0/24 prefix খুঁজুন। ক্লিক করে ডিটেইল পেজে যান।

**এডিট** বাটনে ক্লিক করুন।

```
Prefix: 10.10.10.0/24
...
(সব ফিল্ড আগের মতো)

VLAN: MGMT_MIRPUR (এটা নতুন যোগ করুন - ড্রপডাউন থেকে সিলেক্ট)
```

**আপডেট** করুন।

এখন নটোবট জানে 10.10.10.0/24 prefix টা VLAN 10 (MGMT_MIRPUR) এ ব্যবহার হয়।

---

### কেবল ম্যানেজমেন্ট - ফিজিকাল কানেকশন

এখন ডিভাইসগুলোর মধ্যে ফিজিকাল কেবল কানেকশন ডকুমেন্ট করব।

#### Cable তৈরি করা - Method 1 (Interface থেকে)

সবচেয়ে সহজ পদ্ধতি হলো একটা ইন্টারফেস থেকে সরাসরি কেবল connect করা।

**Devices → Devices → R-DN-MIR-CORE-01** এ যান।

**Interfaces** ট্যাবে ক্লিক করুন।

**sfp-sfpplus2** interface খুঁজুন (এটা জিপিএস কোঅর্ডিনেটস এ যাবে)। ক্লিক করুন।

ইন্টারফেস ডিটেইল পেজে **Connect Cable** বাটন দেখবেন। ক্লিক করুন।

একটা ফর্ম আসবে:

```
Termination A (already filled):
  - Device: R-DN-MIR-CORE-01
  - Interface: sfp-sfpplus2

Termination B:
  - Termination Type: Interface (default selected)
  - Device: SW-DN-MIR-DIST-01 (ড্রপডাউন থেকে সিলেক্ট করুন)
  - Name: sfp1 (এই switch এর SFP port)

Cable:
  - Type: Single-Mode Fiber (SMF) (ড্রপডাউন থেকে)
  - Status: Connected
  - Color: 0000ff (Blue - হেক্স কোড)
  - Length: 5
  - Length Unit: Meters
  - Label: CORE-TO-DIST-01

Description: Core router to distribution switch - 10G fiber link
```

**ক্রিয়েট** ক্লিক করুন।

Cable তৈরি হয়ে গেছে! এখন R-DN-MIR-CORE-01 এর sfp-sfpplus2 interface এ গেলে দেখবেন "Cable: CORE-TO-DIST-01" লেখা আছে। ক্লিক করলে cable details দেখাবে।

#### Cable তৈরি করা -মেথড ২ (সরাসরি কেবল অবজেক্ট)

আরেকটা পদ্ধতি হলো সরাসরি কেবল অবজেক্ট তৈরি করা।

**Devices → Cables** যান। **+ অ্যাড** ক্লিক করুন।

```
Termination A:
  - Termination Type: dcim > interface
  - Device: SW-DN-MIR-DIST-01
  - Name: ether1

Termination B:
  - Termination Type: dcim > interface
  - Device: SW-DN-MIR-ACC-01
  - Name: ether24

Cable:
  - Type: Cat6 (Category 6 Ethernet)
  - Status: Connected
  - Color: ffeb3b (Yellow)
  - Length: 20
  - Length Unit: Meters
  - Label: DIST-TO-ACC-01

Description: distribution to access switch - Building A
```

**ক্রিয়েট** করুন।

একইভাবে আরো কয়েকটা কেবল তৈরি করুন:

**ডিস্ট্রিবিউশন টু অ্যাক্সেস সুইচ 02:**

```
SW-DN-MIR-DIST-01 (ether2) ↔ SW-DN-MIR-ACC-02 (ether24)
Type: Cat6, Color: Yellow, Length: 22m
Label: DIST-TO-ACC-02
```

**ডিস্ট্রিবিউশন টু অ্যাক্সেস সুইচ 03:**

```
SW-DN-MIR-DIST-01 (ether3) ↔ SW-DN-MIR-ACC-03 (ether24)
Type: Cat6, Color: Yellow, Length: 25m
Label: DIST-TO-ACC-03
```

#### Cable Types - কোনগুলো অ্যাভেইলেবল?

নটবটে বিল্ট-ইন কেবল টাইপ আছে:

**Copper:**
- Cat3, Cat5, Cat5e, Cat6, Cat6a, Cat7, Cat8
- Coaxial
- Direct Attach Copper (DAC)

**Fiber:**
- Multi-Mode Fiber (MMF)
- Single-Mode Fiber (SMF)
- Active Optical Cabling (AOC)

**Power:**
- AC Power
- DC Power

আপনার যেটা দরকার সেটা সিলেক্ট করুন।

#### Cable Trace ব্যবহার করা

এখন একটা পাওয়ারফুল ফিচার দেখি - কেবল ট্রেস।

**Devices → Devices → SW-DN-MIR-ACC-01** এ যান।

**Interfaces** ট্যাবে → **ether24** interface ক্লিক করুন।

ইন্টারফেস পেজে **Trace** বাটন দেখবেন। ক্লিক করুন।

একটা ভিজ্যুয়াল ডায়াগ্রাম দেখাবে:

```
SW-DN-MIR-ACC-01 (ether24)
    |
    | Cable: DIST-TO-ACC-01 (Cat6, 20m, Yellow)
    ↓
SW-DN-MIR-DIST-01 (ether1)
    |
    | Cable: CORE-TO-DIST-01 (SMF, 5m, Blue)
    ↓
R-DN-MIR-CORE-01 (sfp-sfpplus2)
```

এভাবে পুরো ফিজিক্যাল পাথ দেখতে পাবেন। এটা ট্রাবলশুটিং এর সময় অসাধারণ কাজে লাগে।

---

### Provider এবং Circuit সেটআপ

নির্ভর কমিউনিকেশনের আপস্ট্রিম কানেক্টিভিটি বিটিসিএল থেকে। চলুন এটা ডকুমেন্ট করি।

#### Provider তৈরি করা

**Circuits → Providers** যান। **+ অ্যাড** ক্লিক করুন।

```
Name: BTCL
ASN: 17494
Account Number: SKYN-BTCL-2023-MIR
Portal URL: https://isp.btcl.gov.bd
NOC Contact: +880 2-9555555
Admin Contact: noc@btcl.gov.bd

Description: Bangladesh Telecommunications Company Limited - Primary upstream provider

Comments:
  - Account Manager: Mr. Kamal Hossain
  - Sales Contact: +880 2-9555556
  - Technical Support: support@btcl.gov.bd
  - 24/7 NOC: +880 1700-000000
```

**ক্রিয়েট** করুন।

#### Circuit Type তৈরি করা

**Circuits → Circuit Types** যান। **+ অ্যাড** ক্লিক করুন।

```
Name: Fiber Optic
Description: Fiber optic connectivity - single-mode or multi-mode
```

**ক্রিয়েট** করুন।

#### Circuit তৈরি করা

**Circuits → Circuits** যান। **+ অ্যাড** ক্লিক করুন।

```
Circuit ID: BTCL-MIR-001
Provider: BTCL (ড্রপডাউন থেকে)
Type: Fiber Optic
Status: Active

Install Date: 2023-03-15
Termination Date: (খালি - চলমান)
Commit Rate: 5000 (Mbps - ৫ Gbps)

Description: Primary 5Gbps internet uplink from Mirpur POP

Comments:
  - Contract Start: 2023-03-01
  - Contract Duration: 2 years
  - Monthly Cost: BDT 400,000
  - Contract Renewal Date: 2025-03-01
  - Bandwidth Burstable: Up to 6 Gbps
  - SLA: 99.5% uptime guarantee
```

**ক্রিয়েট** করুন।

এখন সার্কিট টার্মিনেশন যোগ করতে হবে।

Circuit ডিটেইল পেজে **Terminations** ট্যাবে যান। **+ অ্যাড Termination** ক্লিক করুন।

**Termination A (Provider Side):**

```
Circuit: BTCL-MIR-001 (already selected)
Term Side: A
Location: (খালি - provider সাইড)
Provider Network: (খালি)
Port Speed: 5000 (Mbps)
Upstream Speed: 5000 (Mbps)
Cross Connect ID: BTCL-XC-MIR-12345
Patch Panel/Port: PP-BTCL-MIR-001
Description: BTCL side termination at their Mirpur exchange
```

**ক্রিয়েট  অ্যাড অ্যানাদার** করুন।

**Termination Z (Nirvor Communication Side):**

```
Circuit: BTCL-MIR-001
Term Side: Z
Location: Mirpur POP (ড্রপডাউন থেকে)
Device: R-DN-MIR-CORE-01
Interface: sfp-sfpplus1
Port Speed: 5000 (Mbps)
Description: Connected to core router uplink port
```

**ক্রিয়েট** করুন।

এখন সার্কিট এবং ডিভাইস পুরোপুরি লিঙ্কড। সার্কিট ডিটেইল পেজে দেখবেন Termination Z এ ডিভাইস এবং ইন্টারফেস এর লিংক আছে। ক্লিক করলে সরাসরি সেই ইন্টারফেস এ যাবেন।

একইভাবে উত্তরা পপের জন্য সামিট কমিউনিকেশন এর সার্কিট তৈরি করুন (যদি থাকে)।

---

### Data Validation এবং Quality Check

এতক্ষণ অনেক ডেটা এন্ট্রি করলাম। এখন চেক করা দরকার সব ঠিক আছে কিনা।

#### ডিভাইস লিস্ট রিভিউ

**Devices → Devices** যান।

উপরে ডান দিকে **Filters** প্যানেল দেখবেন। এখান থেকে ফিল্টার করতে পারবেন:

**Location দিয়ে filter:**
- Location: Mirpur POP সিলেক্ট করুন
- শুধু মিরপুর পপের ডিভাইস দেখাবে

**Role দিয়ে filter:**
- Role: Core Router সিলেক্ট করুন
- শুধু core routers দেখাবে

**Status দিয়ে filter:**
- Status: Active সিলেক্ট করুন
- শুধু active ডিভাইস দেখাবে

#### Missing Information চেক করা

কিছু কমন ইস্যু খুঁজুন:

**ডিভাইসেস উইদাউট সিরিয়াল নাম্বার:**

Filters প্যানেলে:
- Has Serial Number: No সিলেক্ট করুন

যদি কোনো ডিভাইস আসে, তাহলে সেগুলোতে সিরিয়াল নাম্বার  যোগ করুন।

**ডিভাইস উইদাউট লোকেশন:**

Filters:
- Location: (empty) - অর্থাৎ কোনো লোকেশন নেই

এরকম ডিভাইস থাকা উচিত না। থাকলে সংশোধন করুন।

#### আইপি অ্যাড্রেস ইউটিলাইজেশন দেখা

**IPAM → Prefixes** যান।

10.10.10.0/24 prefix ক্লিক করুন।

ডিটেইল পেজে দেখবেন:

```
Utilization: 23% (6 of 256 IPs used)
```

একটা ভিজুয়াল বার দেখাবে কতটুকু ব্যবহৃত। যদি 80% এর উপরে যায়, তাহলে নতুন প্রিফিক্স প্ল্যান করুন।

**IP Addresses** ট্যাবে ক্লিক করলে এই প্রিফিক্স এর সব আইপি দেখাবে:

```
10.10.10.11/24 - SW-DN-MIR-DIST-01 (Active)
10.10.10.21/24 - SW-DN-MIR-ACC-01 (Active)
10.10.10.22/24 - SW-DN-MIR-ACC-02 (Active)
...
```

#### ডুপ্লিকেট নেম চেক

নটোবট অটোমেটিকালি ডুপ্লিকেট নেমস প্রিভেন্ট করে, কিন্তু ডাবল-চেক করা ভালো।

**Devices → Devices** লিস্টে নেম কলাম এ সোর্ট করুন (কলাম হেডার এ ক্লিক করুন)। দেখুন কোনো ডুপ্লিকেট আছে কিনা।

---

### কমন মিস্টেক এবং কীভাবে এড়াবেন

নির্ভর কমিউনিকেশনের এনওসি টিম প্রথমে কিছু ভুল করেছিল। চলুন সেগুলো দেখি যাতে আপনি এড়াতে পারেন।

#### Mistake 1: লোকেশন হায়ারারকি ভুল

**ভুল করা হয়েছিল:**

Mirpur POP তৈরি করার সময় প্যারেন্ট লোকেশন দেওয়া হয়নি। ফলে এটা টপ-লেভেল এ চলে গিয়েছিল।

```
(ভুল হায়ারারকি)
Dhaka North Zone
Mirpur Cluster
Mirpur POP (parent নেই - top level এ!)
```

**সমাধান:**

Mirpur POP এডিট করে Parent Location = Mirpur Cluster সেট করতে হয়েছে।

**শিক্ষা:** লোকেশন তৈরি করার সময় সবসময় প্যারেন্ট লোকেশন চেক করুন।

#### Mistake 2: ডিভাইস টাইপ এ ইন্টারফেস টেমপ্লেট ভুলে যাওয়া

**ভুল:**

TP-Link TL-SG1024D ডিভাইস টাইপ তৈরি করা হয়েছিল কিন্তু ইন্টারফেস টেমপ্লেট যোগ করা হয়নি। ফলে ডিভাইস তৈরি করার পরে ম্যানুয়ালি ২৪টা ইন্টারফেস যোগ করতে হয়েছে প্রতিটা ডিভাইস এ!

**সমাধান:**

ডিভাইস টাইপ এ একবার ইন্টারফেস টেমপ্লেট ঠিকমতো যোগ করার পরে সব ঠিক হয়েছে।

**শিক্ষা:** ডিভাইস টাইপ তৈরি করার সময় অবশ্যই ইন্টারফেস টেমপ্লেট যোগ করুন। একটু সময় লাগবে, কিন্তু পরে অনেক সময় বাঁচবে।

#### Mistake 3: আইপি অ্যাড্রেস এ প্রিফিক্স মাস্ক ভুল

**ভুল:**

ম্যানেজমেন্ট আইপি দেওয়া হয়েছিল 10.10.10.11/32 (একটা host IP হিসেবে)। কিন্তু আসলে এটা একটা network এর part, তাই হওয়া উচিত ছিল 10.10.10.11/24।

**সমস্যা:**

/32 দিলে ওই IP শুধু নিজের সাথে কমিউনিকেট করতে পারে, network এর অন্য কারো সাথে না!

**সমাধান:**

সব আইপি অ্যাড্রেস রিভিউ করে সঠিক সাবনেট মাস্ক দিতে হয়েছে।

**শিক্ষা:** 
- Host IPs (loopbacks, point-to-point links): /32 বা /30, /31
- Network IPs (management, LANs): /24, /25, /26 ইত্যাদি

#### Mistake 4: কেবল কালার কোডিং না মানা

**ভুল:**

প্রথমে সব কেবল এ র‍্যান্ডম কালার দেওয়া হয়েছিল। ফলে ফিজিকাল র্যাক এ গিয়ে কোন কেবল কী বোঝা যাচ্ছিল না।

**সমাধান:**

একটা কালার কোডিং স্কিম বানানো হয়েছে:
- Blue: Fiber cables
- Yellow: Cat6 ethernet (access to distribution)
- Green: Cat6 ethernet (customer connections)
- Red: Critical uplink/backbone

নটবটে সব কেবল আপডেট করা হয়েছে এই কালার স্কিম অনুযায়ী।

**শিক্ষা:** একটা কনসিসটেন্ট নেমিং এবং কালার স্কিম ফলো করুন।

#### Mistake 5: ডিসক্রিপশন ফিল্ড খালি রাখা

**ভুল:**

প্রথম দিকে শুধু ডিভাইস নেম দেওয়া হয়েছিল, ডিসক্রিপশন খালি ছিল।

```
Name: SW-DN-MIR-ACC-03
Description: (empty)
```

ছয় মাস পরে যখন দেখা গেল, কেউ বুঝতে পারছিল না এই সুইচ কোন বিল্ডিং এ, কী পারপাস এ।

**সমাধান:**

সব ডিভাইস এ প্রপার ডিসক্রিপশন যোগ করা হয়েছে:

```
Name: SW-DN-MIR-ACC-03
Description: Building C access switch serving 280 residential customers in Mirpur Sector 12
```

**শিক্ষা:** ডিসক্রিপশন এবং কমেন্টস ফিল্ড সবসময় পূরণ করুন। ভবিষ্যতে নিজেই নিজেকে ধন্যবাদ দেবেন।

#### Mistake 6: স্ট্যাটাস সঠিকভাবে সেট না করা

**ভুল:**

একটা ওল্ড সুইচ যেটা আর ব্যবহার হয় না, সেটার স্ট্যাটাস ছিল "Active"।

**সমস্যা:**

রিপোর্ট এ এটা কাউন্ট হচ্ছিল। ফলে অ্যাকচুয়াল অ্যাক্টিভ ডিভাইসেস কাউন্ট ভুল আসছিল।

**সমাধান:**

ডিভাইস স্ট্যাটাস "Decommissioned" করা হয়েছে। অথবা একদম ডিলিট করে দেওয়া হয়েছে।

**শিক্ষা:** ডিভাইস লাইফসাইকেল মেইনটেইন করুন - Active, Offline, Decommissioned, Retired ইত্যাদি সঠিক status দিন।

---

### নেমিং কনভেনশন স্ট্যান্ডার্ড

নির্ভর কমিউনিকেশন একটা ডকুমেন্টেড নেমিং কনভেনশন তৈরি করেছে। এটা একটা Word ডকুমেন্টে রাখা আছে এবং সব টিম মেম্বার কে দেওয়া হয়েছে।

```
========================================
Nirvor Communication Naming Convention
Version 1.0 - Updated: February 2025
========================================

1. DEVICES
----------
Format: [Type]-[Zone]-[Site]-[Role]-[Number]

Type Codes:
  R  = Router
  SW = Switch
  FW = Firewall
  OLT = Optical Line Terminal

Zone Codes:
  DN = Dhaka North
  DS = Dhaka South (future)
  CT = Chittagong (future)

Site Codes (3 letters):
  MIR = Mirpur
  UTT = Uttara
  KAL = Kalyanpur (future)
  BAN = Banani (future)

Role Codes:
  CORE = Core
  DIST = Distribution
  ACC  = Access
  EDGE = Edge

Number: 01, 02, 03... (zero-padded, 2 digits)

Examples:
  R-DN-MIR-CORE-01
  SW-DN-UTT-DIST-01
  SW-DN-MIR-ACC-15

2. INTERFACES
-------------
Use manufacturer's default naming.
Add clear descriptions:

Format: [Purpose] - [Remote End/Details]

Examples:
  "Uplink to BTCL - 5 Gbps circuit"
  "Link to SW-DN-MIR-DIST-01 port sfp1"
  "Customer: ABC Corporation - VLAN 200"

3. CABLES
---------
Format: [Source Device Type]-TO-[Dest Device Type]-[Number]

Examples:
  CORE-TO-DIST-01
  DIST-TO-ACC-03
  UPLINK-BTCL-01

4. IP ADDRESSES
---------------
Management IPs:
  Core Routers: 10.10.X.1/24
  Dist Switches: 10.10.X.11-20/24
  Access Switches: 10.10.X.21-99/24

DNS Naming:
  Format: [device-name].nirvor.bd
  Example: r-mir-core-01.nirvor.bd

5. VLANs
--------
Format: [Purpose]_[Site]

Examples:
  MGMT_MIRPUR
  RESIDENTIAL_MIR
  CORPORATE (global, no site suffix)

6. LOCATIONS
------------
Use full descriptive names:
  Dhaka North Zone
  Mirpur POP
  Rack A

Avoid abbreviations in location names.

========================================
```

এই ডকুমেন্ট টা প্রিন্ট করে এনওসি তে টাঙিয়ে রাখা আছে। নতুন কেউ জয়েন করলে তাকে এটা পড়তে দেওয়া হয়।

---

### Export এবং Backup - ডেটা সংরক্ষণ

কিছু ডেটা এন্ট্রি হয়ে গেলে ব্যাকআপ নেওয়া জরুরি।

#### CSV Export

যেকোনো অবজেক্ট লিস্ট থেকে সিএসভি এক্সপোর্ট করা যায়।

**Devices → Devices** লিস্টে যান।

উপরে ডান দিকে **Export** বাটন দেখবেন। ক্লিক করলে একটা ড্রপডাউন:

- **Export All**: সব ডিভাই
- **Export Selected**: যেগুলো চেকবক্স দিয়ে সিলেক্ট করেছেন
- **Export as CSV**: CSV format
- **Export as YAML**: YAML format

**Export All → CSV** সিলেক্ট করুন।

একটা সিএসভি ফাইল ডাউনলোড হবে যেখানে সব ডিভাই এর সব ইনফরমেশন থাকবে।

এই ফাইল একটা সেফ জায়গায় রাখুন। যদি কখনো ভুল করে কিছু ডিলিট হয়ে যায়, এই সিএসভি দিয়ে ইমপোর্ট করে রিকভার করতে পারবেন।

#### মান্থলি ডাটা স্ন্যাপশট

নির্ভর কমিউনিকেশন প্রতি মাসের ১ তারিখে একটা স্ন্যাপশট নেয়:

```bash
# Create backup directory
mkdir -p /backup/nautobot/monthly/2025-02

# Export all key data types
# Devices
curl -H "Authorization: Token YOUR_TOKEN" \
  "https://nautobot.nirvor.bd/api/dcim/devices/?limit=0&format=csv" \
  > /backup/nautobot/monthly/2025-02/devices.csv

# IP Addresses
curl -H "Authorization: Token YOUR_TOKEN" \
  "https://nautobot.nirvor.bd/api/ipam/ip-addresses/?limit=0&format=csv" \
  > /backup/nautobot/monthly/2025-02/ip-addresses.csv

# (similarly for other object types)
```

এগুলো Google Drive এ ব্যাকআপ করা হয়।

---

### চ্যাপ্টার সারাংশ

এই চ্যাপ্টারে আমরা শিখলাম:

**Location Hierarchy:**
- Location Types তৈরি করা (Zone, Cluster, POP, Rack)
- Actual Locations তৈরি করা
- Hierarchy সঠিকভাবে সেটআপ করা

**Roles Setup:**
- Unified Roles সিস্টেম (Nautobot 3.0)
- Content Types দিয়ে specify করা
- Device roles তৈরি করা

**Device Management:**
- Manufacturers যোগ করা
- Device Types তৈরি করা
- Interface Templates এর গুরুত্ব
- Actual Devices এন্ট্রি করা
- Racks এ device position করা

**IP এবং VLAN:**
- IP Namespaces
- Prefixes (hierarchical)
- IP Addresses assign করা
- VLANs তৈরি করা
- VLAN Groups

**Physical Connectivity:**
- Cables তৈরি করা
- Cable Trace ব্যবহার করা
- Color coding

**Upstream Connectivity:**
- Providers
- Circuit Types
- Circuits
- Circuit Terminations

**Data Quality:**
- Validation checks
- Common mistakes এবং সমাধান
- Naming conventions
- Export/Backup

নির্ভর কমিউনিকেশন এখন তাদের মিরপুর এবং উত্তরা পপের প্রায় সম্পূর্ণ নেটওয়ার্ক নটবটে ডকুমেন্ট করে ফেলেছে। পরের চ্যাপ্টারে আমরা দেখব কীভাবে এই সেটআপ কে স্কেল করা যায় যখন নতুন পপ যুক্ত হয়, আরো ডিভাইস আসে, এবং কীভাবে অ্যাডভান্সড ফিচার ব্যবহার করতে হয়।
