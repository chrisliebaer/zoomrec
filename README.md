# Warning
This is a fork of [zoomrec](https://github.com/kastldratza/zoomrec) by [kastldratza](https://github.com/kastldratza). This readme is outdated and this repo contains only my personal additions, please consider improving the original, rather than my fork.

---

<h1 align="center">
    zoomrec	
</h1>

<h4 align="center">
	A all-in-one solution to automatically join and record Zoom meetings.
</h4>

---

- **Python3** - _Script to automatically join Zoom meetings and control FFmpeg_
- **FFmpeg** - _Triggered by python script to start/stop screen recording_
- **Docker** - _Headless VNC Container based on Ubuntu 20.04 with Xfce window manager and TigerVNC_
- **Telegram** - _Get notified about your recordings_

---

Join with ID and Passcode           |  Join with URL
:-------------------------:|:-------------------------:
![](doc/demo/join-meeting-id.gif)  |  ![](doc/demo/join-meeting-url.gif)

---

## Installation

The entire mechanism runs in a Docker container. So all you need to do is install Docker and use the image from
Registry.

### Requirements

- Docker - [https://docs.docker.com/get-docker/]()

### Docker Registry

Docker images are build and pushed automatically to [**Docker
Hub**](https://hub.docker.com/repository/docker/kastldratza/zoomrec) and [**GitHub Container
Registry**](https://github.com/kastldratza/zoomrec/pkgs/container/zoomrec).

So you can choose and use one of them:

- ```ghcr.io/kastldratza/zoomrec:master```
- ```kastldratza/zoomrec:latest```

*For my examples in this README I used* ```kastldratza/zoomrec:latest```

---

## Usage

- Container saves recordings at **/home/zoomrec/recordings**
- The current directory is used to mount **recordings**-Folder, but can be changed if needed
    - Please check use of paths on different operating systems!
    - Please check permissions for used directory!
- Container stops when Python script is terminated
- Zoomrec uses a CSV file with entries of Zoom meetings to record them
    - The csv can be passed as seen below (mount as volume or add to docker image)
- To "say" something after joining a meeting:
    - ***paplay*** (*pulseaudio-utils*) is used to play a sound to a specified microphone output, which is mapped to a
      microphone input at startup.
    - ***paplay*** is triggered and plays a random file from **/home/zoomrec/audio**
    - Nothing will be played if directory:
        - does not contain **.wav** files
        - is not mounted properly

### CSV structure

CSV must be formatted as in example/meetings.csv

- Delimiter must be a semicolon "**;**"
- Only meetings with flag "**record = true**" are joined and recorded
- "**description**" is used for filename when recording
- "**duration**" in minutes (+5 minutes to the end)

weekday | time | duration | id | password | description | record
-------- | -------- | -------- | -------- | -------- | -------- | --------
monday | 09:55 | 60 | 111111111111 | 741699 | Important_Meeting | true
monday | 14:00 | 90 | 222222222222 | 321523 | Unimportant_Meeting | false
tuesday| 17:00 | 90 | https://zoom.us/j/123456789?pwd=abc || Meeting_with_URL | true

### VNC

You can connect to zoomrec via vnc and see what is happening.

#### Connect (default)

Hostname | Port | Password
-------- | -------- | --------
localhost   | 5901   | zoomrec

### Telegram
Zoomrec can notify you via Telegram about starting and ending a recording or if joining a meeting failed. All you need is a bot token and a chat id of a Telegram channel.

At first [create a new telegram bot](https://core.telegram.org/bots#6-botfather) to get the bot token. After that create a new channel and add the bot with sufficient permissions to write messages in that channel. Finally [get the chat id of your channel](https://gist.github.com/mraaroncruz/e76d19f7d61d59419002db54030ebe35) and look below how to pass your Telegram details to Zoomrec.

### Preparation

To have access to the recordings, a volume is mounted, so you need to create a folder that container users can access.

**[ IMPORTANT ]**

#### Create folders and set permissions (on Host)

```
mkdir -p recordings/screenshots
chown -R 1000:1000 recordings

mkdir -p audio
chown -R 1000:1000 audio
```

### Flags

#### Set timezone (default: Europe/Berlin)

```
docker run -d --restart unless-stopped \
  -e TZ=Europe/Berlin \
  -v $(pwd)/recordings:/home/zoomrec/recordings \
  -v $(pwd)/example/audio:/home/zoomrec/audio \
  -v $(pwd)/example/meetings.csv:/home/zoomrec/meetings.csv:ro \
  -p 5901:5901 \
kastldratza/zoomrec:latest
```

#### Set debugging flag (default: False)

- screenshot on error
- record joining
- do not exit container on error

```
docker run -d --restart unless-stopped \
  -e DEBUG=True \
  -v $(pwd)/recordings:/home/zoomrec/recordings \
  -v $(pwd)/example/audio:/home/zoomrec/audio \
  -v $(pwd)/example/meetings.csv:/home/zoomrec/meetings.csv:ro \
  -p 5901:5901 \
kastldratza/zoomrec:latest
```
#### Set Telegram details
```
docker run -d --restart unless-stopped \
  -e TELEGRAM_BOT_TOKEN="YOUR_BOT_TOKEN" \
  -e TELEGRAM_CHAT_ID="-100_YOUR_CHAT_ID" \
  -v $(pwd)/recordings:/home/zoomrec/recordings \
  -v $(pwd)/example/audio:/home/zoomrec/audio \
  -v $(pwd)/example/meetings.csv:/home/zoomrec/meetings.csv:ro \
  -p 5901:5901 \
kastldratza/zoomrec:latest
```
#### Set Zoom display name 
```
docker run -d --restart unless-stopped \
  -e DISPLAY_NAME="zoomrec" \
  -v $(pwd)/recordings:/home/zoomrec/recordings \
  -v $(pwd)/example/audio:/home/zoomrec/audio \
  -v $(pwd)/example/meetings.csv:/home/zoomrec/meetings.csv:ro \
  -p 5901:5901 \
kastldratza/zoomrec:latest
```
### Windows / _cmd_

```cmd
docker run -d --restart unless-stopped \
  -v %cd%\recordings:/home/zoomrec/recordings \
  -v %cd%\example\audio:/home/zoomrec/audio \
  -v %cd%\example\meetings.csv:/home/zoomrec/meetings.csv:ro \
  -p 5901:5901 \
kastldratza/zoomrec:latest
```

### Windows / _PowerShell_

```powershell
docker run -d --restart unless-stopped \
  -v ${PWD}/recordings:/home/zoomrec/recordings \
  -v ${PWD}/example/audio:/home/zoomrec/audio \
  -v ${PWD}/example/meetings.csv:/home/zoomrec/meetings.csv:ro \
  -p 5901:5901 \
kastldratza/zoomrec:latest
```

### Linux / macOS

```bash
docker run -d --restart unless-stopped \
  -v $(pwd)/recordings:/home/zoomrec/recordings \
  -v $(pwd)/example/audio:/home/zoomrec/audio \
  -v $(pwd)/example/meetings.csv:/home/zoomrec/meetings.csv:ro \
  -p 5901:5901 \
kastldratza/zoomrec:latest
```

## Customization example

### Add meetings.csv to image

```bash
# Switch to example directory
cd example

# Build new image by customized Dockerfile
docker build -t kastldratza/zoomrec-custom:latest .

# Run image without mounting meetings.csv and audio directory
# Linux
docker run -d --restart unless-stopped -v $(pwd)/recordings:/home/zoomrec/recordings -p 5901:5901 kastldratza/zoomrec-custom:latest

# Windows / PowerShell
docker run -d --restart unless-stopped -v ${PWD}/recordings:/home/zoomrec/recordings -p 5901:5901 kastldratza/zoomrec-custom:latest

# Windows / cmd
docker run -d --restart unless-stopped -v %cd%\recordings:/home/zoomrec/recordings -p 5901:5901 kastldratza/zoomrec-custom:latest
```

---

## Supported actions

- [x] Show when the next meeting starts
- [x] _Join a Meeting_ from csv with id and password
- [x] Wrong error: _Invalid meeting ID_ / **Leave**
- [x] _Join with Computer Audio_
- [x] _Please wait for the host to start this meeting._
- [x] _Please wait, the meeting host will let you in soon._
- [x] _Enter Full Screen_
- [x] Switch to _Speaker View_
- [x] Continuously check: _This meeting is being recorded_ / **Continue**
- [x] Continuously check: _Hide Video Panel_
- [x] Continuously check: _This meeting has been ended by host_
- [x] Quit ffmpeg gracefully on abort
- [x] _Host is sharing poll results_
- [x] _This meeting is for authorized attendees only_ / **Leave meeting**
- [x] Play sound after joining a meeting
- [x] _Join a Meeting_ from csv with url

---

## Roadmap

- [ ] Refactoring
- [ ] Create terraform stack to deploy in AWS
- [ ] _Join a Meeting_ from calendar
- [ ] _Sign In_ to existing Zoom account
- [ ] _Join Breakout Room_
- [ ] Support to record Google Meet, MS Teams, Cisco WebEx calls too
- [ ] Ability to monitor recordings sessions in various containers

---

## Testing

Create unittests for different use cases:

- [ ] Join meeting
- [ ] Start / Stop ffmpeg and check if file was created
- [ ] ...

