# How to integrate BBB into a corporate network

  
Nowadays it is not unusual to operate applications such as BigBlueButton in a secure company network. 
In such a network you will find reverse proxies, your own CA, an Internet proxy, etc. 

It is difficult to get BBB for docker running in such an environment and the information on the internet is sparse or widely scattered. 

Therefore, this guide, based on BBB Docker version 2.7.3, will show how to configure the environment so that BBB can work via a reverse proxy with its own certificates and an internet proxy. We also use SIP dial-in and Keycloak for the external authentification but the configuration is not part of this documentation.

>  **Note.** These instructions are based on the implementation in a special company environment. Please check whether the requirements are suitable for your environment before implementation!

## Pre-Installation

1. Clone the BigBlueButton Docker repository as explained.
2. You do not need to run the setup script, you can directly edit the .env file. You can find an example below (Customize the file to your requirements):

```
# ====================================
# ADDITIONS to BigBlueButton
# ====================================
# (place a '#' before to disable them)

# HTTPS Proxy
# fully automated Lets Encrypt certificates
ENABLE_HTTPS_PROXY=false

# coturn (a TURN Server)
# requires either the abhove HTTPS Proxy to be enabled
# or TLS certificates to be mounted to container
ENABLE_COTURN=false
#COTURN_TLS_CERT_PATH=
#COTURN_TLS_KEY_PATH=

# Greenlight Frontend
# https://docs.bigbluebutton.org/greenlight/gl-overview.html
ENABLE_GREENLIGHT=true
# Because of the use of KeyCloak we do not need Greenlight Accounts . Setting this environment to false the "Register"-Button on the homepage will disappear.
ALLOW_GREENLIGHT_ACCOUNTS=false

# Enable Webhooks
# used by some integrations
#ENABLE_WEBHOOKS=true

# Prometheus Exporter
# serves the bigbluebutton-exporter under following URL:
# https://yourdomain/bbb-exporter
#ENABLE_PROMETHEUS_EXPORTER=true
#ENABLE_PROMETHEUS_EXPORTER_OPTIMIZATION=true

# Recording
# IMPORTANT: this is currently a big privacy issues, because it will
# record everything which happens in the conference, even when the button
# suggets, that it does not.
# https://github.com/bigbluebutton/bigbluebutton/issues/9202
# make sure that you get peoples consent, before they join a room
ENABLE_RECORDING=false
#REMOVE_OLD_RECORDING=false
#RECORDING_MAX_AGE_DAYS=14

# ====================================
# SECRETS
# ====================================
# important! change these to any random values
RAILS_SECRET=ThisIsSuperSecret
SHARED_SECRET=ThisIsSuperSecret
ETHERPAD_API_KEY=ThisIsSuperSecret
POSTGRESQL_SECRET=ThisIsSuperSecret
FSESL_PASSWORD=ThisIsSuperSecret

# ====================================
# CONNECTION
# ====================================

DOMAIN=FQDN of your BBB server

EXTERNAL_IPv4=internal IP of your BBB server (e.g. 10.0.1.234)
EXTERNAL_IPv6=

# STUN SERVER
# stun.freeswitch.org
#STUN_IP=216.93.246.18
STUN_IP=IP of your STUN server
STUN_PORT=3478

# TURN SERVER
# uncomment and adjust following two lines to add an external TURN server
TURN_SERVER=turn:<FQDN of your TURN server>?transport=tcp
TURN_SECRET=TopSecret

# Allowed SIP IPs
# due to high traffic caused by bots, by default the SIP port is blocked.
# but you can allow access by your providers IP or IP ranges (comma seperated)
# Hint: if you want to allow requests from every IP, you can use 0.0.0.0/0
SIP_IP_ALLOWLIST=


# ====================================
# CUSTOMIZATION
# ====================================

CLIENT_TITLE=BigBlueButton ACME Inc.

# use following lines to replace the default welcome message and footer
WELCOME_MESSAGE="Welcome to <b>%%CONFNAME%%</b>!<br><br>For help on using BigBlueButton see these (short) <a href='https://www.bigbluebutton.org/html5' target='_blank'><u>tutorial videos</u></a>.<br><br>To join the audio bridge click the speaker button.  Use a headset to avoid causing background noise for others."
WELCOME_FOOTER="This server is running <a href='https://docs.bigbluebutton.org/'' target='_blank'><u>BigBlueButton</u></a>."

# use following line for an additional SIP dial-in message
#WELCOME_FOOTER="This server is running <a href='https://docs.bigbluebutton.org/' target='_blank'><u>BigBlueButton</u></a>. <br><br>To join this meeting by phone, dial:<br>  INSERT_YOUR_PHONE_NUMBER_HERE<br>Then enter %%CONFNUM%% as the conference PIN number."


# for a different default presentation, place the pdf file in ./conf/ and
# adjust the following path
DEFAULT_PRESENTATION=./mod/nginx/default.pdf

# language of sound announcements
# options:
# - en-ca-june - EN Canadian June
# - en-us-allison - US English Allison
# - en-us-callie - US English Callie (default)
# - de-de-daedalus3 - German by Daedalus3 (https://github.com/Daedalus3/freeswitch-german-soundfiles)
# - es-ar-mario - Spanish/Argentina Mario
# - fr-ca-june - FR Canadian June
# - pt-br-karina - Brazilian Portuguese Karina
# - ru-RU-elena - RU Russian Elena
# - ru-RU-kirill - RU Russian Kirill
# - ru-RU-vika - RU Russian Viktoriya
# - sv-se-jakob - Swedish (Sweden) Jakob
# - zh-cn-sinmei - Chinese/China Sinmei
# - zh-hk-sinmei - Chinese/Hong Kong Sinmei
SOUNDS_LANGUAGE=de-de-daedalus3

# set to false to disable listenOnlyMode
LISTEN_ONLY_MODE=true

# set to true to disable echo test
DISABLE_ECHO_TEST=false

# set to true to automatically share webcam
AUTO_SHARE_WEBCAM=false

# set to true to disable video preview for webcam sharing
DISABLE_VIDEO_PREVIEW=false

# set to false to disable chat
CHAT_ENABLED=true

# set to true to start chat closed
CHAT_START_CLOSED=false

# set to true to disable announcements "You are now (un-)muted"
DISABLE_SOUND_MUTED=true

# set to true to disable announcement "You are the only person in this conference"
DISABLE_SOUND_ALONE=false

# maximum count of breakout rooms per meeting
# Warning: increasing the limit of breakout rooms per meeting
# can generate excessive overhead to the server. We recommend
# this value to be kept under 12.
BREAKOUTROOM_LIMIT=8

# set to false to disable the learning dashboard
ENABLE_LEARNING_DASHBOARD=false

# ====================================
# Tuning
# ====================================
# Default = 2; Min = 1; Max = 4
# On powerful systems with high number of meetings you can set values up to 4 to accelerate handling of events
NUMBER_OF_BACKEND_NODEJS_PROCESSES=4

# Default = 2; Min = 1; Max = 8
# Set a number between 1 and 4 times the value of NUMBER_OF_BACKEND_NODEJS_PROCESSES where higher number helps with meetings
# stretching the recommended number of users in BigBlueButton
NUMBER_OF_FRONTEND_NODEJS_PROCESSES=4


# ====================================
# GREENLIGHT CONFIGURATION
# ====================================

### SMTP CONFIGURATION
# Emails are required for the basic features of Greenlight to function.
# Please refer to your SMTP provider to get the values for the variables below
#SMTP_SENDER_EMAIL=
#SMTP_SENDER_NAME=BigBlueButton-Server
#SMTP_SERVER=
#SMTP_PORT=
#SMTP_DOMAIN=
#SMTP_USERNAME=
#SMTP_PASSWORD=
#SMTP_AUTH=
#SMTP_STARTTLS_AUTO=true
#SMTP_STARTTLS=false
#SMTP_TLS=false
#SMTP_SSL_VERIFY=true

### EXTERNAL AUTHENTICATION METHODS
#
# Not a part of this documentation!
OPENID_CONNECT_CLIENT_ID=bbb-id
OPENID_CONNECT_CLIENT_SECRET=TopSecret
OPENID_CONNECT_ISSUER=https://FQDN
OPENID_CONNECT_REDIRECT=https://FQDN/

# To enable hCaptcha on the user sign up and sign in, define these 2 keys
#HCAPTCHA_SITE_KEY=
#HCAPTCHA_SECRET_KEY=

# Set these if you are using a Simple Storage Service (S3)
# Uncomment S3_ENDPOINT only if you are using a S3 OTHER than Amazon Web Service (AWS) S3.
#S3_ACCESS_KEY_ID=
#S3_SECRET_ACCESS_KEY=
#S3_REGION=
#S3_BUCKET=
#S3_ENDPOINT=

# Define the default locale language code (i.e. 'en' for English) from the fallowing list:
#  [en, ar, fr, es]
DEFAULT_LOCALE=de

```


