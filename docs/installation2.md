নির্ভর কমিউনিকেশন আইএসপি হিসেবে বেশ নাম করেছে। ঢাকা নর্থ জোনে মূলতঃ তাদের নেটওয়ার্ক। বর্তমানে মিরপুর এবং উত্তরায় দুটো পপ, প্রায় ৫ হাজার কাস্টমার। তারা সিদ্ধান্ত নিয়েছে এক্সেল শিট/গুগল শিট এবং হাতে আঁকা ডায়াগ্রাম ছেড়ে একটা প্রপার নেটওয়ার্ক সোর্স অফ ট্রুথ সিস্টেম বানাবে। তাদের লক্ষ্য পরের দুই বছরে ৫০ হাজার কাস্টমারে পৌঁছানো, আর তারপর ১ লক্ষ। এজন্য দরকার একটা শক্ত ফাউন্ডেশন। সেই ফাউন্ডেশনের নাম Nautobot 3.0।

এই চ্যাপ্টারে আমরা দেখব কীভাবে নির্ভর কমিউনিকেশন তাদের নেটওয়ার্কের জন্য Nautobot 3.0 ইনস্টল এবং সেটআপ করেছে। আপনিও একইভাবে আপনার ISP-এর জন্য সেটআপ করতে পারবেন।

### কেন Nautobot 3.0?

প্রথম প্রশ্ন - কেন Nautobot 3.0? কেন Nautobot 2.x না?

**Nautobot 3.0-এর মূল সুবিধা:**

১. **ফ্লেক্সিবল লোকেশন হায়ারারকি (Flexible Location Hierarchy):** আগে সাইটস একটা ফ্ল্যাট স্ট্রাকচার ছিল। এখন লোকেশনস দিয়ে যত খুশি লেভেলের হায়ারারকি বানাতে পারবেন। Zone → Cluster → POP → Building → Floor - যেভাবে চান সাজান।

২. **ইউনিফাইড রোল সিস্টেম (Unified Role System):** ডিভাইস রোল, র‍্যাক রোল, আইপিএএম রোল - সব আলাদা আলাদা ছিল। এখন একটা ইউনিফাইড রোলস সিস্টেম। এক জায়গা থেকে সব ম্যানেজ করুন।

৩. **কাস্টম রিলেশনশিপ (Custom Relationships):** দুটো যেকোনো অবজেক্টের মধ্যে কাস্টম রিলেশনশিপ তৈরি করতে পারবেন। যেমন "ডিভাইস সাপোর্টস কাস্টমার" - এরকম রিলেশনশিপ।

৪. **বেটার পারফরম্যান্স (Better Performance):** ডেটাবেস অপ্টিমাইজেশন, ফাস্টার কোয়েরিস, ইমপ্রুভড ইউআই রেসপন্সিভনেস।

৫. **গ্রাফকিউএল এপিআই (GraphQL API):** রেস্ট এপিআই-এর পাশাপাশি গ্রাফকিউএল সাপোর্ট। জটিল কোয়েরি সহজে করা যায়।

৬. **প্লাগইন ইকোসিস্টেম (Plugin Ecosystem):** আরো শক্তিশালী প্লাগইন সিস্টেম। গোল্ডেন কনফিগ, ডিভাইস লাইফসাইকেল ম্যানেজমেন্ট - এসব প্লাগইন সহজে ইন্টিগ্রেট করা যায়।

৭. **মেজর ইউআই রিফ্রেশ (Major UI refresh):** নতুন Bootstrap 5 ভিত্তিক ইন্টারফেস, গ্লোবাল সার্চ, ফ্লাইআউট নেভিগেশন – ব্যবহার করা অনেক সহজ হয়েছে।

৮. **অ্যাপ্রুভাল ওয়ার্কফ্লো (Approval Workflows):** জব/চেঞ্জের জন্য মাল্টি-স্টেজ অ্যাপ্রুভাল সিস্টেম – টিমে কাজ করলে ভুল কমবে।

নির্ভর কমিউনিকেশনএই কারণে 3.0 বেছে নিয়েছে। তারা জানে ভবিষ্যতে স্কেলিং করতে হবে, তাই শুরু থেকেই সবচেয়ে আধুনিক ভার্সন দিয়ে শুরু করছে।

### সিস্টেম রিকোয়ারমেন্ট

