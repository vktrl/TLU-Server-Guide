# Mugavam ligipääs Greenyle Windows, macOS ja Linuxi masinatest

Kõikvõimalikud viisid kuidas oma elu ülikooli serveritega suheldes mugavamaks teha.

* Ühendame Greenyga üle SSH tunneli
* Lõpetame paroolide trükkimise
* Brausime Greenys asuvaid veebilehti oma koduarvutis
* SFTP kasutamine
* Mountime Greeny oma koduse masina failisüsteemi lihtsamaks ligipääsuks
* Muuhulgas õpime kasutama Windowsi Linuxi alamsüsteemi

# Juhised Windowsi kasutajatele

## 1) Paigaldame Tarkvara

### Linux subsystem

Enamasti soovitatakse kasutada PuTTY-t, kuid Windows 10 poolt pakutav WLS (Windows Linux Subsystem) on palju võimsam ja mugavam meetod teiste Linuxi baasil masinatega (ehk siis enamus serverid maamunal) suhtlemiseks.
Informaatikud peaksid end käsureal mugavalt tundma.

1) Vajuta `Winkey+X` ja vali Windows Powershell (Admin)  
2) Kirjuta  
`````````Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux`````````  
3) Ava Microsoft Store ja paigalda endale sobiv distro - soovitan Ubuntut  
4) Pärast installi ava Ubuntu rakendus ja seadista oma kasutajanimi jms  

#### WinFsp ja SSHFS windowsile (vajalik eksperimentaalsete featuuride jaoks allpool)

1) laadi alla ja installi  
https://github.com/billziss-gh/winfsp/releases/  
https://github.com/billziss-gh/sshfs-win/releases/

#### WinSCP

SFTP klient failide üles ja alla laadimiseks, juhuks kui minu poolt pakutavad eksperimentaalsed featuurid liiga hirmsad on või mingil põhjusel ei tööta.

https://winscp.net/eng/download.php

## 2) Seadistame mugava autentimise RSA võtmepaariga

### Võtmete loomine

Ava Ubuntu rakendus ja toimeta järgnevalt:

1. Loome uue võtmepaari. Key location jäta samaks, passphrase pole hetkel vajalik, kuna me ei turva kriitilise tähtsusega serverit  
```ssh-keygen -t rsa -b 4096```
2. Paigaldame oma värske avaliku võtme lin2.tlu.ee serverisse. Sisesta oma TLU parool, kui küsitakse  
```ssh-copy-id -i ~/.ssh/id_rsa *tlu-kasutaja*@lin2.tlu.ee```
3. Loome tunneli, et sama võti Greeny-sse paigaldada  
```ssh *tlu-kasutaja*@lin2.tlu.ee -L 2222:greeny.cs.tlu.ee:22```
4. Avame *uue* Ubuntu akna ja paigaldame võtme Greeny serverisse. Kasuta oma Greeny parooli  
```ssh-copy-id -i ~/.ssh/id_rsa -p 2222 *greeny-kasutaja*@localhost```
5. Sulge uus aken ja logi lin2 serverist välja. Kirjuta mõlemas aknas:  
```exit```

Edaspidi pole sul vaja paroole trükkida :)

### ssh-agent seadistus (valikuline)

SSH agent suunab sinu masinas oleva võtme edasi, kui ühendad masinaga üle teise masina (nt. lin2.tlu.ee shellist greeny.cs.tlu.ee ühendades). Kui kasutad tunneleid, pole seda vaja.
WSL tühistab akna sulgemisel taustprotsessid, seega peame agendi akna avamisel automaatselt kävitama ja hoolitsema selle eest, et neid oleks ainult 1. Kasutame lahti jäänud akent ja seadistame agendi.

1. Avame .bashrc  
```nano ~/.bashrc```
2. Lisame alloleva skripti faili lõppu  
```
#!/bin/bash
 
 # Set up ssh-agent
 SSH_ENV="$HOME/.ssh/environment"
 
 function start_agent {
     echo "Initializing new SSH agent..."
     touch $SSH_ENV
     chmod 600 "${SSH_ENV}"
     /usr/bin/ssh-agent | sed 's/^echo/#echo/' >> "${SSH_ENV}"
     . "${SSH_ENV}" > /dev/null
     /usr/bin/ssh-add
 }
 
 # Source SSH settings, if applicable
 if [ -f "${SSH_ENV}" ]; then
     . "${SSH_ENV}" > /dev/null
     kill -0 $SSH_AGENT_PID 2>/dev/null || {
         start_agent
     }
 else
     start_agent
 fi
```
3. Salvestame faili `Ctrl+O` ja väljume `Ctrl+X`  
4. Laeme uue .bashrc  
````source ~/.bashrc````

## 3) Loome ühenduse ülikooli serveriga

Suuname kohalikud pordid. See võimaldab meil välisvõrgust lin2 vahendusel Greenyga suhelda.

1. Loome lin2 ühenduse ja suuname kohalikud pordid Greeny HTTP ja SSH pihta  
```ssh *TLU-kasutaja*@lin2.tlu.ee -L 2222:greeny.cs.tlu.ee:22 -L 8888:greeny.cs.tlu.ee:80```

Localhost pordid 2222 ja 8888 on nüüd suunatud. *See aken peab jääma avatuks senikaua kuni tahad Greeny ühendust kasutada!*

## 4) Greeny kasutamine üle tunneli

Kõik pöördumised teeme edaspidi locahost või 127.0.0.1 poole

### SSH
Terminalis
```ssh -p 2222 *greeny-kasutaja*@localhost```

