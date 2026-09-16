---
layout: default
title: "SISTEMES D'INICI"
---
# SISTEMES D'INICI

## Índex

**1- SystemV vs Upstart vs Systemd**
* 1.1- Runlevels o Targets?
* 1.2- Quin el nostre SO?
  

**2- SystemV**
* 2.1- Directoris
* 2.2- Procés arrencada

**3- Systemd**
* 3.1- Directoris
* 3.2- systemctl
* 3.3- dependències
* 3.4- Modificar target provisional
* 3.5- Modificar target definitiu
* 3.6- Afegir/treure serveis target
* 3.7- Creem nou target
* 3.8- Creem nou servei
---


<img width="1024" height="572" alt="image" src="https://github.com/user-attachments/assets/d6d2a5fa-8d2d-4d93-aa4a-6054b6f91b4d" />

---

## Conceptes

* **Kernel** -> gestiona processos
* **Aplicació** -> programa interactua usuari i executa 1r pla
* **Servei** -> programa associat SO i 2n pla
* **Procés** -> f(x) intern del SO
  * *Nota:* Aplicacions i serveis -> generen processos (sincronitzar i planificar)

---

## Nivells d'execució

* **0** - power off
* **1** - rescue -> 1 usuari (dimonis mínims)
* **2-5** -> multiusuari, xarxa, sense...
* **6** -> reboot

---

## Comandes d'aturada

* `/etc/init.d/cron stop`
* `service cron stop`
* `systemctl stop cron`


---


**1- SystemV vs Upstart vs Systemd**
* 1.1- Runlevels o Targets?

ACTIVITAT 1
Com sabem a quin nivell d'execució estem?
runlevel (Ubuntu 24.04)
<img width="304" height="54" alt="image" src="https://github.com/user-attachments/assets/a2c038e5-88ab-4345-a83a-43ba62589b87" />


init.d
trobem tots els... com ara cron
tot el que va per systemv esta a init.d 

<img width="840" height="167" alt="image" src="https://github.com/user-attachments/assets/c18e78eb-5baf-4d0e-a5f6-dff36100561a" />




dintre de etc a les carpetes de rc trobem :
<img width="413" height="291" alt="image" src="https://github.com/user-attachments/assets/9485aa78-e523-4106-b38a-acbf6f8ab45f" />

<img width="825" height="127" alt="image" src="https://github.com/user-attachments/assets/4514cd8c-89c7-4a11-9958-03116e35c6f7" />

Comanda "init"
init + (nivell d'execucio) , per ex: init 6 fa un reboot





directori per defecte on s'instala el systemd

/lib/systemd/system
<img width="770" height="446" alt="image" src="https://github.com/user-attachments/assets/b491b996-7fc5-4ea8-b89b-0d73cea8f23e" />


/etc/systemd
quan volem modificar alguna configuracio
en cas de duplicitat sempre preval sobre el 

Per filtrar per tipus:
Per exemple filtrem per target
<img width="715" height="682" alt="image" src="https://github.com/user-attachments/assets/137b0395-2386-44d8-a95c-7cecff975b01" />
per veure el ... perdefecte systemctl get-default
<img width="555" height="42" alt="image" src="https://github.com/user-attachments/assets/9c5c69f9-f22b-42d2-87b8-b713841e9c68" />

isolate
<img width="649" height="26" alt="image" src="https://github.com/user-attachments/assets/1478d00b-6ef0-48dd-92f7-3f8a86645b26" />

per canviar temporalment
systemctl isolate rescue.target
<img width="642" height="20" alt="image" src="https://github.com/user-attachments/assets/d7c86410-12e9-4e3a-a3e0-13224246bfb5" />


filtrem per target i veiem que default.target esta -> graphical.target
<img width="838" height="288" alt="image" src="https://github.com/user-attachments/assets/a5780c8a-d073-44cd-b33d-91cc1f03f456" />

borrem l'enllaç, el tornem a crear al target 
<img width="688" height="37" alt="image" src="https://github.com/user-attachments/assets/12ad743a-cc84-4cf3-a49f-02554b309248" />

ara per defecte no sera graphical sera target.rescue

<img width="684" height="248" alt="image" src="https://github.com/user-attachments/assets/e1936118-af48-4e38-bb6b-defb192c4337" />

Comprovem que en fer el reboot estem en rescue mode. revertim els canvis amb:
<img width="733" height="162" alt="image" src="https://github.com/user-attachments/assets/5c437560-7682-4232-82a6-ee3565c2803e" />


ACTIVITAT 2 SSH

Instalem ssh
<img width="857" height="911" alt="image" src="https://github.com/user-attachments/assets/fe2e9e62-6cb0-4220-908d-8e8b92148a8a" />

L'habilitem
<img width="859" height="359" alt="image" src="https://github.com/user-attachments/assets/e32bc40d-2c8e-4633-bd1d-d4e9c46aded3" />

aquesta comanda ens diu el target graphical quines dependencies te:
<img width="897" height="190" alt="image" src="https://github.com/user-attachments/assets/0ba425c5-644a-43ee-b89b-3fa2761eecbf" />

<img width="870" height="860" alt="image" src="https://github.com/user-attachments/assets/67db002f-93df-4eb7-98b5-80c1492d80f7" />

COMANDA systemd-analyze
<img width="762" height="61" alt="image" src="https://github.com/user-attachments/assets/bd12ead6-4174-4b2b-b719-647e93d06dfd" />

ACTIVITAT 3:
Crear un .service

El ficarem a un target
Per a que quan s'engaguee el so. alli tenim permisos de root aixi que es podra explotar la vulnerabilitat.

Demostració:

Creem l'script:
<img width="609" height="185" alt="image" src="https://github.com/user-attachments/assets/6d3738c5-98a4-4176-9335-ff9998e8507f" />

I li donem permisos d'execució:
<img width="411" height="45" alt="image" src="https://github.com/user-attachments/assets/4ac5f6b7-2e7d-4d9e-9637-38f765fc04ac" />

Ara creem el .service
<img width="646" height="329" alt="image" src="https://github.com/user-attachments/assets/5b317894-32c0-49f7-93e5-ad929c4d958d" />
L'habilitem:

<img width="897" height="636" alt="image" src="https://github.com/user-attachments/assets/85ad35d5-1b23-46d6-9d10-cb2ce080a148" />

Fem un reboot i comprovem que hi hagi una "a" a /etc/pwd

<img width="735" height="602" alt="image" src="https://github.com/user-attachments/assets/4b9793a5-72fb-4c20-96b8-dfc0b3a644c7" />


ACTIVITAT 4:

Fer un target amb el meu nom. Que depengui d'algu.

el getdefault el canviarem al nostre . dintre tindra el .service que cridara un script amb permisos de root abans que s'inicie el SO

Que fara l'script amb permisos de root? El que volguessem (keyloggers, connexions ssh, captures de pantalla del que esta veien l'usuari, etc. el que sigue )



* 1.2- Quin el nostre SO?

  man init

  <img width="894" height="761" alt="image" src="https://github.com/user-attachments/assets/0b0e154c-0560-45de-a417-c0688c578443" />
readlink -v /sbin/init
<img width="500" height="39" alt="image" src="https://github.com/user-attachments/assets/59574d16-7e13-4c72-bc25-96d6ad7b7ff3" />




