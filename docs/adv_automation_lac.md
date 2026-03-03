নির্ভর কমিউনিকেশন এখন ৫০ হাজার কাস্টমারে স্ট্যাবল। নটবট দিয়ে নেটওয়ার্ক ডকুমেন্ট করছে, পাইনটবট দিয়ে বেসিক অটোমেশন করছে। কিন্তু তাদের লক্ষ্য আরো বড় - পরের দুই বছরে ১ লক্ষ কাস্টমার।

জাহাঙ্গীর সাহেব একটা মিটিং ডাকলেন। তিনি বললেন, "আমরা এখন যে পদ্ধতিতে কাজ করছি সেটা ৫০ হাজারে চলছে। কিন্তু ১ লক্ষে গেলে আরো বেশি অটোমেশন দরকার।"

তিনি একটা হোয়াইটবোর্ড এ লিখলেন:

```
বর্তমান সমস্যা:
১. নতুন ডিভাইস অ্যাড করার পরে ম্যানুয়ালি কনফিগার করতে হয়
২. প্রতিটা ডিভাইসের কনফিগ ব্যাকআপ নেই
৩. কনফিগ ড্রিফট ট্র্যাক করতে পারি না (কে কখন কী চেঞ্জ করল)
৪. নতুন পপ যুক্ত করতে ২-৩ দিন সময় লাগে

সমাধান:
১. আনসিবল দিয়ে কনফিগ অটোমেশন
২. গোল্ডেন কনফিগ ম্যানেজমেন্ট
৩. মনিটরিং ইন্টিগ্রেশন
৪. পুরো প্রসেস স্ট্রিমলাইন করা
```

এই চ্যাপ্টারে আমরা দেখব কীভাবে এডভান্সড অটোমেশন করতে হয়।

## আনসিবল - কনফিগ অটোমেশনের রাজা

আনসিবল হলো একটা অটোমেশন টুল যা দিয়ে নেটওয়ার্ক ডিভাইস কনফিগার করা যায়। এটার সাথে নটবট integration করলে পাওয়ারফুল কম্বিনেশন হয়।

### আনসিবল কী করে?

সহজ ভাষায়: আনসিবল একটা প্লেবুক (স্ক্রিপ্ট) পড়ে, তারপর অনেকগুলো ডিভাইসে একসাথে কমান্ডস এক্সিকিউট করে।

উদাহরণ: "সব কোর রাউটারে এনটিপি সার্ভার কনফিগার করো"

আনসিবল প্লেবুক লিখবেন একবার। তারপর ১০টা রাউটার হোক বা ১০০টা - একই প্লেবুক চালাবেন।

### আনসিবল ইনস্টল করা

```bash
# Ubuntu/Linux
sudo apt install ansible

# অথবা pip দিয়ে
pip install ansible

# Check করুন
ansible --version
```

### Nautobot Inventory প্লাগইন

Ansible কে বলতে হবে কোন ডিভাইসগুলোতে কাজ করবে। নটবটে থেকে সেই লিস্ট নেওয়া যায়।

একটা ইনভেন্টরি ফাইল  তৈরি করুন: `nautobot_inventory.yml`

```yaml
plugin: networktocode.nautobot.inventory
api_endpoint: https://nautobot.nirvor.bd
token: your-api-token-here
validate_certs: false

# কোন ডিভাইস নেবো
query_filters:
  - status: active

# ডিভাইস গ্রুপ করার নিয়ম
group_by:
  - role
  - location
```

এখন আনসিবল এর কাছে নটবটে যত ডিভাইস আছে সব দেখতে পাবে:

```bash
ansible-inventory -i nautobot_inventory.yml --list
```

### প্রথম আনসিবল প্লেবুক - এনটিপি কনফিগার করা

সব রাউটারে এনটিপি সার্ভার সেট করব।

`configure_ntp.yml`:

```yaml
---
- name: Configure NTP on all routers
  hosts: core_router  # নটবট থেকে "Core Router" role এর ডিভাইস
  gather_facts: no
  
  vars:
    ntp_server: 103.108.140.150  # BDIX NTP server
  
  tasks:
    - name: Configure NTP server
      ansible.netcommon.cli_config:
        config: |
          /system ntp client set enabled=yes
          /system ntp client set server-dns-names={{ ntp_server }}
```

চালান:

```bash
ansible-playbook -i nautobot_inventory.yml configure_ntp.yml
```

আনসিবল এখন নটবট থেকে সব কোর রাউটারের লিস্ট নেবে, তারপর প্রতিটাতে এনটিপি কনফিগার করবে।

### আরেকটা উদাহরণ - SNMP Community সেট করা

`configure_snmp.yml`:

```yaml
---
- name: Configure SNMP on all devices
  hosts: all
  gather_facts: no
  
  vars:
    snmp_community: Nirvor_R3ad0nly
  
  tasks:
    - name: Set SNMP community
      ansible.netcommon.cli_config:
        config: |
          /snmp community add name={{ snmp_community }} addresses=192.168.10.0/24
          /snmp set enabled=yes
```

একবার প্লেবুক লিখলে ১৫০টা ডিভাইসে একসাথে apply করা যায়। ম্যানুয়ালি করলে দিনের পর দিন লাগত!

### নটবট + আনসিবল ওয়ার্কফ্লো

```
১. নটবটে নতুন ডিভাইস অ্যাড করুন
২. আনসিবল প্লেবুক চালান
৩. ডিভাইস অটোমেটিক্যালি কনফিগার হয়ে যাবে
```

এভাবে আসিফ এখন নতুন সুইচ যোগ করে আনসিবল প্লেবুক চালায়। ৫ মিনিটে সব কনফিগ হয়ে যায়!

---

## গোল্ডেন কনফিগ - কনফিগ ম্যানেজমেন্ট

গোল্ডেন কনফিগ মানে হলো "আদর্শ কনফিগারেশন"। প্রতিটা ডিভাইসের কী কনফিগ হওয়া উচিত সেটা ডিফাইন করা, তারপর actual কনফিগ চেক করা।

### সমস্যা যেটা সলভ করে

**সিনারিও:** গত সপ্তাহে একটা কোর রাউটারে ইন্টারনেট স্লো হচ্ছিল। দেখা গেল কেউ QoS কনফিগ চেঞ্জ করে দিয়েছে। কে করেছে? কখন করেছে? কেন করেছে? কিছু জানা নেই!

গোল্ডেন কনফিগ এই সমস্যা সলভ করে:

- প্রতিটা ডিভাইসের কনফিগ ব্যাকআপ রাখে
- কনফিগ চেঞ্জ ট্র্যাক করে
- "কাঙ্ক্ষিত কনফিগ" থেকে বিচ্যুত হলে alert দেয়

### নটবট গোল্ডেন কনফিগ প্লাগইন

নটবটের একটা প্লাগইন আছে - `nautobot-golden-config`।

**ইনস্টল করা:**

```bash
pip install nautobot-plugin-golden-config
```

নটবটের `nautobot_config.py` ফাইলে যোগ করুন:

```python
PLUGINS = [
    "nautobot_golden_config",
]

PLUGINS_CONFIG = {
    "nautobot_golden_config": {
        "enable_backup": True,
        "enable_compliance": True,
        "enable_intended": True,
    }
}
```

নটবট রিস্টার্ট করুন।

### গোল্ডেন কনফিগ সেটআপ

এখন নটবট UI তে **গোল্ডেন কনফিগ** মেনু দেখবেন।

**১. কনফিগ ব্যাকআপ  সেটআপ:**

গোল্ডেন কনফিগ → Settings → Backup

```
Backup Repository: Git repository যেখানে কনফিগ সেভ হবে
Backup Path Template: কনফিগs/{{ obj.location.name }}/{{ obj.name }}.cfg
```

