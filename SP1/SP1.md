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

* 3.7- Creem nou target

ACTIVITAT 4: Creació d'un Target Propi i Execució d'un Servei Personalitzat com a Root

**Objectiu:** Configurar i generar un nou `target` que forçarà al sistema a executar-lo per defecte. Seguidament crearem un servei (.service) amb instruccions específiques que es carregaran abans de veure l'inici gràfic. L'objectiu consisteix a demostrar que podem injectar petites peces de codi o scripts i garantir que aquestes actuaran completament amb tot l'abast i plens permisos d'Administrador.

**(Nota general: Realitza aquests passos en una instància de terminal des de l'usuari virtual sota identificació Root, emprant `sudo su` com ja posseeixes).**

**PAS 1: Crear l'script de demostració controlat**

El primer de tot és generar el fragment de programa (l'arxiu tipus script de Bash) que el servei s'ocuparà per defecte d'encendre una vegada sigui programat.

Executa `nano /usr/local/bin/activitat4.sh` al teu intèrpret, i emplena'l amb l'estructura de codi en Bash:
```bash
#!/bin/bash
echo "** INFILTRACIÓ COMPLETA ** Target custom accionat correctament per l'activitat." >> /var/log/activitat4_secret.log
echo "S'han explotat accions mitjançant l'usuari: $(whoami) en data: $(date)" >> /var/log/activitat4_secret.log
```
_Nota: El sistema `whoami` ens reportarà de forma indubtable l'autoria del procediment automàtic._

A continuació el guardem (`CTRL+O`) i certifiquem immediatament que disposarà de capacitats obligatòries d'executar-se dins el cervell intern del sistema operatiu modificant un paràmetre base:
`chmod +x /usr/local/bin/activitat4.sh`

> **(#Captura de: el llistat `ls -la /usr/local/bin/activitat4.sh` on demostri amb colors clars o la referència "*x*" que el fitxer ja disposa del lliurament dels permisos operatius d'execució necessaris).**


**PAS 2: Crear l'estructura del Target Principal**

En endavant forjarem el nostre entorn! El denominarem amb els teus atributs: canvia "NomAlumne" segons desitgis adreçar l'exercici.
Crearem el target original introduint el visor d'establiment en `nano /etc/systemd/system/NomAlumne.target` (recorda intercanviar sempre "NomAlumne" per la resta de guies). 

Posem codi necessari afirmant-li que requereix el motor multi-usuari bàsic del Linux (multi-user.target), ja que si no es quedaria literalment corrupte a l'arrancada.
```ini
[Unit]
Description=Target Propi exclusiu
Requires=multi-user.target
After=multi-user.target
AllowIsolate=yes
```
> **(#Captura de: Tot el contingut intern visible mentre l'arxiu .target editat roman obert sencer a l'editor temporal i les referències estan fixades en verd).**


**PAS 3: Vincular-ho generant un .service silenciós**

Aquest procés és clau per aconseguir injectar la crida a l'script directament només quan ens trobem a dins d'aquest subgrup `NomAlumne.target`.
Acoblament amb editor un fitxer nou a `nano /etc/systemd/system/activitat4.service`

La construcció general s’establirà com la d'un protocol típic, per instruccions simples i concretament definint la crida directa. **Atenció: Recorda fixar al `WantedBy` el nom definitiu o alias exacte del target col·locat al Pas 2 anterior**

```ini
[Unit]
Description=Servei d'encesa d'escript ocult en el nou custom target
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/activitat4.sh
User=root

[Install]
WantedBy=NomAlumne.target
```
> **(#Captura de: Demostració total on surt l'editor assenyalant i revelant el recurs "activitat4.service" sencer a la pestanya on s'il·lustra fixat correctament `WantedBy=NomAlumne.target`).**


**PAS 4: Habilitar demanant al sistema una reconstrucció**

Informarem als entorns del teu SO modern basat en Systemd d'un nou esquema, seguidament d'imposar el `.service` per al pròxim reinici actiu i dictant un tancament global canviant el destí estàndard del PC cap a aquest foradat en lloc de la unitat d'Escriptori!

Acciona de seqüència aquestes 3 línies:
- 1. Recarregar arbredes del sistema operatiu complet: `systemctl daemon-reload`
- 2. Assentiment automàtic al programari per mantenir arrencant definitivament per persistència general: `systemctl enable activitat4.service`
- 3. Alteració d'allà on arrenca permanentment la ruta de càrrega: `systemctl set-default NomAlumne.target`

> **(#Captura de: Únicament la terminal general assenyalant en successió immediata els outputs directes que t'alliberi especialment quan programis el `set-default` creant els enllaços tous (Symlinks) que confirmen el canvi de direcció).**


**PAS 5: Comprovació irrefutable d'èxit de l'explotació del buit al target**

Només cal reinicialitzar virtualment la màquina sencera simulant una pràctica tancada per defecte, l'acció es comprovarà després manualment sense veure interrupcions en l'engegat.
`reboot`! (o derivat clàssic `init 6`)

Quan ens torni la capacitat del tauler principal en una pestanya de terminal i tornem a un estat normal al fons virtual visual... comprovarem manualment que algú ocult, fora de qualsevol finestra ja ha treballat silenciosament escrivint on li vam demanar amb l'exhibició `cat`:
Visualitza el fitxer on havia de llançar la comprovació l'anterior script!

Llença pas a pas la comanda: `cat /var/log/activitat4_secret.log`

Al costat podrem extreure que on se'ns manifesta "l'usuari:" es referenciarà "root" perquè, amb qualsevol servei incrustat i no modificat expressament, qui el carrega al Systemd amb condició en alt per obligació predeterminada sense interactuar des d'un humà, posseeix accés i permisos irrestrictes de base `root` sobre l'arrel de control general.

> **(#Captura de: Línia d'èxit obtinguda després d'executar "cat ..." on s'evidenciarà perfectament "usuari: root" indicant-ho fora del quadre normal generat de control i la data autogenerada).**

* 3.8- Creem nou servei

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
