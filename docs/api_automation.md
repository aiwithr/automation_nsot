আগের তেরোটা চ্যাপ্টারে আমরা নটবটের ওয়েব ইন্টারফেস দিয়ে সব কাজ করেছি। সাইট তৈরি করেছি, ডিভাইস এড করেছি, আইপি ম্যানেজ করেছি - সবকিছু ম্যানুয়ালি। কিন্তু এখন সময় এসেছে পরের লেভেলে যাওয়ার। 

যখন আপনার নেটওয়ার্ক বড় হতে থাকে, তখন ম্যানুয়াল কাজ করা সম্ভব না। কল্পনা করুন - প্রতিদিন ২০০টা নতুন কাস্টমার যুক্ত হচ্ছে। প্রতিটার জন্য আইপি অ্যালোকেট করতে হবে, ডকুমেন্ট করতে হবে। UI থেকে একটা একটা করে করলে পুরো দিন চলে যাবে। এজন্যই দরকার অটোমেশন। আর অটোমেশনের চাবি হলো API।

API কী জিনিস? সহজ ভাষায় বলতে গেলে, API (Application Programming Interface) হলো একটা সিস্টেমের সাথে অন্য সিস্টেম বা প্রোগ্রাম কথা বলার মাধ্যম। আপনি যেমন নটবটের ওয়েব ইন্টারফেসে ক্লিক করে কাজ করেন, API দিয়ে একই কাজ করা যায় কোড লিখে। ধরুন আপনি একটা Python স্ক্রিপ্ট লিখলেন যেটা নটবটকে বলল "Mirpur POP-এর সব ডিভাইসের লিস্ট দাও"। নটবট তখন API-এর মাধ্যমে সেই তথ্য পাঠিয়ে দেবে। আমাদের আগের বই "বাংলাদেশি আইএসপি এবং নেটওয়ার্ক অটোমেশন" বইতে REST API সম্পর্কে বিস্তারিত আলোচনা আছে - HTTP methods, JSON, authentication এসব নিয়ে। এই বইয়ে আমরা সরাসরি PyNautobot লাইব্রেরি ব্যবহার করব, কারণ এটা API ইউজ করাকে অনেক সহজ করে দেয়। আপনাকে REST API-এর সব টেকনিক্যাল ডিটেইলস জানতে হবে না - PyNautobot ব্যাকগ্রাউন্ডে সব সামলাবে। আগের বই "নেটওয়ার্ক এবং আইএসপি অটোমেশন" এ দুটো চ্যাপ্টার আছে এটা নিয়ে।

এই চ্যাপ্টারে আমরা দেখব কীভাবে Python দিয়ে নটবটের সাথে কথা বলতে হয়, কীভাবে ডেটা পড়তে হয়, কীভাবে নতুন জিনিস যোগ করতে হয়। শুরু করি PyNautobot দিয়ে।

## PyNautobot দিয়ে অটোমেশন

নির্ভর কমিউনিকেশন এখন ৫০ হাজার কাস্টমারে পৌঁছেছে। নটবটের ওয়েব UI দিয়ে সব কাজ করছে - ডিভাইস যোগ করা, আইপি এসাইন করা, রিপোর্ট দেখা। কিন্তু একদিন জাহাঙ্গীর সাহেব একটা সমস্যার মুখোমুখি হলেন।

গত সপ্তাহে নতুন একটা বিল্ডিংয়ে ২০টা এক্সেস সুইচ ইনস্টল করা হয়েছে। প্রতিটা সুইচের জন্য:
- নটবটে ডিভাইস এন্ট্রি করতে হবে
- ম্যানেজমেন্ট আইপি এসাইন করতে হবে

আসিফ UI থেকে একটা একটা করে করছিল। একটা ডিভাইস এন্ট্রি করতে ৫-৭ মিনিট লাগছে। ২০টায় প্রায় ২ ঘণ্টা!

জাহাঙ্গীর সাহেব বললেন, "এভাবে চলবে না। আমাদের অটোমেশন দরকার। Python স্ক্রিপ্ট লিখে নটবট API দিয়ে কাজ করা যায়।"

### PyNautobot - Python থেকে Nautobot

PyNautobot হলো একটা Python লাইব্রেরি যেটা নটবট API এর সাথে কাজ করা সহজ করে দেয়।

### Python Environment সেটআপ

**Windows-এ:**
১. python.org থেকে Python ডাউনলোড করুন (৩.১১+)
২. ইনস্টল করার সময় "Add Python to PATH" চেক করুন
৩. Command Prompt এ চেক করুন: `python --version`

**PyNautobot ইনস্টল:**

```bash
pip install pynautobot
```

### API Token তৈরি করা

Nautobot → Profile → API Tokens → + Add Token