**২. ডেইলি ব্যাকআপ জব চালান:**

গোল্ডেন কনফিগ → জবs → কনফিগ ব্যাকআপ 

এটা চালালে সব ডিভাইসের কনফিগ ব্যাকআপ নেবে এবং Git এ সেভ করবে।

**৩. কমপ্লায়েন্স চেক:**

একটা "ইন্টেন্ডেড কনফিগ" template তৈরি করুন যেটা বলে দেয় ডিভাইসে কী কনফিগ থাকা উচিত।

`templates/mikrotik_core_router.j2`:

```jinja2
# NTP Configuration
/system ntp client set enabled=yes server-dns-names=103.108.140.150

# SNMP Configuration  
/snmp community add name=Nirvor_R3ad0nly addresses=192.168.10.0/24
/snmp set enabled=yes contact="NOC Team" location="{{ device.location }}"

# DNS Configuration
/ip dns set servers=8.8.8.8,8.8.4.4

# Logging
/system logging add topics=info action=remote remote=192.168.10.100
```

কমপ্লায়েন্স জব চালালে দেখাবে কোন ডিভাইস এই স্ট্যান্ডার্ড মেনে চলছে, কোনটা চলছে না।

**কমপ্লায়েন্স রিপোর্ট:**

```
R-DN-MIR-CORE-01: ✓ Compliant
R-DN-UTT-CORE-01: ✗ Non-Compliant
  - Missing: NTP configuration
  - Mismatch: SNMP community name different
R-DN-GUL-CORE-01: ✓ Compliant
```

এখন জাহাঙ্গীর সাহেব জানেন উত্তরা কোর রাউটারে কনফিগ ঠিক নেই। সেখানে কাজ করতে হবে।

### কনফিগ ড্রিফট ডিটেকশন

প্রতিদিন একবার backup নিন। তারপর দেখুন কোন কনফিগ চেঞ্জ হয়েছে কিনা।

**ড্রিফট রিপোর্ট:**

```
R-DN-MIR-CORE-01
  Changed: 2027-02-08 14:35:22
  Modified by: asif@192.168.10.25
  Changes:
    - Added: /ip firewall filter rule
    - Removed: Old ACL rule
```

এখন জানা যাচ্ছে আসিফ গতকাল দুপুর ২:৩৫ এ একটা ফায়ারওয়াল রুল যোগ করেছে। যদি কোনো সমস্যা হয়, তাহলে আগের কনফিগ restore করা যাবে।

---

## মনিটরিং ইন্টিগ্রেশন - নটবট + প্রমিথিউস/গ্রাফানা

নটবট শুধু ডকুমেন্টেশন না - এটা মনিটরিং এর সাথেও ইন্টিগ্রেট করা যায়।

### ধারণা

নটবট থেকে ডিভাইস লিস্ট নিয়ে প্রমিথিউস/জ্যাবিক্স/লিব্রে এনএমএস এ অটোমেটিকালি অ্যাড করা যায়।

**Python স্ক্রিপ্ট:**

`sync_to_prometheus.py`:

```python
from pynautobot import api
import yaml

nb = api(url="https://nautobot.nirvor.bd", token="your-token")

# সব অ্যাক্টিভ ডিভাইস নিয়ে আসুন
devices = nb.dcim.devices.filter(status="active")

# প্রমিথিউস টার্গেটস তৈরি করুন
targets = []

for device in devices:
    # ম্যানেজমেন্ট আইপি খুঁজুন
    mgmt_ips = nb.ipam.ip_addresses.filter(device=device.name)
    
    if mgmt_ips:
        ip = str(mgmt_ips[0].address).split('/')[0]
        
        targets.append({
            'targets': [f"{ip}:161"],  # এসএনএমপি পোর্ট
            'labels': {
                'device': device.name,
                'location': str(device.location),
                'role': str(device.role)
            }
        })

# প্রমিথিউস কনফিগ ফাইলে লিখুন
with open('/etc/prometheus/targets/nautobot.yml', 'w') as f:
    yaml.dump(targets, f)

print(f"✓ Synced {len(targets)} devices to Prometheus")
```

