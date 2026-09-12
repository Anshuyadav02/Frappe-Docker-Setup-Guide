# 🐳 Docker + Frappe Site Setup on Ubuntu

A simple step-by-step guide to install Docker and set up a Frappe site on your Ubuntu laptop.

---

## 🗺️ Roadmap

| # | Stage | What you get |
|---|---|---|
| 1 | Install Docker | Containers ready |
| 2 | Install Docker Compose | Multi-container support |
| 3 | Give user permission | Run Docker without `sudo` |
| 4 | Clone frappe_docker | Project files + dev config |
| 5 | Setup bench | Frappe framework installed |
| 6 | Connect containers | DB + Redis linked |
| 7 | Create site | Your website is live |
| 8 | Create app | Your custom app |
| 9 | Install app | App added to site |

---

## 1️⃣ Install Docker Package

Update your system first, then install Docker.

```bash
sudo apt update
```

```bash
sudo apt install docker.io
```

> 💡 **What's happening:** `apt update` refreshes the package list, and `docker.io` installs the Docker engine.

---

## 2️⃣ Install Docker Compose

```bash
sudo apt install docker-compose
```

> 💡 **What's happening:** Frappe needs several containers running together — Frappe, MariaDB, and Redis. Docker Compose manages all of them with a single command.

---

## 3️⃣ Give Your User Docker Permission

Add your user to the `docker` group:

```bash
sudo usermod -aG docker $USER
```

Apply the new group without logging out:

```bash
newgrp docker
```

Check that it works:

```bash
docker ps
```

> 💡 **What's happening:** By default only `root` can talk to the Docker daemon. Adding yourself to the `docker` group lets you run Docker commands without typing `sudo` every time.
>
> ⚠️ **Don't skip this step.** Without it you will hit "permission denied while trying to connect to the Docker daemon" errors later.
>
> ℹ️ If `newgrp docker` does not work, log out and log back in (or restart your laptop).

---

## 4️⃣ Clone the Frappe Docker Repo

Download the official project:

```bash
git clone https://github.com/frappe/frappe_docker.git
```

Go inside the folder:

```bash
cd frappe_docker
```

Copy the example devcontainer config into `.devcontainer`:

```bash
cp -R devcontainer-example .devcontainer
```

> 💡 **What's happening:** The `.devcontainer` folder tells Docker which containers to build for development. It is a hidden folder — use `ls -a` to see it.

---

## 5️⃣ Setup Bench

Initialize bench:

```bash
bench init --skip-redis-config-generation frappe-bench
```

Enter the bench folder:

```bash
cd frappe-bench
```

### 🎯 Want a specific version instead?

Use this command with the branch name:

```bash
bench init --skip-redis-config-generation --frappe-branch version-14 frappe-bench
```

> 💡 **What's happening:** `--skip-redis-config-generation` is used because Redis runs in its own separate container, so bench does not need to generate its own Redis config.
>
> ⚠️ Run only **one** of the two `bench init` commands. Use the second one if you need a specific branch, otherwise use the first.

---

## 6️⃣ Point Bench to the Right Containers

By default bench looks for the database and Redis on `localhost`. But here they run in separate containers, so we tell bench their real addresses.

**Run these inside the container:**

```bash
bench set-config -g db_host mariadb
```

```bash
bench set-config -g redis_cache redis://redis-cache:6379
```

```bash
bench set-config -g redis_queue redis://redis-queue:6379
```

```bash
bench set-config -g redis_socketio redis://redis-queue:6379
```

| Setting | Points to |
|---|---|
| `db_host` | MariaDB container |
| `redis_cache` | Cache container |
| `redis_queue` | Background jobs container |
| `redis_socketio` | Real-time updates container |

> 💡 **What's happening:** The `-g` flag means **global**, so this setting applies to all sites.
>
> ⚠️ Do not skip this step, otherwise you will get a database connection error while creating the site.

---

## 7️⃣ Create a New Site

```bash
bench new-site site_name.localhost
```

> 💡 **What's happening:** Replace `site_name` with your own name, for example `mysite.localhost`.
>
> ✅ Always end the site name with `.localhost`.
>
> 🔑 This command will ask for the MySQL root password and an Administrator password. **Remember the Administrator password** — you will use it to log in.

---

## 8️⃣ Create a New App

```bash
bench new-app ap_name
```

> 💡 **What's happening:** Replace `ap_name` with your own app name.
>
> ✅ Use **lowercase** letters and **underscores** — `my_app` is correct, `My App` or `my-app` will fail.
>
> It will ask for App Title, Description, Publisher and so on. You can press Enter to skip these.

---

## 9️⃣ Install the App on Your Site

```bash
bench --site site_name install-app appname
```

> 💡 **What's happening:** Here `site_name` is the site you created in Step 7, and `appname` is the app you created in Step 8.
>
> 🎉 Done! Your Frappe site and custom app are ready.

---

## 📌 All Commands in One Place

```bash
# 1. Install Docker
sudo apt update
sudo apt install docker.io

# 2. Install Docker Compose
sudo apt install docker-compose

# 3. Give user Docker permission
sudo usermod -aG docker $USER
newgrp docker
docker ps

# 4. Clone repo
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
cp -R devcontainer-example .devcontainer

# 5. Setup bench
bench init --skip-redis-config-generation frappe-bench
cd frappe-bench
# (for a specific version)
# bench init --skip-redis-config-generation --frappe-branch version-14 frappe-bench

# 6. Connect containers
bench set-config -g db_host mariadb
bench set-config -g redis_cache redis://redis-cache:6379
bench set-config -g redis_queue redis://redis-queue:6379
bench set-config -g redis_socketio redis://redis-queue:6379

# 7. Create site
bench new-site site_name.localhost

# 8. Create app
bench new-app ap_name

# 9. Install app
bench --site site_name install-app appname
```

---

## ✅ Quick Checklist

- [ ] Docker installed
- [ ] Docker Compose installed
- [ ] User added to the `docker` group
- [ ] Repo cloned + `.devcontainer` copied
- [ ] Bench initialized
- [ ] All 4 `set-config` commands run
- [ ] Site created
- [ ] App created
- [ ] App installed on site
