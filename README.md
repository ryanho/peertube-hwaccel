2025/03/19 - Hardware transcoding is not recommended because of the poor quality. Change this repository into an archive.

# peertube-hwaccel
This is based on **[docker-ffmpeg](https://github.com/linuxserver/docker-ffmpeg)**, thanks for their hard work. The following is based on the assumption that a PeerTube service has already been set up.

## How to use

Get Dockerfile and peertube source code. Replace **$tag_version** with specific release version, eg. v5.2.0

    cd /tmp
    wget https://raw.githubusercontent.com/ryanho/peertube-hwaccel/main/Dockerfile
    git clone --depth 1 --branch $tag_version https://github.com/Chocobozzz/PeerTube.git

Build image

    cd PeerTube
    docker build . -t peertube-hwaccel -f ../Dockerfile

Edit peertube section in docker-compose.yml, replace "image: peertube-hwaccel" and add "runtime: nvidia"

      peertube:
        # If you don't want to use the official image and build one from sources:
        # build:
        #   context: .
        #   dockerfile: ./support/docker/production/Dockerfile.bullseye
        image: peertube-hwaccel
        runtime: nvidia
        # Use a static IP for this container because nginx does not handle proxy host change without reload
        # This container could be restarted on crash or until the postgresql database is ready for connection
        networks:
          default:
            ipv4_address: 172.18.0.42
        env_file:
          - .env

Restart service

    docker compose down -v
    docker compose up -d


In PeerTube admin interface install "hardware-transcode-nvenc" plugin. You can change quality and bitrate in the plugin setting.

Then in Setting/VOD transcode, select "nvenc" in transcode profile drop down list.

That's it.