```
Description: Python Scripts
Write Enabled: ✓
```

টোকেন কপি করুন: `f88c10c67357174593e09b189a57852a078bba06`

⚠️ এটা পাসওয়ার্ডের মতো - কাউকে দেবেন না!

### প্রথম Python স্ক্রিপ্ট

`hello_nautobot.py` ফাইল তৈরি করুন:

```python
from pynautobot import api

## Nautobot connection
nb = api(
    url="https://nautobot.nirvor.bd",
    token="your-api-token"
)

## Test connection
print(f"Connected to Nautobot: {nb.version}")
```

চালান:

```bash
python hello_nautobot.py
```

আউটপুট: `Connected to Nautobot: 3.0.0`

### ডেটা রিড করা - সব ডিভাইস দেখা

`list_devices.py`:

```python
from pynautobot import api

nb = api(url="https://nautobot.nirvor.bd", token="your-api-token")

## সব ডিভাইস নিয়ে আসুন
devices = nb.dcim.devices.all()

print(f"Total devices: {len(devices)}\n")

## প্রতিটা ডিভাইসের নাম এবং লোকেশন
for device in devices[:10]:  ## প্রথম ১০টা
    print(f"- {device.name} at {device.location}")
```

আউটপুট:

```
Total devices: 152

- R-DN-MIR-CORE-01 at Mirpur POP
- R-DN-UTT-CORE-01 at Uttara POP
- SW-DN-GUL-DIST-01 at Gulshan POP
...
```

### ফিল্টার করা - নির্দিষ্ট ডিভাইস খোঁজা

```python
from pynautobot import api

## Nautobot connection
nb = api(url="https://nautobot.nirvor.bd",token="your-api-token")
## শুধু Mirpur POP এর ডিভাইস
mirpur_devices = nb.dcim.devices.filter(location="Mirpur POP")
print(f"Mirpur devices: {len(mirpur_devices)}")

## শুধু Core Routers
core_routers = nb.dcim.devices.filter(role="Core Router")
print(f"Core Routers: {len(core_routers)}")

## একাধিক ফিল্টার
active_switches = nb.dcim.devices.filter(
    role="Access Switch",
    status="active",
    location="Gulshan POP"
)
```

### আইপি ইউটিলাইজেশন রিপোর্ট

জাহাঙ্গীর সাহেব প্রতি সপ্তাহে জানতে চান কোন প্রিফিক্স প্রায় ফুল।

`ip_utilization.py`:

```python
from pynautobot import api

nb = api(url="https://nautobot.nirvor.bd", token="your-api-token")

prefixes = nb.ipam.prefixes.all()

print("IP Utilization Report")
print("=" * 60)

for prefix in prefixes:
    if "/24" in str(prefix.prefix):  ## শুধু /24 দেখাই
        util = prefix.utilization if hasattr(prefix, 'utilization') else 0
        
        print(f"\nPrefix: {prefix.prefix}")
        print(f"  Location: {prefix.location}")
        print(f"  Utilization: {util}%")
        
        if util > 80:
            print(f"  ⚠️  WARNING: High utilization!")
        elif util > 60:
            print(f"  ⚠️  NOTICE: Approaching capacity.")
```

আউটপুট:

```
IP Utilization Report
============================================================

Prefix: 103.125.40.0/24
  Location: Mirpur POP
  Utilization: 92%
  ⚠️  WARNING: High utilization!

Prefix: 103.125.48.0/24
  Location: Gulshan POP
  Utilization: 45%
```

### নতুন ডিভাইস যোগ করা

একটা ডিভাইস:

```python
from pynautobot import api

nb = api(url="https://nautobot.nirvor.bd", token="your-api-token")

new_device = nb.dcim.devices.create(
    name="SW-DN-GUL-ACC-26",
    device_type="TP-Link TL-SG3428MP",
    role="Access Switch",
    location="Gulshan POP",
    status="active",
    serial="TPLACC026NEW",
    comments="Added via Python script"
)

print(f"✓ Created: {new_device.name}")
```

### বাল্ক ডিভাইস তৈরি - আসিফের সমস্যা সমাধান

CSV ফাইল থেকে:

`new_devices.csv`:

```csv
name,serial,building
SW-DN-GUL-ACC-27,TPLACC027,Building F
SW-DN-GUL-ACC-28,TPLACC028,Building F
SW-DN-GUL-ACC-29,TPLACC029,Building G
```

`import_devices.py`:

```python
import csv
from pynautobot import api

nb = api(url="https://nautobot.nirvor.bd", token="your-api-token")

with open('new_devices.csv', 'r') as f:
    reader = csv.DictReader(f)
    
    for row in reader:
        try:
            device = nb.dcim.devices.create(
                name=row['name'],
                device_type="TP-Link TL-SG3428MP",
                role="Access Switch",
                location="Gulshan POP",
                status="active",
                serial=row['serial'],
                comments=f"Auto-added - {row['building']}"
            )
            print(f"✓ Created: {device.name}")
        except Exception as e:
            print(f"✗ Failed: {row['name']} - {e}")
```

