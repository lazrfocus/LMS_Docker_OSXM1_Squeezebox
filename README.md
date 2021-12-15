# Music Server
2021-12-14

## Logitech Media Server
### W/ local Squeezebox player

Run the following docker compose with logitech media server. 
Modify the volumes that are mounted to the docker directories to match your local setup.  For now, I put my playlists in a different directory so my music directory stays read only.

```yaml
version: '3.7'
services:
  lms:
    container_name: lms
    image: lmscommunity/logitechmediaserver
    volumes:
      - /media/user/Backup_1/apps/logitechmediaserver/config:/config:rw
      - /media/user/Backup_1/music:/music:ro
      - /media/user/Backup_1/apps/logitechmediaserver/playlists:/playlist:rw
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
    restart: always
    environment:
      - PUID=1000
      - PGID=1001
      - HTTP_PORT=9337 #set new web browser port instead of 9000
    #network_mode: "host" # (chromecast/airplay)
    ports: #comment this section out if using network_mode = "host"
      - 9337:9337
      - 9338:9090
      - 3483:3483  #for now using default 3483 port because of ipeng mobile app compatibility questions
  squeezelite-tailscale:
    image: giof71/squeezelite:stable
    container_name: squeezelite-tailscale
    devices:
      - /dev/snd:/dev/snd
    environment:
      - SQUEEZELITE_NAME=squeezelite #Display name of the playback system listed in the webUI
      - SQUEEZELITE_AUDIO_DEVICE=surround21:CARD=K6,DEV=0  #change this to your specific audio card.  The squeezelite docker log will list your audio devices on startup if you arent sure of the formatting.  The above is a format for a Native Instruments Komplete 6 USB audio device on Ubuntu.
      #- SQUEEZELITE_SERVER_PORT=192.168.1.7:3483 #use if network_mode=host (for chromecast/airplay support)
      - SQUEEZELITE_SERVER_PORT=lms:3483 #use if not using host network mode
    restart: unless-stopped
    #network_mode: "host"   #i honestly forget if this is needed, experiment
```

This docker compose file should start a webserver on http://localhost:9337

I recommend using Portainer to setup and control the docker by pasting the above docker-compose code into a new "stack".

**Squeezelite** is a background service that is always connected to the logitech media server waiting to playback whatever the server feeds it

iPhone playback:
- Purchase $8.99 + $4.99 iPeng app to access LMS and have synchronous playback to the iphone
	- Sync doesnt work good with wireless headphones due to latency issues

OSX playback:
- On an M1 chip (non-x86/x64), there is practically no information on running squeezelite
	- *Was unable to build the git repo from scratch because of library linking issues.  Got pretty far with missing header files, but at the end it still couldnt find the -Lxxx linker files*
- Take the pre-built app for M1 on sourceforge.  The .app file located in the DMG will not work out of the box.
	- Open the .app package contents and take out the squeezelite binary file.
		- You can also try to edit the .plist attributes file to run the squeezelite player with the app icon, but I have had mixed results
		- To enable running the .app without OSX complaining the file is damaged or other security issues, run the following command:
```bash 
xattr -r -d com.apple.quarantine SqueezePlay.app>)
``` 

- Run the binary like this to enable playback on OSX
```bash
./squeezelite -a 1000:1 -f ~/.squeeze.log -s 192.168.1.7:3483 -o 1 -C 2 -D 500 -n MBA
```

- Access the player from the web browser remote, or from iPhone app.

## For lazy, The squeezelite binary file from the .app file:
![[Squeezelite_1392_appandbin.zip]]

### UPNP / Chromecast / Airplay:
- I was only able to see chromecast and UPNP devices by putting my docker behind the network_mode = "host" to share the local IP instead of creating a containerized network
	- Most likely due to how it uses ports, unable to look into this any further


## Links
[Logitech Media Server Git Repo](https://github.com/Logitech/slimserver)
[Squeezelite Git Repo from Ralph Irving](https://github.com/ralph-irving/squeezelite)
[Squeezelite Binaries not on github for some reason](https://sourceforge.net/projects/lmsclients/files/squeezelite/)
[Portainer](https://github.com/portainer/portainer/)



