# HomeAssist_Voice

# Home Assistant & Voice Assistant Docker Setup

This project uses Docker Compose to run a local smart home and voice assistant system. It deploys:

- **Home Assistant** – An open-source home automation platform.
- **Whisper** – A speech-to-text engine (using the Rhasspy Wyoming Whisper image).
- **Piper** – A text-to-speech engine (using the Rhasspy Wyoming Piper image).
- **OpenWakeWord** – A wake word detection engine (using the Rhasspy Wyoming OpenWakeWord image).

They are all connected on a custom Docker network (`ha_network`) to enable smooth integration.

## Project Components

- **Home Assistant**: Runs your smart home dashboard on port `8123`.
- **Whisper**: Processes spoken commands (STT) on port `10300` using the tiny-int8 model.
- **Piper**: Generates spoken responses (TTS) on port `10200` using an English (UK) female voice.
- **OpenWakeWord**: Listens for a wake word on port `10400` with a preloaded model (`ok_nabu`).

## Prerequisites

- **Docker**: Install Docker on your host machine.  [Docker Installation Instructions](https://docs.docker.com/get-docker/) 
- **Docker Compose**: Install Docker Compose.   [Docker Compose Installation Guide](https://docs.docker.com/compose/install/)
- Basic command-line knowledge.

## Installation

Follow these steps to get your system up and running:

### 1. Clone or Download the Repository

Clone this repository to your local machine:
```bash
git clone https://github.com/HermiTManCode/HomeAssist_Voice.git
cd HomeAssist_Voice
````

### 2. Create Required Directories

Create local folders for persistent data:

```bash
mkdir config whisper-data piper-data openwakeword-data
```

### 3. Review the Docker Compose File

Your `docker-compose.yaml` file should look similar to this:

```yaml
version: "3"
services:
  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    container_name: homeassistant
    restart: unless-stopped
    privileged: true
    networks:
      - ha_network
    volumes:
      - ./config:/config
      - /etc/localtime:/etc/localtime:ro
    environment:
      - TZ=Africa/Nairobi
    ports:
      - "8123:8123"

  whisper:
    container_name: whisper
    image: rhasspy/wyoming-whisper:latest
    ports:
      - "10300:10300"
    networks:
      - ha_network
    volumes:
      - ./whisper-data:/data
    command: --model tiny-int8 --language en
    environment:
      - TZ=Africa/Nairobi
    restart: unless-stopped

  piper:
    container_name: piper
    image: rhasspy/wyoming-piper:latest
    ports:
      - "10200:10200"
    networks:
      - ha_network
    volumes:
      - ./piper-data:/data
    command: --voice en-gb-southern_english_female-low
    environment:
      - TZ=Africa/Nairobi
    restart: unless-stopped

  openwakeword:
    container_name: openwakeword
    image: rhasspy/wyoming-openwakeword:latest
    ports:
      - "10400:10400"
    volumes:
      - ./openwakeword-data:/data
    networks:
      - ha_network
    environment:
      - TZ=Africa/Nairobi
    command: --preload-model 'ok_nabu' --custom-model-dir /data/custom
    restart: unless-stopped

networks:
  ha_network:
    driver: bridge
```

### 4. Start the Containers

In the same directory as your `docker-compose.yaml`, run:

```bash
docker-compose up -d
```

This command downloads the images (if not already cached) and starts the containers in detached mode.

### 5. Access Home Assistant

Open your browser and navigate to:

```
http://<YOUR_HOST_IP>:8123
```

Replace `<YOUR_HOST_IP>` with the IP address of your Docker host.

## Configuration & Usage

- **Time Zone**: The system uses `TZ=Africa/Nairobi`. Adjust this variable if you’re in a different time zone.
- **Persistent Data**: The directories (`config`, `whisper-data`, `piper-data`, `openwakeword-data`) are mounted as volumes to store your settings and data persistently.
- **Home Assistant Integration**:
    
    - Once Home Assistant is up, use its web interface to add integrations.
    - To connect the voice assistant services, configure the Wyoming protocol integration in Home Assistant. When prompted, use:
        - **Whisper (STT)**: Port `10300`
        - **Piper (TTS)**: Port `10200`
        - **OpenWakeWord**: Port `10400`
    
    For further details, see the [Home Assistant Wyoming Integration Documentation](https://www.home-assistant.io/integrations/wyoming/)

## Troubleshooting

- **Logs**: Check container logs with:
    
    ```bash
    docker logs homeassistant
    docker logs whisper
    docker logs piper
    docker logs openwakeword
    ```
    
- **Network Issues**: Ensure no other service is using the specified ports and that the `ha_network` is active.
- **Restart Home Assistant**: If the Wyoming integration is not detected automatically, try restarting Home Assistant as some users have found this resolves detection issues.

## References

- [Docker Installation](https://docs.docker.com/get-docker/)
- [Docker Compose Installation](https://docs.docker.com/compose/install/)
- [Home Assistant Official Site](https://www.home-assistant.io/)
- [Home Assistant Wyoming Integration Documentation](https://www.home-assistant.io/integrations/wyoming/)
- [Rhasspy Wyoming Addons on GitHub](https://github.com/rhasspy/wyoming-addons)

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.


