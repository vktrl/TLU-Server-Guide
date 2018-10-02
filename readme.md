# Mugavam ligipääs Greenyle

Oma arvutis Greeny failidega tegelemine on natuke tüütu. Üks variant on kasutada SSH-d ja vim + tmux failidega töötamiseks ja SCP klieni failide üles/alla laadimiseks. Võid kasutada mõne tekstiredaktori SFTP pluginat (mul tekkisid synciga probleemid). Kolmas variant on mountida Greeny failid oma kohalikku failisüsteemi (minu meelest kõige mugavam).

## Laadi alla ja installi FUSE ja SSHFS

https://osxfuse.github.io

SSHFS 2.5.0 on vana, aga töötab. Uuema versiooni (pead ise kompileerima) saad siit: https://github.com/libfuse/sshfs

## Tekita kohalik mount point

`mkdir -p  ~/Documents/TLU/Greeny`

## Loo SSH tunnel(id) lin2.tlu.ee-ga

`ssh kasutaja@lin2.tlu.ee -L 2222:greeny.cs.tlu.ee:22 -L 8888:greeny:cs:tlu.ee:80`

Nüüd saad välisest võrgust Greeny veebiserverile ligi aadressil http://localhost:8888 ja üle SSH $ssh -p 2222 kasutaja@localhost

## Loo SSHFS ühendus

`sshfs -p 2222 kasutaja@localhost:/home/kasutaja ~/Documents/TLU/Greeny`

Nüüd saad Finderist kaustade ja failidega tegeleda nagu need oleks su oma arvutis.

## Lahtiühendamiseks

`umount ~/Documents/TLU/Greeny`