২০টা ডিভাইস ২ মিনিটে তৈরি! ২ ঘন্টার কাজ ২ মিনিটে শেষ।

### IP Address Assignment

নতুন ডিভাইসে আইপি এসাইন করা:

```python
from pynautobot import api

nb = api(url="https://nautobot.nirvor.bd", token="your-api-token")

## ডিভাইস খুঁজুন
device = nb.dcim.devices.get(name="SW-DN-GUL-ACC-26")

## Interface খুঁজুন (বা তৈরি করুন)
## প্রথমে দেখুন vlan10 interface আছে কিনা
interfaces = nb.dcim.interfaces.filter(device_id=device.id, name="vlan10")

if not interfaces:
    ## নেই তো তৈরি করুন
    interface = nb.dcim.interfaces.create(
        device=device.id,
        name="vlan10",
        type="virtual",
        enabled=True,
        description="Management interface"
    )
    print(f"✓ Created interface: vlan10")
else:
    interface = interfaces[0]

## এখন IP assign করুন
new_ip = nb.ipam.ip_addresses.create(
    address="10.10.12.26/24",  ## Gulshan POP management network
    status="active",
    assigned_object_type="dcim.interface",
    assigned_object_id=interface.id,
    dns_name="sw-gul-acc-26.nirvor.bd",
    description="Management IP"
)

print(f"✓ IP assigned: {new_ip.address}")
```

### ডেটা আপডেট করা

বিদ্যমান ডিভাইস পরিবর্তন করা:

```python
from pynautobot import api

nb = api(url="https://nautobot.nirvor.bd", token="your-api-token")

## ডিভাইস খুঁজুন
device = nb.dcim.devices.get(name="SW-DN-GUL-ACC-26")

## Status আপডেট করুন
device.status = "offline"
device.comments = "Under maintenance - scheduled completion: 2027-02-15"
device.save()

print(f"✓ Updated: {device.name}")
```

### বাল্ক আপডেট - সব গুলশান পপ ডিভাইসে ট্যাগ যোগ

```python
from pynautobot import api

nb = api(url="https://nautobot.nirvor.bd", token="your-api-token")

## প্রথমে ট্যাগ খুঁজুন (বা তৈরি করুন)
try:
    production_tag = nb.extras.tags.get(name="production")
except:
    production_tag = nb.extras.tags.create(
        name="production",
        color="4caf50",
        description="Production equipment"
    )

## গুলশান পপের সব ডিভাইস
gulshan_devices = nb.dcim.devices.filter(location="Gulshan POP")

for device in gulshan_devices:
    ## Check if tag already exists
    tag_names = [tag.name for tag in device.tags]
    if "production" not in tag_names:
        device.tags.append(production_tag)
        device.save()
        print(f"✓ Tagged: {device.name}")
```

### ডেইলি রিপোর্ট স্ক্রিপ্ট

আসিফ প্রতিদিন সকালে একটা রিপোর্ট চায়। এখন অটোমেট করা যাক।

`daily_report.py`:

```python
from pynautobot import api
from datetime import datetime

nb = api(url="https://nautobot.nirvor.bd", token="your-api-token")

print("=" * 70)
print(f"Nirvor Communication Bangladesh - Daily Network Report")
print(f"Generated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
print("=" * 70)

## 1. Overall Statistics
print("\n OVERALL STATISTICS")
print("-" * 70)

total_devices = nb.dcim.devices.count()
active_status = nb.extras.statuses.get(
    name="active",
    content_types="dcim.device"
)
active_devices = nb.dcim.devices.filter(status_id=active_status.id).count()

offline_status = nb.extras.statuses.get(
    name="offline",
    content_types="dcim.device"
)
offline_devices = nb.dcim.devices.filter(status_id=offline_status.id).count()

print(f"Total Devices: {total_devices}")
print(f"  ├─ Active: {active_devices}")
print(f"  └─ Offline: {offline_devices}")

## 2. Per-Location Breakdown
print("\n LOCATION BREAKDOWN")
print("-" * 70)

locations = nb.dcim.locations.filter(location_type="POP")
for location in locations:
    device_count = nb.dcim.devices.filter(location=location.name).count()
    print(f"{location.name}: {device_count} devices")

## 3. IP Utilization Alerts
print("\n  IP UTILIZATION ALERTS")
print("-" * 70)

prefixes = nb.ipam.prefixes.all()
high_util = []

for prefix in prefixes:
    if hasattr(prefix, 'utilization') and prefix.utilization > 80:
        high_util.append({
            'prefix': str(prefix.prefix),
            'location': str(prefix.location),
            'utilization': prefix.utilization
        })

if high_util:
    for item in high_util:
        print(f" {item['prefix']} at {item['location']}: {item['utilization']}%")
else:
    print("✓ All prefixes have healthy utilization")

## 4. Devices Under Maintenance
print("\n🔧 MAINTENANCE STATUS")
print("-" * 70)

maintenance_devices = nb.dcim.devices.filter(tags="maintenance")
if maintenance_devices:
    print(f" {len(maintenance_devices)} devices under maintenance:")
    for device in maintenance_devices:
        print(f"   - {device.name}")
else:
    print("✓ No devices under maintenance")

## 5. Data Quality Issues
print("\n DATA QUALITY CHECK")
print("-" * 70)

no_serial = nb.dcim.devices.filter(status="active", serial="")
if no_serial:
    print(f" {len(no_serial)} active devices missing serial numbers")
else:
    print("✓ All active devices have serial numbers")

print("\n" + "=" * 70)
print("Report completed successfully")
print("=" * 70)
```