3. **Do not start the containers now!**

## Internal Reverse Proxy
It does not matter which reverse proxy you use. The following settings are important here:
1. Bind your service to a service IP, protocol https with your internal CA certificate in place
2. Forward the traffic via http to your BBB server on port 48087
3. Websockets **MUST** be enabled! If websockets aren't enabled you get the 3 dots running forever after starting a conference... 

## Proxy settings for the docker daemon
1. Create a folder **/etc/systemd/system/docker.service.d**
2. Create a file **proxy.conf**
3. Edit the file:
```
[Service]
Environment="HTTP_PROXY=http://this.is.my.proxy:8080"
Environment="HTTPS_PROXY=http://this.is.my.proxy:8080"
Environment="NO_PROXY="localhost,127.0.0.1,::1"
```
4. Restart docker daemon or system. 

## Proxy settings for docker container
The container for freeswitch requires a web proxy if, for example, the german language soundfiles have been configured. The container then downloads the corresponding zip file from the internet at startup. When no proxy is defined the container will just fail to start and restart forever. 
However, exceptions must be defined for the remaining containers, as otherwise there may be communication problems or containers such as the mediaserver may have problems with its own health check (Up XXX hours (Unhealthy)). 
So create a config.json in the .docker folder in the home directory of the user starting the containers, e.g. /root/.docker/config.json
It should look like:

