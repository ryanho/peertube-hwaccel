

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


In PeerTube admin interface install "transcoding-profile-debug" plugin. Change its setting as following

Transcoding profiles

    {
      "vod": [
        {
          "encoderName": "aac",
          "profileName": "test",
          "outputOptions": []
        },
        {
          "encoderName": "h264_nvenc",
          "profileName": "test",
         "inputOptions": [
            "-hwaccel nvdec"
        ],
          "outputOptions": [
            "-c:v h264_nvenc",
            "-b:v 4M", 
            "-maxrate:v 5M", 
            "-bufsize:v 8M"
          ]
        }
      ],
      "live": []
    }


Encoders priorities

    {
      "vod": [
        {
          "encoderName": "aac",
          "streamType": "audio",
          "priority": 1000
        },
    
       {
          "encoderName": "h264_nvenc",
          "streamType": "video",
          "priority": 1000
        }
      ],
    
      "live": [ ]
    
    }

**Above profile just works, does not optimize.**

Then in Setting/VOD transcode, select "test" in transcode profile drop down list.

That's it.
