# Master Your Media: Step-by-Step Guide to TVHeadend, Kodi, and Plex Integration

This guide will take you through the complete process of setting up TVHeadend, configuring Kodi and Plex as clients, and seamlessly managing your live TV, recorded shows, and media libraries.

---

## Table of Contents
1. [Introduction](#introduction)
2. [Installing Docker](#installing-docker-tested-on-linux-debian)
3. [Setting Up TVHeadend](#setting-up-tvheadend)
4. [Configuring Kodi with TVHeadend](#configuring-kodi-with-tvheadend)
5. [Configuring Plex with TVHeadend](#configuring-plex-with-tvheadend)
6. [Managing Libraries and Live TV](#managing-libraries-and-live-tv)
7. [Troubleshooting and Tips](#troubleshooting-and-tips)

---

## Introduction
**TVHeadend** is a powerful, open-source TV streaming server that lets you watch and record live TV. Combined with **Kodi** and **Plex**, you can create a seamless home entertainment experience. Follow this guide to:
- Set up TVHeadend as your TV server.
- Configure Kodi and Plex to access live TV and recorded content.
- Enjoy a fully integrated media library across devices.

---

## Installing Docker (Tested on Linux Debian)
### Steps
1. **Uninstall old versions**
   - Before you can install Docker Engine, you need to uninstall any conflicting packages.

     ```bash
     for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
     ```
2. **Add Docker's official GPG key:**
   - Set up Docker's apt repository.

     ```bash
     # Add Docker's official GPG key:
     sudo apt-get update
     sudo apt-get install ca-certificates curl
     sudo install -m 0755 -d /etc/apt/keyrings
     sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
     sudo chmod a+r /etc/apt/keyrings/docker.asc
      
     # Add the repository to Apt sources:
     echo \
       "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
       $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
       sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
     sudo apt-get update
     ```
3. **Install Docker packages**:
   - To install the latest version, run:

     ```bash
     sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
     ```
   - Verify that the installation is successful by running the hello-world image:

     ```bash
     sudo docker run hello-world
     ```
---

## Setting Up TVHeadend
### Prerequisites
- Docker installed.
- Configuring docker compose.yaml
- (Suggested) Install Portainer to have a visual dashboard
- A TV tuner (hardware) connected to your server.


### Steps
1. **Setup docker compose.yaml**:
   - Create a file called `compose.yaml` into a `/opt` folder (sudo is required)

     ```bash
      cd /opt
      touch compose.yaml
     ```

2. **Install Portainer**
   - Portainer is a UI that allows the main functions you should do from the console. Copy following code to your compose file:

     ```bash
     ---
       portainer:
         container_name: portainer
         image: portainer/portainer-ce
         restart: always
         environment:
           - TZ=Europe/Rome
         network_mode: host
         volumes:
           - /var/run/docker.sock:/var/run/docker.sock
           - /opt/docker/portainer/data:/data
      ```

### Install and Setup Tvheadend

1. **Install TVHeadend**:

   - Copy following code to your compose file:

     ```bash
     tvheadend:
       image: lscr.io/linuxserver/tvheadend:latest
       container_name: tvheadend
       environment:
         - PUID=1000
         - PGID=1000
         - TZ=Europe/Rome
         - RUN_OPTS= #optional
       volumes:
         - /opt/docker/tvheadend/data:/config
         - /opt/docker/tvheadend/recordings:/recordings
       ports:
         - 9981:9981
         - 9982:9982
       devices:
         - /dev/dri:/dev/dri #optional
         - /dev/dvb:/dev/dvb #optional
       restart: unless-stopped
     ```

3. **Access TVHeadend**:
   - Open your browser and go to `http://<your-server-ip>:9981`.

4. **Initial Configuration**:
   - Follow the wizard to set up:
     - **Expert Mode**: First of all select `Default view level:` to `Expert`
     - **Networks**: Add your tuner and assign the correct frequencies.
     - **Channel Scanning**: Scan for available channels.
     - **User Access**: Create accounts for Kodi and Plex.

5. **Enable HDHomeRun Emulation**:
   - In TVHeadend, go to `Configuration > General > Base`.
   - Enable **HDHomeRun emulation** to allow Plex to detect TVHeadend as a DVR.

6. **Test TVHeadend**:
   - Ensure you can stream live TV through the TVHeadend web interface.

---

## Configuring Kodi with TVHeadend
### Steps
1. **Install the TVHeadend Add-on**:
   - Open Kodi, go to `Add-ons > Install from Repository > PVR Clients > TVHeadend HTSP Client`.

2. **Configure the Add-on**:
   - Go to `Settings` in the add-on and enter your TVHeadend server details:
     - **IP Address**: `<your-server-ip>`
     - **Port**: `9982`
     - **Username** and **Password**: From TVHeadend configuration.

3. **Enable the Add-on**:
   - After configuring, enable the add-on. Live TV should now appear in Kodi's main menu.

4. **Test Playback**:
   - Navigate to `Live TV` in Kodi and start streaming.

---

## Configuring Plex with TVHeadend
### Steps
1. **Add TVHeadend as a DVR Source**:
   - Open Plex and go to `Settings > Live TV & DVR`.
   - Plex will automatically detect TVHeadend via HDHomeRun emulation.

2. **Configure the DVR**:
   - Follow the setup wizard to map channels and set up the EPG.

3. **Test Playback**:
   - Go to `Live TV` in Plex and test the stream.

---

## Managing Libraries and Live TV
1. **Add Recorded Shows to Kodi and Plex**:
   - Set the recording path in TVHeadend (in this case `/opt/docker/tvheadend/recordings:/recordings`).
   - Add this path as a library in Kodi and Plex.

2. **Organize Media Libraries**:
   - In Plex, go to `Settings > Libraries` and organize your content.
   - In Kodi, use the `Videos > Add Source` option to include your recordings.

3. **Schedule Recordings in TVHeadend**:
   - Use the TVHeadend web interface to schedule recordings.

---

## Troubleshooting and Tips
- **Driver Issue**:
  - Check if Linux System recognizes the tuner otherwise search the internet for the model and if it is supported by the system
- **Buffering Issues**:
  - Check network bandwidth and reduce stream quality if necessary.
- **EPG Not Loading**:
  - Verify the EPG source in TVHeadend.
- **Access Denied**:
  - Ensure usernames and passwords are correctly set in both TVHeadend and clients.

---

With this guide, you can now enjoy live TV, manage recordings, and organize your media libraries effortlessly across devices. Happy streaming!