নটবট চালাতে হলে একটা লিনাক্স সার্ভার দরকার। নির্ভর কমিউনিকেশনের ক্ষেত্রে তারা একটা ডেডিকেটেড উবুন্টু (Ubuntu) সার্ভার সেটআপ করেছে।

#### হার্ডওয়্যার রিকোয়ারমেন্ট (নির্ভর কমিউনিকেশনের সাইজের জন্য):

```
CPU: 4 cores (Intel/AMD)
RAM: 8 GB (minimum), 16 GB (recommended)
Storage: 100 GB SSD
Network: 1 Gbps ethernet
```

যদি আপনার নেটওয়ার্ক খুব ছোট হয় (১-২ হাজার কাস্টমার), তাহলে 2 cores এবং 4 GB RAM দিয়েও চলবে। কিন্তু ফিউচার-প্রুফিং-এর জন্য একটু বেশি নেওয়া ভালো।

#### সফটওয়্যার রিকোয়ারমেন্ট:

```
Operating System: Ubuntu 24.04 LTS (recommended)
                  অথবা Ubuntu 22.04 LTS
                  অথবা Debian 12

Python: 3.12 বা তার উপরে
PostgreSQL: 13 বা তার উপরে
Redis: 6.0 বা তার উপরে
```

নির্ভর কমিউনিকেশন Ubuntu 24.04 LTS বেছে নিয়েছে কারণ এটা লং-টার্ম সাপোর্ট পাবে ২০২৯ সাল পর্যন্ত।

### ইনস্টলেশন পদ্ধতি - কোনটা বেছে নেবেন?

নটবট ইনস্টল করার দুটো প্রধান উপায়:

**১. ডকার (Docker) + পোয়েট্রি (Poetry) দিয়ে (সহজ, দ্রুত, প্রোডাকশন-রেডি)**

- ১৫-২০ মিনিটে চালু করা যায়
- ডেভেলপমেন্ট এবং প্রোডাকশন উভয়ের জন্যই উপযুক্ত
- প্লাগইন ম্যানেজমেন্ট সহজ (পোয়েট্রি দিয়ে)
- অফিসিয়াল `nautobot-docker-compose` রিপোজিটরি ব্যবহার করে

**২. ট্র্যাডিশনাল ইনস্টলেশন (সম্পূর্ণ নিয়ন্ত্রণ নিজ হাতে)**

- একটু সময় লাগে (৩০-৪৫ মিনিট)
- Bare-metal বা non-Docker environment-এর জন্য
- সব কিছু নিজের মতো সাজানো যায়

নির্ভর কমিউনিকেশন দুটো পদ্ধতিই অনুসরণ করেছে:

আমরা দুটোই দেখব, তবে পদ্ধতি ১-এ বেশি মনোযোগ দেব।

---

### পদ্ধতি ১: ডকার + পোয়েট্রি দিয়ে  প্রোডাকশন-রেডি সেটআপ

নির্ভর কমিউনিকেশন এই পদ্ধতিটাই প্রোডাকশন-এ ব্যবহার করেছে। এটা অফিসিয়াল `nautobot-docker-compose` রিপোজিটরি এবং Poetry ব্যবহার করে — প্লাগইন ম্যানেজমেন্ট এবং version upgrade অনেক সহজ হয়।

#### অটোমেটেড ডেপ্লয়মেন্ট স্ক্রিপ্ট

নির্ভর কমিউনিকেশন একটা অটোমেটেড ডিপ্লয়মেন্ট স্ক্রিপ্ট  তৈরি করেছে যা সব কিছু একসাথে করে দেয়:

```bash
# deploy-nautobot.sh ডাউনলোড বা তৈরি করুন
nano deploy-nautobot.sh
chmod +x deploy-nautobot.sh
sudo ./deploy-nautobot.sh
```

এই স্ক্রিপ্ট নিচের সব কাজ করে:
- ডকার এবং ডকার কম্পোজ প্লাগইন ইনস্টল
- পোয়েট্রি ইনস্টল (পাইথন ডিপেনডেন্সি ম্যানেজমেন্ট)
- `nautobot-docker-compose` রিপোজিটরি ক্লোন
- সব এনভায়রনমেন্ট ফাইলস তৈরি (র‍্যান্ডম ক্রেডেনশিয়ালস সহ)
- কন্টেইনার বিল্ড এবং স্টার্ট
- সিস্টেমড সার্ভিস তৈরি (boot-এ অটো-স্টার্ট)
- ক্রেডেনশিয়াল একটা ফাইলে সেভ