#### Veeb

Ava brauseris
```http://localhost:8888/~*sinu-greeny-kasutaja*```

#### SFTP (WinSCP)
Seadistus:
```
File protocol: SFTP
Hostname: localhost
Port: 2222
User name: greeny-kasutaja
```

Kui tahad kasutada parooli asemel võtit, siis kopeeri oma võti WLSist üle Windowsi failisüsteemi  
```cp ~/.ssh/id_rsa /mnt/c/Users/Kasutajanimi/Documents/tlu_rsa```  
Vali kopeeritud fail WinSCP login aknas
```Advanced... > SSH > Athentication > Private key file: ______```

#### Eksperimentaalne värk

SFTP asemel on võmalik Greeny kataloog otse oma kohalikku failisüsteemi mountida ja töötada failidega otse, mitte neid üles/alla laadides.
WinFsp ja sshfs-win peavad olema installitud.

1. Ava file exploreris This PC ja vali "Map network drive"
2. Folder väljale kirjuta:  
```\\sshfs\*sinu-windowsi-kasutaja*=*greeny-kasutaja*@localhost!2222```

Sinu Greeny kataloog on nüüd võrgukettana sinu arvutis

P.S. Atom editor on üle SSHFSi aeglane. Ilmselt git muudatuste trackimise pärast vms.

# macOS ja Linux

Juhised macOS töötavad ka Linuxi masinates. Võib-olla pead käske ümber korraldama, aga kui sa Linuxit töölauana kasutad, siis peaksid sellega ilusti hakkama saama.

## 1) Paigaldame tarkvara

### macOS

Hangi FUSE ja SSHFS:  
https://osxfuse.github.io
SSHFS 2.5.0 on vana, aga töötab. Uuema versiooni (pead ise kompileerima) saad siit: https://github.com/libfuse/sshfs

### Linux

Debiani baasil distrotes (Ubuntu, Mint, jms)

1. Installi SSHFS  
```sudo apt-get update && sudo apt-get -y install sshfs```

## 2) Seadistame mugava autentimise RSA võtmepaariga

### Võtmete loomine

Ava Terminal ja toimeta järgnevalt:

1. Loome uue võtmepaari. Key location jäta vaikimisi, passphrase pole hetkel vajalik, kuna me ei turva kriitilise tähtsusega serverit  
```ssh-keygen -t rsa -b 4096```
2. Paigaldame oma värske avaliku võtme lin2.tlu.ee serverisse. Sisesta oma TLU parool, kui küsitakse  
```ssh-copy-id -i ~/.ssh/id_rsa *tlu-kasutaja*@lin2.tlu.ee```
3. Loome tunneli, et sama võti Greeny-sse paigaldada  
```ssh tlu-kasutaja@lin2.tlu.ee -L 2222:greeny.cs.tlu.ee:22```
4. Avame *uue* Terminali akna ja paigaldame võtme Greeny serverisse. Kasuta oma Greeny parooli  
```ssh-copy-id -i ~/.ssh/id_rsa -p 2222 *greeny-kasutaja*@localhost```
5. Sulge uus aken ja logi lin2 serverist välja. Selleks kirjuta mõlemas aknas:  
```exit```

#### ssh-agent (valikuline)

Ma ei mäleta täpseid protsesse, aga macOS peal käis asi umbes nii, et ```ssh-add -K``` lisab parooliga genereeritud võtmed keychaini ja lisades ```ssh-add -A 2>/dev/null;``` oma .bash_profile faili tõmmatakse võtmed automaatselt sisse. Vähemalt mul tundub töötavat ja mul ei ole huvi oma arvutit puhtaks teha, et asja algusest peale testida. Tunneleid tehes pole seda enivei vaja.
Linuxi jaoks googelda ```ssh-agent linux```

## 2) Loome ühenduse ülikooli serveriga
Suuname kohalikud pordid. See võimaldab meil välisvõrgust lin2 vahendusel Greenyga suhelda.

1. Loome lin2 ühenduse ja suuname kohalikud pordid Greeny HTTP ja SSH pihta  
```ssh TLU-kasutaja@lin2.tlu.ee -L 2222:greeny.cs.tlu.ee:22 -L 8888:greeny.cs.tlu.ee:80```

Localhost pordid 2222 ja 8888 on nüüd suunatud.  

NB! See aken peab jääma avatuks senikaua kuni tahad Greeny ühendust kasutada!

## 3) Greeny kasutamine üle tunneli
Kõik pöördumised teeme edaspidi locahost või 127.0.0.1 poole

#### SSH
Terminalis
```ssh -p 2222 greeny-kasutaja@localhost```

#### Veeb

Ava brauseris
```http://localhost:8888/~greeny-kasutaja```

#### Failisüsteemi mountimine üle SSH
Failide üle SCP/SFTP edasi-tagasi saatmise asemel on võimalik oma kodukataloog Greenys otse enda masinasse ühendada.

1. Tekita kohalik mount point  
```mkdir -p  ~/Documents/TLU/Greeny```
2. Loo SSHFS ühendus  
```sshfs -p 2222 kasutaja@localhost:/home/greeny-kasutaja ~/Documents/TLU/Greeny```

Nüüd saad Finderist kaustade ja failidega tegeleda nagu need oleks su oma arvutis.

3. Lahtiühendamiseks  
```umount ~/Documents/TLU/Greeny```

# About

Murede ja soovituste korral saad minuga ühendust võtta Informaatika 2018 Slackis - ifikas18.slack.com

Viktor Lillemäe
