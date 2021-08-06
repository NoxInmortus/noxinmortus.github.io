---
layout: post
title: Discord Webhook
date: 2021-08-06
---

Hello,

Petite note technique sur un peu d'automatisation.

Contexte : Association qui organise des évènements mensuels, le dernier weekend du mois, et ce avec une alternance samedi / dimanche. Un serveur/canal Discord est utilisé pour envoyer l'annonce de chaque évènement par un membre du Bureau. Le problème étant que le message est envoyé quand quelqu'un pense à le faire, donc pas forcément à la même fréquence, parfois un peu tard, pas forcément avec le même format de message, etc.

J'ai donc utilisé un peu de bash, python, cron et un WebHook Discord pour réaliser tout ceci. Je ne développerais pas le webhook, on l'obtient en trois clics depuis Discord (premier lien dans les sources).

Le premier bloc à résoudre était d'obtenir la date du dernier samedi et du dernier dimanche de chaque mois afin de pouvoir envoyer cette date dans l'annonce.

J'ai déniché un bout de code Python qui faisait pratiquement tout le job (deuxième lien dans les sources), et l'ai très légèrement adapté afin que l'année soit récupérée depuis le système hôte, et que son seul et unique argument soit le "numéro du jour de la semaine dans l'ordre inverse". Donc que `Dimanche = 1`, `Samedi = 2`, etc. Le reste est assez facilement lisible :

```python
# last.day.py
#!/usr/bin/env python3

# First and only parameter is the day number you want in reverse order.
# Examples: Sunday is 1, saturday is 2, etc.

import sys, calendar
from datetime import date

year = date.today().year
day=1

if len(sys.argv) > 1:
    try:
        day = int(sys.argv[-1])
    except ValueError:
        pass

for month in range(1, 13):
    last_day = max(week[-day] for week in calendar.monthcalendar(year, month))
    print('{}-{}-{:2}'.format(year, calendar.month_abbr[month], last_day))
```

Vient ensuite le script bash un peu moche et fait à la va-vite mais qui permet de faire le reste du travail (je suis parti du troisième lien des sources, c'est d'ailleurs ce dernier lien qui m'a donné envie de faire tout ceci):

```bash
# discord-webhook-event.sh
#!/usr/bin/env bash
set -eu

# This script is used to send Discord messages through Discord's webhooks
# Its purpose is to send a monthly announce, then one or more reminder for the next event
# This script accept from one to three optional arguments:
# - First argument is the "WANTED_DAY" which is the day number you want in reverse order. Default is '1'
# Examples: Sunday is 1, saturday is 2, etc. This is used to fetch the event full date.
# - Second argument can be a string 'reminder' or 'test'.
# 'reminder' is used to send a reminder message instead of the original event message
# 'test' message is used to print the message that would be send
# - Third argument is the string 'test' in case you want to use 'reminder' and 'test' at the same time

WANTED_DAY=${1:-1}
USERNAME=Annonceur
## Put your webhook here
WEBHOOK=''

# Convert WANTED_DAY as string
case ${WANTED_DAY} in
  '1' )
  DAY_STRING='dimanche'
  ;;
  '2' )
  DAY_STRING='samedi'
  ;;
esac

# Cron host is in EN so for the tests sake we ensure to always be in EN
LC_TIME="en_US.UTF-8"

# Fetch the full date of the last sunday or saturday through another python script as it is far easier
LAST_DAY=$(last.day.py "${WANTED_DAY}"| grep "$(date '+%b')")

# Extract day number
DAY_NUMBER=$(echo "${LAST_DAY}" | awk -F '-' '{print $NF}')

# As we send the message in French, we fall back to FR
LC_TIME="fr_FR.UTF-8"

# Our message
MESSAGE="Hello @everyone, prochaine partie à XXXX le ${DAY_STRING} ${DAY_NUMBER} $(date '+%b'), RDV 10h la-haut. Pour organiser du covoiturage, merci d'en parler sur #public !
:sun_with_face: = Je serais présent le matin seulement
:last_quarter_moon_with_face: = Je serais présent l'après-midi seulement
:rainbow: = Je serais présent toute la journée
:x: = Je serais absent
:one: = Je ramène une personne de plus
:two: = Je ramène deux personnes de plus
etc."

if [[ ${2:-''} == 'reminder' ]]; then
  MESSAGE="Hello @everyone, ceci est un rappel pour la prochaine partie à XXXX le ${DAY_STRING} ${DAY_NUMBER} $(date '+%b') !"
fi

if [[ ${2:-''} == 'test' ]] || [[ ${3:-''} == 'test' ]] ; then
  echo "${MESSAGE}"
  exit 0
fi

curl -X POST \
    -F "content=${MESSAGE}" \
    -F "username=${USERNAME}" \
    "${WEBHOOK}"
```

Et enfin le tout lancé via des crons depuis un RaspberryPi :
```
# /etc/cron.d/discord
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin
#Ansible: sunday events
0 17 1 1-11/2 * root  /usr/local/bin/discord-webhook-event.sh 1 >/dev/null 2>&1
#Ansible: saturday events
0 17 1 */2 * root  /usr/local/bin/discord-webhook-event.sh 2 >/dev/null 2>&1
#Ansible: sunday events reminder
0 17 15,24 1-11/2 * root  /usr/local/bin/discord-webhook-event.sh 1 reminder >/dev/null 2>&1
#Ansible: saturday events reminder
0 17 15,24 */2 * root  /usr/local/bin/discord-webhook-event.sh 2 reminder >/dev/null 2>&1
```

# Sources:
- <https://support.discord.com/hc/fr/articles/228383668-Utiliser-les-Webhooks>
- <https://rosettacode.org/wiki/Find_the_last_Sunday_of_each_month#Python>
- <https://christine.website/blog/howto-automate-discord-webhook-cron-2018-03-29>

A+