স্ক্রিপ্ট শেষ হলে ক্রেডেনশিয়াল এখানে পাওয়া যাবে:

```bash
cat /opt/nautobot/CREDENTIALS.txt
```

#### ম্যানুয়াল সেটআপ (ধাপে ধাপে)

অটোমেটেড স্ক্রিপ্ট ব্যবহার না করলে, নিচের ধাপগুলো অনুসরণ করুন।

#### স্টেপ ১: সার্ভার প্রস্তুত করা

একটা নতুন Ubuntu 24.04 সার্ভার নিন। নির্ভর কমিউনিকেশন একটা ভিএম (VM: Virtual Machine) ব্যবহার করেছে তাদের এনওসি-এর ভেতরে।

এসএসএইচ (SSH) করে সার্ভারে লগইন করুন:

```bash
ssh admin@nautobot.nirvor.bd
```

সিস্টেম আপডেট করুন:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y git curl python3 python3-pip openssl
```

#### স্টেপ ২: ডকার ইনস্টল করা

ডকার এবং ডকার কম্পোজ প্লাগইন ইনস্টল করুন:

```bash
# ডকার এর অফিসিয়াল জিপিজি কি যোগ করুন
sudo apt install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# রিপোজিটরি যোগ করুন
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# ডকার ইনস্টল করুন
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# কারেন্ট ইউজার-কে docker group-এ যোগ করুন
sudo usermod -aG docker $USER
sudo systemctl enable docker && sudo systemctl start docker
```

যাচাই করুন:

```bash
docker --version
```

#### স্টেপ ৩: পোয়েট্রি ইনস্টল করা

পোয়েট্রি হলো পাইথন ডিপেনডেন্সি ম্যানেজমেন্ট টুল। নটবটের প্লাগইন ম্যানেজমেন্ট এবং বিল্ড প্রসেস পোয়েট্রি দিয়ে চলে।

```bash
curl -sSL https://install.python-poetry.org | python3 -

# পাথ-এ যোগ করুন
export PATH="$HOME/.local/bin:$PATH"
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

যাচাই করুন:

```bash
poetry --version
```

#### স্টেপ ৪: নটবট রিপোজিটরি সেটআপ

```bash
# Database credentials
NAUTOBOT_DB_USER=nautobot
NAUTOBOT_DB_PASSWORD=nirvor@2025$Secure  # শক্তিশালী পাসওয়ার্ড দিন
POSTGRES_PASSWORD=nirvor@2025$PostgreSQL  # আলাদা শক্তিশালী পাসওয়ার্ড

# Nautobot superuser
NAUTOBOT_CREATE_SUPERUSER=true
NAUTOBOT_SUPERUSER_USERNAME=admin
NAUTOBOT_SUPERUSER_PASSWORD=nirvor@Admin2025!  # শক্তিশালী পাসওয়ার্ড
NAUTOBOT_SUPERUSER_EMAIL=admin@nirvor.bd
NAUTOBOT_SUPERUSER_API_TOKEN=0123456789abcdef0123456789abcdef01234567  # Random 40-char token

# রেডিস পাসওয়ার্ড
NAUTOBOT_REDIS_PASSWORD=nirvor@Redis2025$
```

Invoke configuration:

```bash
NAUTOBOT_IMAGE=networktocode/nautobot:3.0-py3.11
NAUTOBOT_VERSION=3.0.0

# নির্ভর কমিউনিকেশন স্পেসিফিক সেটিংস
NAUTOBOT_ALLOWED_HOSTS=nautobot.nirvor.bd,localhost,127.0.0.1
```

#### স্টেপ ৫: এনভায়রনমেন্ট ফাইলস কনফিগার করা

`environments/local.env` তৈরি করুন:

```bash
nano environments/local.env
```