```

{
 "proxies": {
   "default": {
     "httpProxy": "http://this.is.my.proxy:8080",
     "httpsProxy": "http://this.is.my.proxy:8080",
     "noProxy": "localhost,0.0.0.0,jodconverter,bbb-web,bbb-pads,freeswitch,nginx,etherpad,mongodb,redis,kurento,webrtc-sfu,fsesl-akka,apps-akka,libreoffice,periodic,webhooks,greenlight,html5,127.0.0.1,127.0.0.0/8,10.7.7.0/24"
   }
 }
}

```

# Prerequisities before starting the containers

## SSL certificates
1. Create a folder named **ssl** under the conf folder in the directory where the BBB files are located.
2. Place your certificate chain in this folder. The format should be ** .crt ** and the certificates must be separated so that each file includes only one certificate.

## Soundfiles
In this case the German soundfiles are used. Because of an issue (#324) you have to place the soundfiles manually in a folder and mount it into the freeswitch container. 
** This is just a workaround and can be ignored after the issue is resolved **
1. Create a folder structure **soundfiles_de\freeswitch-german-soundfiles-master\conference** under the conf folder in the directory where the BBB files are located.
2. Download the German soundfiles from GitHub:
 https://github.com/Daedalus3/freeswitch-german-soundfiles/archive/master.zip
 3. Extract the files into the **conference** folder. Now there should be 4 folders **16000  32000  48000  8000** directly under **conference**. 

## Adjustments to the template file
Adjustments must now be made to the template file so that the certificates and sound files are mounted into the containers. 
You will find the **docker-compose.tmpl.yml** file in the directory where the BBB files are located.
1. Create a copy of the file (just in case): **cp docker-compose.tmpl.yml docker-compose.tmpl.yml.org**
2. Edit or replace docker-compose.tmpl.yml: **nano docker-compose.tmpl.yml**
3. Here is the template file with some comments on the lines that have been added:

```
{{/* if you read this, you can ignore the following lines */}}
# auto generated by ./scripts/generate-compose
# don't edit this directly.
# extensions for custom ssl chain and soundfiles
{{/* -------- */}}

version: '3.6'

# html5 templates
x-html5-backend: &html5backend
  build: 
    context: mod/html5
    additional_contexts:
      - source=./repos/bigbluebutton/bigbluebutton-html5
    args:
      BBB_BUILD_TAG: bbb27-2023-06-13-java17
      TAG_BBB: {{ .Env.TAG_BBB }}
  image: alangecker/bbb-docker-html5:{{ .Env.TAG_BBB }}
  restart: unless-stopped
  depends_on:
    - redis
    - mongodb
    - etherpad
  environment: &html5backend-env
    DOMAIN: ${DOMAIN}
    CLIENT_TITLE: ${CLIENT_TITLE}
    LISTEN_ONLY_MODE: ${LISTEN_ONLY_MODE:-true}
    DISABLE_ECHO_TEST: ${DISABLE_ECHO_TEST:-false}
    AUTO_SHARE_WEBCAM: ${AUTO_SHARE_WEBCAM:-false}
    DISABLE_VIDEO_PREVIEW: ${DISABLE_VIDEO_PREVIEW:-false}
    CHAT_ENABLED: ${CHAT_ENABLED:-true}
    CHAT_START_CLOSED: ${CHAT_START_CLOSED:-false}
    BREAKOUTROOM_LIMIT: ${BREAKOUTROOM_LIMIT:-8}
    #DEV_MODE set to true for this containers to disable SSL check (have not found a better solution yet)
    DEV_MODE: ${DEV_MODE:-true}
    BBB_HTML5_ROLE: backend

x-html5-frontend: &html5frontend
    <<: *html5backend
    volumes:
      - html5-static:/html5-static:rw
    environment: &html5frontend-env
      <<: *html5backend-env
      BBB_HTML5_ROLE: frontend
# =========================

services:
  bbb-web:
    build: 
      context: mod/bbb-web
      additional_contexts:
        - src-web=./repos/bigbluebutton/bigbluebutton-web
        - src-common-message=./repos/bigbluebutton/bbb-common-message
        - src-common-web=./repos/bigbluebutton/bbb-common-web
      args:
        BBB_BUILD_TAG: bbb27-2023-06-13-java17
    image: alangecker/bbb-docker-web:{{ .Env.TAG_BBB }}
    restart: unless-stopped
    depends_on:
        - redis
        - etherpad
        - bbb-pads
    healthcheck:
      test: wget --no-proxy --no-verbose --tries=1 --spider http://10.7.7.2:8090/bigbluebutton/api || exit 1
      start_period: 2m
    environment:
      USE_SYSTEM_CA_CERTS: ${USE_SYSTEM_CA_CERTS:-true}
      DEV_MODE: ${DEV_MODE:-}
      DOMAIN: ${DOMAIN}
      ENABLE_RECORDING: ${ENABLE_RECORDING:-false}
      SHARED_SECRET: ${SHARED_SECRET}
      WELCOME_MESSAGE: ${WELCOME_MESSAGE:-}
      WELCOME_FOOTER: ${WELCOME_FOOTER}
      STUN_SERVER: stun:${STUN_IP}:${STUN_PORT}
      TURN_SERVER: ${TURN_SERVER:-}
      TURN_SECRET: ${TURN_SECRET:-}
      ENABLE_LEARNING_DASHBOARD: ${ENABLE_LEARNING_DASHBOARD:-true}
      NUMBER_OF_BACKEND_NODEJS_PROCESSES: {{ .Env.NUMBER_OF_BACKEND_NODEJS_PROCESSES }}
    # overwrite entrypoint so that ssl certs will be imported before service startup
    entrypoint: ["/bin/sh", "-c", "./__cacert_entrypoint.sh ./entrypoint.sh"]
    volumes:
      - bigbluebutton:/var/bigbluebutton
      - vol-freeswitch:/var/freeswitch/meetings
      # mount certificates into container
      - ./conf/ssl:/certificates:ro
    networks:
      bbb-net:
        ipv4_address: 10.7.7.2


{{ range $i := loop 0 (atoi .Env.NUMBER_OF_BACKEND_NODEJS_PROCESSES) }}
  html5-backend-{{ add $i 1 }}:
    <<: *html5backend
    environment:
      <<: *html5backend-env
      INSTANCE_ID: {{ add $i 1 }}
      PORT: {{ add 4000 $i }}
    networks:
      bbb-net:
        ipv4_address: 10.7.7.{{ add 100 $i }}
{{end}}

{{ range $i := loop 0 (atoi .Env.NUMBER_OF_FRONTEND_NODEJS_PROCESSES) }}
  html5-frontend-{{ add $i 1 }}:
    <<: *html5frontend
    environment:
      <<: *html5frontend-env
      INSTANCE_ID: {{ add $i 1 }}
      PORT: {{ add 4100 $i }}
    networks:
      bbb-net:
        ipv4_address: 10.7.7.{{ add 200 $i }}
{{end}}


  freeswitch:
    container_name: bbb-freeswitch
    build: 
      context: mod/freeswitch
      additional_contexts:
        - freeswitch=./repos/freeswitch/
        - build-files=./repos/bigbluebutton/build/packages-template/bbb-freeswitch-core/
        - fs-config=./repos/bigbluebutton/bbb-voice-conference/config/freeswitch/conf/
      args:
        BBB_BUILD_TAG: bbb27-2023-06-13-java17
    image: alangecker/bbb-docker-freeswitch:{{ .Env.TAG_FREESWITCH }}-{{ .Env.TAG_BBB }}
    restart: unless-stopped
    cap_add:
      - IPC_LOCK
      - NET_ADMIN
      - NET_RAW
      - NET_BROADCAST
      - SYS_NICE
      - SYS_RESOURCE
    environment:
      DOMAIN: ${DOMAIN}
      EXTERNAL_IPv4: ${EXTERNAL_IPv4}
      EXTERNAL_IPv6: ${EXTERNAL_IPv6:-::1}
      SIP_IP_ALLOWLIST: ${SIP_IP_ALLOWLIST:-}
      DISABLE_SOUND_MUTED: ${DISABLE_SOUND_MUTED:-false}
      DISABLE_SOUND_ALONE: ${DISABLE_SOUND_ALONE:-false}
      SOUNDS_LANGUAGE: ${SOUNDS_LANGUAGE:-en-us-callie}
      ESL_PASSWORD: ${FSESL_PASSWORD:-ClueCon}
    volumes:
      - ./conf/sip_profiles:/etc/freeswitch/sip_profiles/external
      - ./conf/dialplan_public:/etc/freeswitch/dialplan/public_docker
      - vol-freeswitch:/var/freeswitch/meetings
      # mount German soundfiles (unzip is missing in container image, just a workaround) -> see issue #324
      - ./conf/soundfiles_de/freeswitch-german-soundfiles-master:/opt/freeswitch/share/freeswitch/sounds/de/de/daedalus3
    network_mode: host
    logging:
      # reduce logs to a minimum, so `docker compose logs -f` still works
      driver: "local"
      options:
        max-size: "10k"
        max-file: "1"
        compress: "false"

  nginx:
    build:
      context: mod/nginx
      additional_contexts:
        - src-learning-dashboard=./repos/bigbluebutton/bbb-learning-dashboard
        - src-playback=./repos/bbb-playback
      args:
        BBB_BUILD_TAG: bbb27-2023-06-13-java17
    image: alangecker/bbb-docker-nginx:1.23-{{ .Env.TAG_PLAYBACK }}-{{ .Env.TAG_BBB }}
    restart: unless-stopped
    depends_on:
      - etherpad
      - webrtc-sfu
      - html5-backend-1
    volumes:
      - bigbluebutton:/var/bigbluebutton
      - html5-static:/html5-static:ro
      - ${DEFAULT_PRESENTATION:-/dev/null}:/www/default.pdf
    network_mode: host
    extra_hosts:
      - "host.docker.internal:10.7.7.1"
      - "bbb-web:10.7.7.2"
      - "etherpad:10.7.7.4"
      - "webrtc-sfu:10.7.7.1"
      - "html5:10.7.7.11"
      - "greenlight:10.7.7.21"

  etherpad:
    build: 
      context: mod/etherpad
      additional_contexts:
        - plugin=./repos/bbb-etherpad-plugin
        - skin=./repos/bbb-etherpad-skin
      args:
        TAG_ETHERPAD: "1.9.1"
    image: alangecker/bbb-docker-etherpad:1.9.1-s{{ .Env.COMMIT_ETHERPAD_SKIN }}-p{{ .Env.COMMIT_ETHERPAD_PLUGIN }}
    restart: unless-stopped
    depends_on:
      - redis
    environment:
      ETHERPAD_API_KEY: ${ETHERPAD_API_KEY}
    networks:
      bbb-net:
        ipv4_address: 10.7.7.4

  bbb-pads:
    build: 
      context: mod/bbb-pads
      additional_contexts:
        - src=./repos/bbb-pads
    image: alangecker/bbb-docker-pads:{{ .Env.TAG_PADS }}
    restart: unless-stopped
    depends_on:
      - redis
      - etherpad
    environment:
      ETHERPAD_API_KEY: ${ETHERPAD_API_KEY}
    networks:
      bbb-net:
        ipv4_address: 10.7.7.18

  redis:
    image: redis:7.2-alpine
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 1s
      timeout: 3s
      retries: 30
    networks:
      bbb-net:
        ipv4_address: 10.7.7.5

  mongodb:
    container_name: bbb-mongodb
    image: mongo:4.4
    restart: unless-stopped
    volumes:
      - ./mod/mongo/mongod.conf:/etc/mongod.conf
      - ./mod/mongo/init-replica.sh:/docker-entrypoint-initdb.d/init-replica.sh
    tmpfs:
      - /data/configdb
      - /data/db
    command: mongod --config /etc/mongod.conf --oplogSize 8 --replSet rs0 --noauth
    healthcheck:
      test: bash -c "if mongo --eval 'quit(db.runCommand({ ping':' 1 }).ok ? 0 ':' 2)'; then exit 0; fi; exit 1;"
    networks:
      bbb-net:
        ipv4_address: 10.7.7.6

  #  TODO: remove as soon as not required anymore by webrtc-sfu
  kurento:
    image: kurento/kurento-media-server:6.18
    restart: unless-stopped
    network_mode: host
    volumes:
      - vol-kurento:/var/kurento

  webrtc-sfu:
    build: 
      context: mod/webrtc-sfu
      additional_contexts:
        - source=./repos/bbb-webrtc-sfu
      args:
        BBB_BUILD_TAG: bbb27-2023-06-13-java17
    image: alangecker/bbb-docker-webrtc-sfu:{{ .Env.TAG_WEBRTC_SFU }}
    restart: unless-stopped
    depends_on:
      - redis
      - freeswitch
      - kurento
    environment:
      CLIENT_HOST: 10.7.7.1
      REDIS_HOST: 10.7.7.5
      FREESWITCH_IP: 10.7.7.1
      FREESWITCH_SIP_IP: ${EXTERNAL_IPv4}
      MCS_HOST: 0.0.0.0
      MCS_ADDRESS: 127.0.0.1
      ESL_IP: 10.7.7.1
      ESL_PASSWORD: ${FSESL_PASSWORD:-ClueCon}
      # TODO: add mediasoup IPv6
      # TODO: can listen to 0.0.0.0 for nat support? https://github.com/versatica/mediasoup/issues/487
    {{ if .Env.EXTERNAL_IPv6 }}
      MS_WEBRTC_LISTEN_IPS: '[{"ip":"{{ .Env.EXTERNAL_IPv6 }}", "announcedIp":"{{ .Env.EXTERNAL_IPv6 }}"}, {"ip":"${EXTERNAL_IPv4}", "announcedIp":"${EXTERNAL_IPv4}"}]'
    {{else}}
      MS_WEBRTC_LISTEN_IPS: '[{"ip":"${EXTERNAL_IPv4}", "announcedIp":"${EXTERNAL_IPv4}"}]'
    {{end}}
      MS_RTP_LISTEN_IP: '{"ip":"0.0.0.0", "announcedIp":"${EXTERNAL_IPv4}"}'
    volumes:
      - vol-mediasoup:/var/mediasoup
    tmpfs:
      - /var/log/bbb-webrtc-sfu
    network_mode: host

  fsesl-akka:
    build: 
      context: mod/fsesl-akka
      additional_contexts:
        - src-common-message=./repos/bigbluebutton/bbb-common-message
        - src-fsesl-client=./repos/bigbluebutton/bbb-fsesl-client
        - src-fsesl-akka=./repos/bigbluebutton/akka-bbb-fsesl
      args:
        BBB_BUILD_TAG: bbb27-2023-06-13-java17
    image: alangecker/bbb-docker-fsesl-akka:{{ .Env.TAG_BBB }}
    restart: unless-stopped
    depends_on:
      - redis
      - freeswitch
    environment:
      FSESL_PASSWORD: ${FSESL_PASSWORD:-ClueCon}
    networks:
      bbb-net:
        ipv4_address: 10.7.7.14

  apps-akka:
    build: 
      context: mod/apps-akka
      additional_contexts:
        - src-common-message=./repos/bigbluebutton/bbb-common-message
        - src-apps-akka=./repos/bigbluebutton/akka-bbb-apps
      args:
        BBB_BUILD_TAG: bbb27-2023-06-13-java17
    image: alangecker/bbb-docker-apps-akka:{{ .Env.TAG_BBB }}
    restart: unless-stopped
    depends_on:
      - redis
    environment:
      DOMAIN: ${DOMAIN}
      SHARED_SECRET: ${SHARED_SECRET}
    volumes:
      - vol-freeswitch:/var/freeswitch/meetings
    networks:
      bbb-net:
        ipv4_address: 10.7.7.15

  jodconverter:
    build: mod/jodconverter
    image: alangecker/bbb-docker-jodconverter:latest
    security_opt:
      - 'no-new-privileges:true'
    restart: unless-stopped
    tmpfs:
      - /tmp
    deploy:
      resources:
        limits:
          memory: 512M
    networks:
      bbb-net:
        ipv4_address: 10.7.7.20

  periodic:
    build: mod/periodic
    image: alangecker/bbb-docker-periodic:v2.7.0
    restart: unless-stopped
    depends_on:
      - mongodb
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - bigbluebutton:/var/bigbluebutton
      - vol-mediasoup:/var/mediasoup
    tmpfs:
      - /var/log/bigbluebutton
    environment:
      ENABLE_RECORDING: ${ENABLE_RECORDING}
      REMOVE_OLD_RECORDING: ${REMOVE_OLD_RECORDING}
      RECORDING_MAX_AGE_DAYS: ${RECORDING_MAX_AGE_DAYS}
    networks:
      bbb-net:
        ipv4_address: 10.7.7.12

{{ if isTrue .Env.ENABLE_RECORDING }}
  # recordings
  recordings:
    build: 
      context: mod/recordings
      additional_contexts:
        - record-core=./repos/bigbluebutton/record-and-playback/core
        - presentation=./repos/bigbluebutton/record-and-playback/presentation
        - bbb-conf=./repos/bigbluebutton/bigbluebutton-config
      args:
        BBB_BUILD_TAG: bbb27-2023-06-13-java17
        TAG_BBB_PRESENTATION_VIDEO: "4.0.3"
    image: alangecker/bbb-docker-recordings:{{ .Env.TAG_BBB }}
    restart: unless-stopped
    depends_on:
      - redis
      - bbb-pads
    environment:
      DOMAIN: ${DOMAIN}
      SHARED_SECRET: ${SHARED_SECRET}
    volumes:
      - bigbluebutton:/var/bigbluebutton
      - vol-freeswitch:/var/freeswitch/meetings
      - vol-mediasoup:/var/mediasoup
      - vol-kurento:/var/kurento
    tmpfs:
      - /var/log/bigbluebutton
      - /tmp
    networks:
      bbb-net:
        ipv4_address: 10.7.7.16
{{end}}

{{ if isTrue .Env.ENABLE_WEBHOOKS }}
  # webhooks
  webhooks:
    build: 
      context: mod/webhooks
      additional_contexts:
        - src=./repos/bbb-webhooks
    image: alangecker/bbb-docker-webhooks:{{ .Env.TAG_WEBHOOKS }}
    restart: unless-stopped
    environment:
      DOMAIN: ${DOMAIN}
      SHARED_SECRET: ${SHARED_SECRET}
    depends_on:
      - redis
    networks:
      bbb-net:
        ipv4_address: 10.7.7.17
{{end}}

{{ if isTrue .Env.ENABLE_HTTPS_PROXY }}
  # https
  https_proxy:
    image: valian/docker-nginx-auto-ssl
    restart: unless-stopped
    volumes:
      - ssl_data:/etc/resty-auto-ssl
    {{ if .Env.EXTERNAL_IPv6 }}
      - ./mod/https/site.conf:/etc/nginx/conf.d/bbb-docker.conf
    {{else}}
      - ./mod/https/site-ipv4only.conf:/etc/nginx/conf.d/bbb-docker.conf
    {{end}}
    {{ if isTrue .Env.DEV_MODE }}
      # allow bbb api access without https
      - ./mod/https/force-https.conf:/usr/local/openresty/nginx/conf/force-https.conf
    {{end}}
    environment:
      {{ if isTrue .Env.DEV_MODE }}
      ALLOWED_DOMAINS: ""
      {{else}}
      ALLOWED_DOMAINS: ${DOMAIN}
      {{end}}
      RESOLVER_ADDRESS: ${RESOLVER_ADDRESS:-9.9.9.9}
    network_mode: host
{{end}}

{{ if isTrue .Env.ENABLE_COTURN }}
  # coturn
  coturn:
    image: coturn/coturn:4.6-alpine
    restart: unless-stopped
    command:
      - "--external-ip=${EXTERNAL_IPv4}/${EXTERNAL_IPv4}"
      - "--external-ip=${EXTERNAL_IPv6:-::1}/${EXTERNAL_IPv6:-::1}"
      - "--static-auth-secret=${TURN_SECRET}"
    volumes:
      {{ if isTrue .Env.ENABLE_HTTPS_PROXY }}
      - ssl_data:/etc/resty-auto-ssl
      {{else}}
      - ${COTURN_TLS_CERT_PATH}:/tmp/cert.pem
      - ${COTURN_TLS_KEY_PATH}:/tmp/key.pem
      {{end}}
      - ./mod/coturn/entrypoint.sh:/usr/local/bin/docker-entrypoint.sh
      - ./mod/coturn/turnserver.conf:/etc/coturn/turnserver.conf
    environment:
      ENABLE_HTTPS_PROXY:
    user: root
    network_mode: host
{{end}}


{{ if isTrue .Env.ENABLE_GREENLIGHT }}
  # greenlight
  greenlight:
    build:
      context: .
    # image version 3.3.1 is needed for environmental variable ALLOW_GREENLIGHT_ACCOUNTS=false to work
    image: bigbluebutton/greenlight:v3.3.1
    # overwrite entrypoint so that ssl certs will be imported before service startup
    entrypoint: ["/bin/sh", "-c"]
    command: [
      "cp /ssl/*.crt /usr/local/share/ca-certificates/ &&
      /usr/sbin/update-ca-certificates &&
      exec bin/start" # exec gives control back to originally entrypoint
    ]
    restart: unless-stopped
    env_file: .env
    depends_on:
      - postgres
      - redis

    environment:
      DATABASE_URL: postgres://postgres:${POSTGRESQL_SECRET:-password}@postgres:5432/greenlight-v3
      REDIS_URL: redis://redis:6379
      {{ if isTrue .Env.DEV_MODE }}
      BIGBLUEBUTTON_ENDPOINT: http://10.7.7.1/bigbluebutton/api
      {{else}}
      BIGBLUEBUTTON_ENDPOINT: https://${DOMAIN}/bigbluebutton/api
      {{end}}
      BIGBLUEBUTTON_SECRET: ${SHARED_SECRET}
      SECRET_KEY_BASE: ${RAILS_SECRET}
      RELATIVE_URL_ROOT: /
    volumes:
       - ./greenlight-data:/usr/src/app/storage
       # mount certificates into container
       - ./conf/ssl:/ssl:ro
    networks:
      bbb-net:
        ipv4_address: 10.7.7.21

  postgres:
    image: postgres:12-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: greenlight-v3
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: ${POSTGRESQL_SECRET:-password}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - ./postgres-data:/var/lib/postgresql/data
    networks:
      bbb-net:
        ipv4_address: 10.7.7.22
{{end}}

{{ if isTrue .Env.ENABLE_PROMETHEUS_EXPORTER }}
  # prometheus
  prometheus-exporter:
    image: greenstatic/bigbluebutton-exporter:latest
    restart: unless-stopped
    environment:
      API_BASE_URL: http://10.7.7.1:48087/bigbluebutton/api/
      API_SECRET: ${SHARED_SECRET}
      RECORDINGS_METRICS_READ_FROM_DISK: "${ENABLE_PROMETHEUS_EXPORTER_OPTIMIZATION:-false}"
    networks:
      bbb-net:
        ipv4_address: 10.7.7.33
    {{ if isTrue .Env.ENABLE_PROMETHEUS_EXPORTER_OPTIMIZATION }}
    volumes:
      - bigbluebutton:/var/bigbluebutton:ro
    {{end}}

    # the exporter requires /etc/bigbluebutton/bigbluebutton-release
    tmpfs:
      - /etc/bigbluebutton:mode=777
    entrypoint: sh -c 'echo "BIGBLUEBUTTON_RELEASE=2.7.3" > /etc/bigbluebutton/bigbluebutton-release && python server.py'
{{end}}


volumes:
  bigbluebutton:
  vol-freeswitch:
  vol-kurento:
  vol-mediasoup:
  html5-static:
{{ if isTrue .Env.ENABLE_HTTPS_PROXY }}
  ssl_data:
{{end}}

networks:
  bbb-net:
    ipam:
      driver: default
      config:
        - subnet: "10.7.7.0/24"


```
## Startup
With the new template file in place you can start your containers by running:
1. ./scripts/generate-compose
2. docker compose up -d --no-build
3. Get a coffee, sit back and wait until the containers have all been downloaded and started
4. Now all the required Docker containers should be running. BigBlueButton listens to port 48087. 

## What was not tested?
1. Recording
2. Webhooks
3. Prometheus Exporter
4. Coturn & HTTPS Proxy (makes no sense here!!!)