আউটপুট:

```
======================================================================
Nirvor Communication - Daily Network Report
Generated: 2026-02-19 22:32:00
======================================================================

OVERALL STATISTICS
----------------------------------------------------------------------
Total Devices: 152
  ├─ Active: 148
  └─ Offline: 4

LOCATION BREAKDOWN
----------------------------------------------------------------------
Mirpur POP: 28 devices
Uttara POP: 22 devices
Gulshan POP: 24 devices
Banani POP: 18 devices
...

  IP UTILIZATION ALERTS
----------------------------------------------------------------------
  103.125.40.0/24 at Mirpur POP: 92%
  103.125.48.0/24 at Gulshan POP: 84%

 MAINTENANCE STATUS
----------------------------------------------------------------------
✓ No devices under maintenance

 DATA QUALITY CHECK
----------------------------------------------------------------------
  3 active devices missing serial numbers

======================================================================
Report completed successfully
======================================================================
```

##### রিপোর্ট অটোমেট করা

**Windows Task Scheduler:**

১. Task Scheduler ওপেন করুন
২. Create Basic Task → Name: "Nautobot Daily Report"
৩. Trigger: Daily, 8:00 AM
৪. Action: Start a program
   - Program: `python`
   - Arguments: `C:\nautobot-scripts\daily_report.py`
৫. Finish

**Linux Cron:**

```bash
## crontab -e
0 8 * * * /usr/bin/python3 /home/nirvor/scripts/daily_report.py > /home/nirvor/reports/daily_$(date +\%Y\%m\%d).txt
```

এখন প্রতিদিন সকাল ৮টায় অটোমেটিক রিপোর্ট জেনারেট হবে।

### Error Handling - ভুল সামলানো

স্ক্রিপ্ট লেখার সময় Error Handling করা জরুরি।

```python
from pynautobot import api

nb = api(url="https://nautobot.nirvor.bd", token="your-api-token")

## নিরাপদ উপায়ে ডিভাইস খুঁজুন
try:
    device = nb.dcim.devices.get(name="SW-DN-MIR-ACC-99")
    print(f"Found: {device.name}")
except ValueError:
    print("Device not found")
except Exception as e:
    print(f"Error: {e}")
```

### Best Practices

**১. Token সিকিউরিটি - হার্ডকোড করবেন না:**

```python
##  খারাপ
token = "abc123..."

##  ভালো - Environment variable
import os
token = os.environ.get('NAUTOBOT_TOKEN')

## অথবা config ফাইল
import configparser
config = configparser.ConfigParser()
config.read('config.ini')
token = config['nautobot']['token']
```

**২. Logging যোগ করুন:**

```python
import logging

logging.basicConfig(
    filename='nautobot_script.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

logging.info("Script started")
devices = nb.dcim.devices.all()
logging.info(f"Retrieved {len(devices)} devices") 
```

**৩. Rate Limiting - একবারে হাজার হাজার রিকোয়েস্ট পাঠাবেন না:** 

```python
import time

for device in large_list:
    process_device(device)
    time.sleep(0.1)  ## ১০০ms delay
```

নির্ভর কমিউনিকেশন এখন নটবটকে অটোমেট করতে পারছে। আসিফ আর ম্যানুয়ালি ক্লিক করে সময় নষ্ট করে না। একটা স্ক্রিপ্ট লিখে সে ৫০টা ডিভাইস ৫ মিনিটে যোগ করতে পারে।

পরবর্তী স্টেপ হলো আরো এডভান্সড অটোমেশন - Ansible integration, Golden Config management, এবং ১ লক্ষ কাস্টমারের দিকে যাত্রা।

---