```bash
# নটবট সেটিংস
NAUTOBOT_ALLOWED_HOSTS=*
NAUTOBOT_DEBUG=False
NAUTOBOT_LOG_LEVEL=INFO
NAUTOBOT_TIME_ZONE=Asia/Dhaka

# ডাটাবেস সেটিংস
NAUTOBOT_DB_ENGINE=django.db.backends.postgresql
NAUTOBOT_DB_HOST=db          # গুরুত্বপূর্ণ: 'postgres' নয়, 'db' লিখুন
NAUTOBOT_DB_PORT=5432
NAUTOBOT_DB_NAME=nautobot
NAUTOBOT_DB_USER=nautobot
NAUTOBOT_DB_TIMEOUT=300

# রেডিস সেটিংস
NAUTOBOT_REDIS_HOST=redis
NAUTOBOT_REDIS_PORT=6379

# সেলারি সেটিংস — রেডিস পাসওয়ার্ড থাকলে ইউআরএল-এ %40 দিয়ে @ encode করুন
NAUTOBOT_CELERY_BROKER_URL=redis://:your_redis_password@redis:6379/0
NAUTOBOT_CELERY_RESULT_BACKEND=redis://:your_redis_password@redis:6379/0
NAUTOBOT_CACHES_LOCATION=redis://:your_redis_password@redis:6379/1

# সুপারইউজার
NAUTOBOT_CREATE_SUPERUSER=true
NAUTOBOT_SUPERUSER_NAME=admin
NAUTOBOT_SUPERUSER_EMAIL=admin@nirvor.bd
```

> ⚠️ **সতর্কতা:** Redis password-এ `@` চিহ্ন থাকলে connection URL-এ `%40` দিয়ে replace করতে হবে। যেমন `nirvorn@Redis2025$` হবে `nirvor%40Redis2025$`। নাহলে Celery connect করতে পারবে না।

`environments/creds.env` তৈরি করুন (এই ফাইল কখনো গিট-এ কমিট করবেন না!):

```bash
nano environments/creds.env
```

```bash
# সিক্রেট কি — এটা জেনারেট করুন:
# python3 -c "import secrets; print(secrets.token_urlsafe(50))"
NAUTOBOT_SECRET_KEY=your_generated_secret_key

# ডাটাবেস পাসওয়ার্ড
NAUTOBOT_DB_PASSWORD=your_strong_db_password
POSTGRES_PASSWORD=your_strong_db_password
POSTGRES_USER=nautobot
POSTGRES_DB=nautobot

# সুপারইউজার পাসওয়ার্ড
NAUTOBOT_SUPERUSER_PASSWORD=your_strong_admin_password
NAUTOBOT_SUPERUSER_API_TOKEN=your_40_char_hex_token

# রেডিস পাসওয়ার্ড
NAUTOBOT_REDIS_PASSWORD=your_redis_password
```

ফাইল দুটো secure করুন:

```bash
chmod 0600 environments/creds.env
chmod 0600 environments/local.env
```

#### স্টেপ ৬: পাইথন ভার্সন এবং নটবট ভার্সন সেট করা

```bash
export PYTHON_VER=3.12
export NAUTOBOT_VERSION=3.0.6
```

এই দুটো ভ্যারিয়েবল পরবর্তী সব বিল্ড কমান্ড-এ দরকার হবে।

#### স্টেপ ৭: পোয়েট্রি ডিপেনডেন্সিস ইনস্টল করা

```bash
cd /opt/nautobot
poetry install --no-interaction
```

#### স্টেপ ৮: কন্টেইনারস বিল্ড করা

```bash
export PYTHON_VER=3.12
export NAUTOBOT_VERSION=3.0.6

docker compose --project-name nautobot-docker-compose \
  --project-directory "environments/" \
  -f environments/docker-compose.postgres.yml \
  -f environments/docker-compose.base.yml \
  -f environments/docker-compose.local.yml \
  build --no-cache
```

এটা ১০-২০ মিনিট লাগতে পারে।

#### স্টেপ ৯: নটবট চালু করা

```bash
docker compose --project-name nautobot-docker-compose \
  --project-directory "environments/" \
  -f environments/docker-compose.postgres.yml \
  -f environments/docker-compose.base.yml \
  -f environments/docker-compose.local.yml \
  up -d
```

স্ট্যাটাস চেক করুন:

```bash
docker ps
```

দেখাবে:

```
nautobot-docker-compose-nautobot-1      Up (healthy)
nautobot-docker-compose-db-1           Up (healthy)
nautobot-docker-compose-redis-1        Up
nautobot-docker-compose-celery_worker-1 Up
nautobot-docker-compose-celery_beat-1  Up
```

লগস দেখুন:

```bash
docker logs nautobot-docker-compose-nautobot-1 --tail 50
```