এই স্ক্রিপ্ট চালালে নটবট থেকে সব ডিভাইস নিয়ে প্রমিথিউস এ অ্যাড করবে। প্রমিথিউস রিস্টার্ট করলেই মনিটরিং শুরু হবে।

**ক্রন দিয়ে অটোমেট:**

```bash
# প্রতি ঘন্টায় sync করুন
0 * * * * /usr/bin/python3 /opt/scripts/sync_to_prometheus.py
```

এখন নটবটে নতুন ডিভাইস অ্যাড করলে ১ ঘন্টার মধ্যে অটোমেটিকালি মনিটরিং এ যুক্ত হবে!

---

## স্কেলিং মাইলস্টোন - ১০k থেকে ১০০k এর যাত্রা

নির্ভর কমিউনিকেশনের জার্নি:

```
২০২৩: ৮k কাস্টমার - নটবট সেটআপ
২০২৫: ৩০k কাস্টমার - পাইনটবট অটোমেশন শুরু
২০২৭: ৫০k কাস্টমার - আনসিবল integration
২০২৯: ১০০k কাস্টমার (লক্ষ্য) - Full automation
```

### প্রতিটা স্তরে কী দরকার

**১০k - ৩০k কাস্টমার:**

- লোকেশন হায়ারার্কি ঠিক রাখা
- বেসিক পাইনটবট স্ক্রিপ্টস
- উইকলি ডাটা অডিট

**৩০k - ৫০k কাস্টমার:**

- আনসিবল প্লেবুকs
- গোল্ডেন কনফিগ শুরু করা
- মনিটরিং ইন্টিগ্রেশন

**৫০k - ১০০k কাস্টমার:**

- ফুল অটোমেশন পাইপলাইন
- সিআই/সিডি ফর নেটওয়ার্ক চেঞ্জেস
- এআই-অ্যাসিস্টেড ট্রাবলশুটিং (ভবিষ্যৎ)

### ১০০k এ পৌঁছানোর রোডম্যাপ

**বছর ১ (২০২৭-২০২৮):**

- আরো ১০টা পপ যোগ করা
- সব নতুন পপে day-1 automation
- গোল্ডেন কনফিগ সব ডিভাইসে enable করা

**বছর ২ (২০২৮-২০২৯):**

- নেটওয়ার্ক-অ্যাজ-কোড অ্যাপ্রোচ 
- সেলফ-সার্ভিস পোর্টাল (কাস্টমার নিজেই কিছু কাজ করতে পারবে)
- প্রেডিকটিভ মেইনটেন্যান্স (এআই দিয়ে সমস্যা হওয়ার আগেই ধরা)

### টেকনিক্যাল রিকোয়ারমেন্ট ১০০k এর জন্য

**ইনফ্রাস্ট্রাকচার:**

- নটবট সার্ভার: ১৬ জিবি র‍্যাম, ৮ কোরস
- পোস্টগ্রেস্কিউএল অপ্টিমাইজেশন
- রেডিস ক্যাশিং
- লোড ব্যালান্সিং (যদি অনেক ইউজার একসাথে access করে)

**টিম:**

- ৩-৪ জন নেটওয়ার্ক অটোমেশন ইঞ্জিনিয়ার
- ১০+ জন এনওসি
- ১৫+ জন ফিল্ড অপারেশনস

**প্রসেসস:**

- সব চেঞ্জ নটবট দিয়ে ট্র্যাক করতে হবে
- কোনো ম্যানুয়াল চেঞ্জ নয় (emergency ছাড়া)
- উইকলি অটোমেশন স্প্রিন্ট (নতুন অটোমেশন যোগ করা)

---

## এআই এবং এলএলএম ইন্টিগ্রেশন - ভবিষ্যৎ সম্ভাবনা

