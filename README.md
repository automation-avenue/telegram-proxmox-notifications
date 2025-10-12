### telegram-proxmox-notifications <br />
List of commands used to send notifications from Proxmox / Debian / other Linux distros to Telegram application <br />
You need to create a new bot in Telegram using /newbot in @BotFather <br />
Copy the HTTP API token for your bot. <br />
Go to https://api.telegram.org/bot<API token>/getUpdates in your browser to get Chat ID for your Telegram channel / bot <br />
You can get all instructions on https://core.telegram.org/bots/api#authorizing-your-bot <br />

Now let's build a bash script for our Proxmox / other linux distro. <br />
First install `lm-sensors` application by running `apt install lm-sensors` <br />
Then run command `sensors-detect` <br />
After that you run just `sensors` to see current temperatures. <br />
Run command below: <br />
`sensors | grep Core | awk '{print $3}' | sed 's/[+]\([0-9]\+\)\..*/\1/' | sort -nr | head -n 1` -command to see if you get the desired output ( just a temp of hottest core ) <br />

First script with notify_teams function to test connectivity <br />

```bash
#!/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/games:/usr/local/sbin:/usr/local/bin

TELEGRAM_ENDPOINT='https://api.telegram.org/bot<token>/sendMessage'
CHAT_ID='<chat_id>'
set -x

notify_teams(){
	curl --request 'POST' --url $TELEGRAM_ENDPOINT --header 'Content-Type: application/json' --data '{ "chat_id": '"$CHAT_ID"', "text": "This is a test message" }'
}

notify_teams
```

Add execute permission with `chmod +x telegram.sh` command <br />
Second script that runs temps but not in cron: <br />

```bash
#!/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/games:/usr/local/sbin:/usr/local/bin

TELEGRAM_ENDPOINT='https://api.telegram.org/bot<token>/sendMessage'
CHAT_ID='<chat_id>'
CPU_TEMP=$(sensors | grep Core | awk '{print $3}' | sed 's/[+]\([0-9]\+\)\..*/\1/' | sort -nr | head -n 1)
set -x

notify_teams(){
	curl --request 'POST' --url $TELEGRAM_ENDPOINT --header 'Content-Type: application/json' --data '{ "chat_id": '"$CHAT_ID"', "text": "WARNING: CPU temp is '"$CPU_TEMP"'°C"}'
}

notify_teams
```

Last script and cronjob <br />

```bash
root@pve:/tmp# cat telegram.sh 
#!/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin:/usr/games:/usr/local/sbin:/usr/local/bin

TELEGRAM_ENDPOINT='https://api.telegram.org/bot<token>/sendMessage'
CHAT_ID='<chat_id>'
CPU_TEMP=$(sensors | grep Core | awk '{print $3}' | sed 's/[+]\([0-9]\+\)\..*/\1/' | sort -nr | head -n 1)

if  [ "$CPU_TEMP" -gt 65 ];
 then
  curl --request 'POST' --url $TELEGRAM_ENDPOINT --header 'Content-Type: application/json' --data '{ "chat_id": '"$CHAT_ID"', "text": "WARNING: CPU temp is '"$CPU_TEMP"'°C"}'
fi
```


`crontab -l` to list current cron jobs <br />
`crontab -e` to edit list of cron jobs <br />
Our cron: <br />
`* * * * * /tmp/telegram.sh` <br />
Install stress-ng with `apt install stress-ng -y` 
Run stress test with stress-ng: <br />
`stress-ng --cpu 4 --timeout 60s --verbose` <br />