#### স্টেপ ১০: প্রথম লগইন

ব্রাউজারে যান:

```
http://your-server-ip:8080
```

নির্ভর কমিউনিকেশনের ক্ষেত্রে:

```
https://nautobot.nirvor.bd:8080
```

`creds.env`-এ দেওয়া অ্যাডমিন ইউজারনেম ও পাসওয়ার্ড দিয়ে লগইন করুন।

অভিনন্দন! Nautobot 3.0.6 চালু হয়ে গেছে।

#### সাধারণ ম্যানেজমেন্ট কমান্ড

```bash
cd /opt/nautobot

# কন্টেইনার রিস্টার্ট করুন
docker compose --project-name nautobot-docker-compose \
  --project-directory "environments/" \
  -f environments/docker-compose.postgres.yml \
  -f environments/docker-compose.base.yml \
  -f environments/docker-compose.local.yml restart

# লগস দেখুন (লাইভ)
docker logs -f nautobot-docker-compose-nautobot-1

# নটবট শেল-এ যান
docker exec -it nautobot-docker-compose-nautobot-1 nautobot-server shell

# নতুন সুপারইউজার তৈরি করুন
docker exec -it nautobot-docker-compose-nautobot-1 \
  nautobot-server createsuperuser --username newuser --email user@nirvor.bd

# ডাটাবেস ব্যাকআপ নিন
docker exec nautobot-docker-compose-db-1 \
  pg_dump -U nautobot nautobot > nautobot_backup_$(date +%Y%m%d).sql
```

---

### পদ্ধতি ২: ট্র্যাডিশনাল ইনস্টলেশন বেয়ার-মেটাল (Bare-Metal)

এই পদ্ধতিতে ডকার ছাড়া সরাসরি সার্ভারে নটবট ইনস্টল করা হয়। ডকার ব্যবহারের সুযোগ না থাকলে বা সম্পূর্ণ নিয়ন্ত্রণ চাইলে এই পদ্ধতি ব্যবহার করুন।

https://aiwithr.github.io/automation_nsot/installation/
---

### প্লাগইন ইনস্টলেশন

নটবট চালু হওয়ার পর নির্ভর কমিউনিকেশন দুটো গুরুত্বপূর্ণ প্লাগইন যোগ করেছে।

#### ডকার পদ্ধতিতে প্লাগইন যোগ করা (পোয়েট্রি)

ডকার + পোয়েট্রি পদ্ধতিতে প্লাগইন যোগ করতে `poetry add` ব্যবহার করুন:

```bash
cd /opt/nautobot

# nautobot-ssot (Single Source of Truth integration framework)
poetry add nautobot-ssot[all]

# nautobot-chatops (Slack/Teams integration)
poetry add nautobot-chatops
```

প্লাগইন যোগ করার পর কন্টেইনার রিবিল্ড করতে হবে:

```bash
export PYTHON_VER=3.12
export NAUTOBOT_VERSION=3.0.6

docker compose --project-name nautobot-docker-compose \
  --project-directory "environments/" \
  -f environments/docker-compose.postgres.yml \
  -f environments/docker-compose.base.yml \
  -f environments/docker-compose.local.yml \
  build --no-cache

docker compose --project-name nautobot-docker-compose \
  --project-directory "environments/" \
  -f environments/docker-compose.postgres.yml \
  -f environments/docker-compose.base.yml \
  -f environments/docker-compose.local.yml \
  up -d
```

#### nautobot_config.py-তে প্লাগইন এনেবল করা

`/opt/nautobot/config/nautobot_config.py` ফাইলে প্লাগইনস লিস্ট-এ যোগ করুন:

```python
PLUGINS = [
    "nautobot_ssot",
    "nautobot_chatops",
]

PLUGINS_CONFIG = {
    "nautobot_ssot": {
        # এসএসওটি কনফিগারেশন
    },
    "nautobot_chatops": {
        # চ্যাটঅপস কনফিগারেশন (স্ল্যাক টোকেন ইত্যাদি)
    },
}
```

কন্টেইনারস রিস্টার্ট করুন:

```bash
docker compose --project-name nautobot-docker-compose \
  --project-directory "environments/" \
  -f environments/docker-compose.postgres.yml \
  -f environments/docker-compose.base.yml \
  -f environments/docker-compose.local.yml \
  restart
```