এটা একটু ফিউচারিস্টিক, কিন্তু সম্ভব।

### চ্যাটজিপিটি/ক্লড দিয়ে নেটওয়ার্ক অ্যাসিস্ট্যান্ট

একটা চ্যাটবট যেটা নটবট এপিআই অ্যাক্সেস করতে পারে:

**কথোপকথন:**

```
You: "Show me all devices in Gulshan POP"
Bot: *calls Nautobot API*
     "Found 24 devices in Gulshan POP:
      - 1 Core Router
      - 2 Distribution Switches  
      - 21 Access Switches"

You: "Which ones are offline?"
Bot: "2 devices are offline:
      - SW-DN-GUL-ACC-15 (offline since 2027-02-08)
      - SW-DN-GUL-ACC-22 (under maintenance)"

You: "Add a new access switch SW-DN-GUL-ACC-30"
Bot: *calls Nautobot API*
     "✓ Device created successfully!
      Do you want me to configure it with Ansible?"
```

এরকম চ্যাটবট তৈরি করা যায় ল্যাংচেইন দিয়ে। নটবট এআই কে ফাংশন কলিং এর মাধ্যমে ইন্টিগ্রেট করতে হবে।

### প্রেডিকটিভ মেইনটেন্যান্স

মেশিন লার্নিং দিয়ে প্রেডিক্ট করা:

- কোন ডিভাইস শীঘ্রই ফেইল করতে পারে (হিস্টোরিকাল ডাটা থেকে)
- কোন প্রিফিক্স শীঘ্রই ফুল হবে (গ্রোথ ট্রেন্ড থেকে)
- কোন লিংকে ব্যান্ডউইথ আপগ্রেড দরকার

এগুলো ২০২৮-২০২৯ এ আসবে যখন যথেষ্ট ডেটা জমা হবে।

---

## পারফরম্যান্স অ্যাট স্কেল - ১০০k এর জন্য প্রস্তুতি

### ডাটাবেস অপ্টিমাইজেশন

পোস্টগ্রেস্কিউএল টিউনিং:

```sql
# /etc/postgresql/16/main/postgresql.conf

shared_buffers = 8GB
effective_cache_size = 24GB
work_mem = 128MB
maintenance_work_mem = 2GB

max_connections = 200

# Indexing
CREATE INDEX idx_device_location ON dcim_device(location_id);
CREATE INDEX idx_ipaddress_device ON ipam_ipaddress(assigned_object_id);
```

### ক্যাশিং স্ট্র্যাটেজি

রেডিস ক্যাশিং এনেবল করুন:

```python
# nautobot_config.py

CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/0',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        },
        'TIMEOUT': 300,  # 5 minutes
    }
}
```

### এপিআই রেট লিমিটিং

অনেক স্ক্রিপ্টস একসাথে চললে এপিআই ওভারলোড হতে পারে। রেট লিমিটিং সেট করুন:

```python
# nautobot_config.py

REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/hour',
        'user': '1000/hour'
    }
}
```

নির্ভর কমিউনিকেশন এখন একটা সলিড অটোমেশন ফাউন্ডেশন পেয়েছে। তারা জানে কীভাবে ১ লক্ষ কাস্টমারে স্কেল করতে হবে। নটবট শুধু একটা ডকুমেন্টেশন টুল না - এটা তাদের পুরো নেটওয়ার্ক অপারেশনস এর হৃদয়।

এই বইয়ের শেষে এসে আমরা দেখলাম কীভাবে একটা ছোট আইএসপি (৫ হাজার কাস্টমার) একটা মিডিয়াম আইএসপি (৫০ হাজার) হয়ে বড় আইএসপি (১ লক্ষ+) এর দিকে যাত্রা করতে পারে। আর সেই যাত্রায় নেটওয়ার্ক সোর্স অফ ট্রুথ  (নটবট) হলো মূল ভিত্তি।

---
