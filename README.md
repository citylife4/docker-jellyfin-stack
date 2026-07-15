# 🐳docker-jellyfin-stack
This repository contains a docker-compose configuration for a fully automated home media server. It integrates media management, downloading, streaming, and system monitoring into a unified stack.
### Requirements
- Linux (Ubuntu, Debian, or Alpine) is recommended for lowest overhead
- Docker: Engine v20.10 or newer.
- Docker Compose: v2.0 or newer.

*In my case i am currently running this entire stack on my Raspberry Pi 5 (8GB RAM)

## 📂 Directory Structure & Architecture
To ensure optimal performance and avoid storage duplication, this project relies on a specific folder hierarchy.

Run the following command on your **external disk mount** to generate the required directory tree:

```bash
mkdir -p /mnt/external-disk/jellyfin-stack/data/{media/{movies,series},torrents}
```

```text
├── /mnt/external-disk/jellyfin-stack/data
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
Set `DATA_DIR` and `DOWNLOAD_DIR` to your external disk path so all media/movies/series/torrents live on that disk.

## 🛠️ Installation & Usage
Run the following command in your terminal inside the project folder:
```bash
docker compose up -d
```
Once started, you can access the services
| Service       | Port  | URL (Example)             | Description |
| :---          | :---  | :---                      | :---                            |
| **qBittorrent**| 8080 | `http://<your-ip>:8080`   | Torrent Downloader |
| **Radarr** | 7878  | `http://<your-ip>:7878`   | Movie Management |
| **Sonarr** | 8989  | `http://<your-ip>:8989`   | TV Series Management |
| **Bazarr** | 6767  | `http://<your-ip>:6767`   | Subtitle Management |
| **Prowlarr** | 9696  | `http://<your-ip>:9696`   | Indexer/Tracker Proxy |
| **Unpackerr** ||| Extracts Radarr/Sonarr downloads|
| **Jellyfin** | 8096  | `http://<your-ip>:8096`   | Media Server & Streaming |
| **Homepage** | 3000  | `http://<your-ip>:3000`   | Main Dashboard |
| **Glances** | 61208 | `http://<your-ip>:61208`  | System Monitoring |
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
- Sonarr Server: http://your-ip:8989  
- API Key: Get this from Sonarr (Settings > General).

Click "Test" and "Save". This will automatically sync your indexers.

#### Bazarr
Bazarr handles subtitle downloads and sync.

Connect applications from Bazarr:
- Settings > Sonarr > Add Sonarr:
  - Address: `http://sonarr:8989`
  - API Key: from Sonarr (Settings > General)
  - Root folders in container: `/data/media/series`
- Settings > Radarr > Add Radarr:
  - Address: `http://radarr:7878`
  - API Key: from Radarr (Settings > General)
  - Root folders in container: `/data/media/movies`

After saving, Bazarr will scan your existing media and fetch subtitles based on your language/profile settings.

#### Unpackerr
Unpackerr is responsible for automatically extracting compressed files (.zip, .rar, .7zip...) downloaded by qBitorrent so Radarr and Sonarr can import them.  
To make Unpackerr work, you must link it with Radarr and Sonarr using your API Keys.

Open your .env file and add your keys from the web settings (Settings > General) of each app:
```text
RADARR_API_KEY= your_key_here  
SONARR_API_KEY= your_key_here
```
After saving the file, you must run the following command to refresh the configuration:
```bash
docker compose up -d
```
To check if Unpackerr is working and connected to Radarr/Sonarr run:
```bash
docker compose logs unpackerr
```
It should look something like this:
```text
[Radarr] Updated (http://radarr:7878): 0 Items Queued, 0 Retrieved
[Sonarr] Updated (http://sonarr:8989): 0 Items Queued, 0 Retrieved
```
### 🪼 Jellyfin
Follow the instructions in the initial setup.

Add Media Libraries:  
Movies: Content type "Movies". Folder: /data/media/movies.  
Series: Content type "Series". Folder: /data/media/series.

Check out [ElegantFin](https://github.com/lscambo13/ElegantFin) by lscambo13 for a more modern look, it's personally my favorite.
### 🛖 Homepage
Homepage service provides a clean UI to access all your tools. To customize it, edit the files in ${CONFIG_DIR}/homepage_config. It is pre-configured to communicate with the Docker socket to show container status.

There are many configuration options available to customize your dashboard. For the best results and correct setup, it is highly recommended to follow the official Homepage [documentation](https://gethomepage.dev/configs/).
