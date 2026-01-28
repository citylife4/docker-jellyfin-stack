# 🐳docker-jellyfin-stack
This repository contains a docker-compose configuration for a fully automated home media server. It integrates media management, downloading, streaming, and system monitoring into a unified stack.
### Requirements
- Linux (Ubuntu, Debian, or Alpine) is recommended for lowest overhead
- Docker: Engine v20.10 or newer.
- Docker Compose: v2.0 or newer.

*In my case i am currently running this entire stack on my Raspberry Pi 5 (8GB RAM)

## 📂 Directory Structure & Architecture
To ensure optimal performance and avoid storage duplication, this project relies on a specific folder hierarchy.

Run the following command to generate the required directory tree:

```bash
mkdir -p data/{media/{movies,series},torrents}
```

```text
├── data
    ├── media
    │   ├── movies
    │   └── series
    └── torrents
```
⚠️ Important: Before installing dependencies or running the application, you must configure your environment variables.

Locate the .env.example file in the root directory. Create a copy of this file and rename it to .env.
```bash
cp .env.example .env
```
Open the new .env file and fill in the required values (API keys, database credentials, specific ports, etc.).

## 🛠️ Installation & Usage
Run the following command in your terminal inside the project folder:
```bash
docker compose up -d
```
Once started, you can access the services
| Service       | Port  | URL (Example)             | Description                     |
| :---          | :---  | :---                      | :---                            |
| **qBittorrent**| 8080 | `http://<your-ip>:8080`   | Torrent Downloader              |
| **Radarr** | 7878  | `http://<your-ip>:7878`   | Movie Management                |
| **Sonarr** | 8989  | `http://<your-ip>:8989`   | TV Series Management            |
| **Prowlarr** | 9696  | `http://<your-ip>:9696`   | Indexer/Tracker Proxy           |
| **Jellyfin** | 8096  | `http://<your-ip>:8096`   | Media Server & Streaming        |
| **Homepage** | 3000  | `http://<your-ip>:3000`   | Main Dashboard |
| **Glances** | 61208 | `http://<your-ip>:61208`  | System Monitoring               |
### 🌍 qBittorrent
On the very first startup, qBittorrent generates a random temporary password. You need to check the container logs to find it.
```bash
docker compose logs qbittorrent
```
Once you have logged in with the temporary password found in the logs:

Go to Tools > Options > Web UI  
Under the Authentication section, enter your desired Username and Password.

Configure Download Location

Go to Tools > Options > Downloads  
Set the Default Save Path to: /data/torrents

Scroll down and click Save.
### 🦜 Arr Config
#### Radarr/Sonnar
The setup for both is nearly identical.  
Go to Settings > Media Management.  
Click "Add Root Folder" browse to /data/media/movies (for Radarr) and /data/media/series (for Sonarr).

Download Client:  
Go to Settings > Download Clients > Select qBittorrent
- Host: your ip address
- Port: 8080.
- Username/Password: Use the credentials you set in qBittorrent.
- Test and Save.

#### Prowlarr
This connects your torrent trackers to Sonarr and Radarr.

Add Indexers: Go to Indexers and search the trackers you wanna add

Connect Apps: Go to Settings > Apps > +

Radarr:  
- Prowlarr Server: http://your-ip:9696
- Radarr Server: http://your-ip:7878  
- API Key: Get this from Radarr (Settings > General).

Sonarr:  
- Prowlarr Server: http://your-ip:9696
- Sonarr Server: http://your-ip8989  
- API Key: Get this from Sonarr (Settings > General).

Click "Test" and "Save". This will automatically sync your indexers.
### 🪼 Jellyfin
Follow the instructions in the initial setup.

Add Media Libraries:  
Movies: Content type "Movies". Folder: /data/media/movies.  
Series: Content type "Series". Folder: /data/media/series.

Check out [ElegantFin](https://github.com/lscambo13/ElegantFin) by lscambo13 for a more modern look, it's personally my favorite.
### 🛖 Homepage
Homepage service provides a clean UI to access all your tools. To customize it, edit the files in ${CONFIG_DIR}/homepage_config. It is pre-configured to communicate with the Docker socket to show container status.

There are many configuration options available to customize your dashboard. For the best results and correct setup, it is highly recommended to follow the official Homepage [documentation](https://gethomepage.dev/configs/).
