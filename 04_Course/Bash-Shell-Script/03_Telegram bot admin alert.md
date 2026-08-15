---
date: 2025-12-13
tags:
  - course
  - bash
  - shell-script
  - linux
---
## Create Telegram bot
Telegram app
chat 에서 BotFather 검색 & 추가

## Get API key & ID
다음 채팅 순서대로 입력 (이름과 아이디는 맘대로)
`/newbot`
`Bashbomb`
`Bashbomb_0_bot`

API 복사해서 browser켜고 `https://api.telegram.org/bot{{api}}/getUpdates` 접속
`{"ok":true,"result":[]}` 가 떠야함
이대로 안뜨면 채팅에서 `/token` 입력하여 API키 다시 받고 재접속

마지막 채팅에서 보이는 `t.me/...`로 시작하는 채팅방 입장하고 암거나 전송

브라우저로 돌아가서 `"from":{"id":...` 로 된 부분 찾아서 id 복사

## Write shell script
tel_push.sh
```bash
#!/bin/bash

if [ $# -ne 2 ]
then
	echo 
	echo "Usage "
	echo "$0 {HOSTNAME} {MESSAGE}"
	echo
	echo "example) "
	echo "$0 "
fi

ID="8304225411"
API_TOKEN="8364512989:AAHDnqqarqUVsOKH8v5GdcM879LdVfdceZ8"
URL="https://api.telegram.org/bot${API_TOKEN}/sendMessage"

DATE="$(date "+%Y-%m-%d %H:%M")"

TEXT="${DATE} [$1] $2"

curl -s -d "chat_id=${ID}&text=${TEXT}" ${URL} > /dev/null	
```

log_mon.sh
```bash
#!/bin/bash

DIR=""
SIZE="$(du -m ${DIR} | awk '{print $1}')"
HOST="$(hostname)"

if [ ${SIZE} -ge 0 ]
then
	TEXT="${DIR} usage is over 0MB"
	/root/SHELL/monitor/tel_push.sh "${HOST}" "${TEXT}"
fi
```

