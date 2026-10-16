# NF1.UD3 — Usuaris i grups

**Mòdul:** Sistemes Operatius en Xarxa
**Activitat:** Gestió d'usuaris i grups a Ubuntu Server
**Empresa:** TechData S.L.

## Índex

1. [Creació de la jerarquia de grups](#tasca-1-creació-de-la-jerarquia-de-grups)
2. [Gestió dels perfils d'usuari](#tasca-2-gestió-dels-perfils-dusuari)
3. [Creació i configuració d'usuaris](#tasca-3-creació-i-configuració-dusuaris)
4. [Inspecció del sistema i fitxers de configuració](#tasca-4-inspecció-del-sistema-i-fitxers-de-configuració)
5. [Manteniment, bloqueig i eliminació](#tasca-5-manteniment-bloqueig-i-eliminació)
6. [Personalització de l'entorn d'usuari](#tasca-6-personalització-de-lentorn-dusuari)
7. [Conclusions](#conclusions)

---

## Tasca 1. Creació de la jerarquia de grups

### Objectiu

Crear els tres grups que necessita l'empresa per organitzar el personal: `devs` per al departament de desenvolupament, `sysadmin` per al de sistemes i `auditor` per als auditors externs.

### Comandes utilitzades

```bash
sudo groupadd devs
sudo groupadd sysadmin
sudo groupadd auditor
tail -n 5 /etc/group
```


### Explicació

La comanda `groupadd` crea un grup nou al sistema i hi afegeix una línia al fitxer `/etc/group`. Si no li indiquem cap GID amb l'opció `-g`, el sistema assigna automàticament el primer identificador lliure a partir de 1000, que és el rang reservat per als grups d'usuaris normals (per sota de 1000 hi ha els grups del sistema).

Cada línia de `/etc/group` té quatre camps separats per dos punts:

```
nom_del_grup : contrasenya : GID : llista_de_membres_secundaris
```

La `x` del segon camp indica que la contrasenya del grup, si n'hi hagués, estaria a `/etc/gshadow`. L'últim camp apareix buit perquè encara no hem creat cap usuari.

Fem servir `tail -n 5` per veure només les últimes cinc línies del fitxer, que són les més recents i per tant on apareixen els grups que acabem de crear.

![Creació dels grups i comprovació a /etc/group](/img/1.png)

### Comprovació

Els tres grups s'han creat correctament amb els GID següents:

| Grup | GID |
|------|-----|
| `devs` | 1001 |
| `sysadmin` | 1002 |
| `auditor` | 1003 |

Són consecutius perquè el sistema els ha anat assignant en l'ordre en què hem executat les comandes, partint del 1001 (el 1000 ja l'ocupava el grup de l'usuari administrador).

---

## Tasca 2. Gestió dels perfils d'usuari


### Objectiu

Configurar el directori plantilla `/etc/skel` perquè tots els usuaris que es creïn a partir d'ara rebin automàticament un fitxer de benvinguda amb les normes d'ús, una carpeta `Documents` i un àlies `ll` al seu terminal. També cal assegurar que la shell per defecte sigui `/bin/bash`.

### 2.1 Fitxer de benvinguda

```bash
sudo nano /etc/skel/BENVINGUDA.txt
```

Dins de l'editor hi escrivim les normes d'ús del servidor i guardem amb `Ctrl+O`, `Enter` i sortim amb `Ctrl+X`.

![Edició del fitxer de benvinguda amb nano](/img/2.png)

L'etiqueta `[ New File ]` que apareix a la part inferior indica que el fitxer no existia i que nano l'està creant de zero.

Comprovem que s'ha guardat correctament:

```bash
cat /etc/skel/BENVINGUDA.txt
```

![Comprovació del contingut del fitxer de benvinguda](/img/3.png)

El fitxer conté les quatre normes d'ús del servidor que rebrà tot usuari nou al seu directori personal.

### 2.2 Carpeta Documents i àlies

```bash
sudo mkdir /etc/skel/Documents
sudo nano /etc/skel/.bashrc

```

Al fitxer `.bashrc` hi afegim la línia:

```bash
alias ll='ls -la --color=auto'
```

![Creació de la carpeta Documents i àlies al .bashrc](/img/4.png)

**Explicació:** un àlies és un nom curt que substitueix una comanda més llarga. En aquest cas, quan l'usuari escrigui `ll`, bash executarà `ls -la --color=auto`, que mostra tots els fitxers (inclosos els ocults) en format llarg i amb colors.

Cal tenir en compte que el `.bashrc` per defecte d'Ubuntu ja porta un àlies `ll` definit com `ls -alF`. Com que bash llegeix el fitxer de dalt a baix, la nostra definició, col·locada més avall, sobreescriu l'anterior i és la que acaba tenint efecte.

### 2.3 Shell per defecte

```bash
grep DSHELL /etc/adduser.conf
```

![Comprovació de la variable DSHELL](/img/5.png)

**Explicació:** el fitxer `/etc/adduser.conf` conté la configuració per defecte de la comanda `adduser`. La variable `DSHELL` defineix quina shell s'assignarà als usuaris nous. A la captura es veu que el valor és `/bin/bash`, que és exactament el que demana l'enunciat, i que la línia apareix comentada amb `#` perquè ja és el valor predeterminat del programa: les línies comentades serveixen de documentació i només cal descomentar-les si volem canviar el valor.

### 2.4 Comprovació amb un usuari de prova

Creem un usuari `prova10` i iniciem sessió per verificar que hereta les personalitzacions:

```bash
sudo adduser prova10
su - prova10
ls -la

ll
```

![Primer intent amb l'usuari prova10](/img/6.png)

**Observació:** en aquest primer intent el fitxer `BENVINGUDA.txt` no apareix al directori personal. El motiu és que s'havia sortit de nano sense desar cap contingut, de manera que el fitxer no existia a `/etc/skel` quan es va crear l'usuari. Un cop corregit, repetim la prova amb un usuari nou:

```bash
sudo adduser prova10b
su - prova10b
ls -la
ll
```

![Comprovació correcta amb l'usuari prova10b](/img/7.png)

### Comprovació

Ara el directori personal de `prova10b` conté les tres personalitzacions:

- El fitxer **`BENVINGUDA.txt`**, copiat automàticament des de `/etc/skel`.
- La carpeta **`Documents`**.
- L'àlies **`ll`**, que funciona i mostra el llistat en format llarg.

Això confirma que `/etc/skel` actua com a plantilla: tot el seu contingut es copia al `/home` de cada usuari nou en el moment de crear-lo.

### 2.5 Eliminació de l'usuari de prova

```bash
sudo deluser -r prova10
```


![Error en utilitzar l'opció abreujada -r](/img/121212.png)

**Incidència trobada:** la comanda retorna l'error `Option r is ambiguous (remove-all-files, remove-home)`. En aquesta versió d'Ubuntu, `deluser` té dues opcions que comencen per `r` —`--remove-home` i `--remove-all-files`— i per això l'abreviatura `-r` resulta ambigua i el programa no pot decidir quina volem. La solució és escriure l'opció completa:

```bash
sudo deluser --remove-home prova10
sudo deluser --remove-home prova10b
ls /home
```

![Eliminació correcta dels usuaris de prova](/img/9.png)

### Comprovació

A `/home` només hi queden els directoris dels usuaris reals (`laura_dev`, `marc_sys`, `pau_dev` i `usuari`), de manera que els usuaris de prova i els seus directoris personals s'han eliminat completament.

---

## Tasca 3. Creació i configuració d'usuaris

### Objectiu

Donar d'alta els quatre usuaris que demana l'empresa, assignant-los els grups principals i secundaris corresponents, i aplicar-hi la política de contrasenyes inicial.

| Usuari | Nom real | Grup principal | Grups secundaris | Shell |
|--------|----------|----------------|------------------|-------|
| `pau_dev` | Pau Garcia | devs | — | /bin/bash |
| `laura_dev` | Laura Martí | devs | — | /bin/bash |
| `marc_sys` | Marc Soler | sysadmin | devs | /bin/bash |
| `auditor` | Usuari Auditor | auditor | — | /bin/bash |

### 3.1 Creació amb `adduser`


```bash
sudo adduser --ingroup devs pau_dev
sudo adduser --ingroup devs laura_dev
```

![Creació de pau_dev i laura_dev amb adduser](/img/44.png)

**Explicació:** `adduser` és un script interactiu d'alt nivell que crea l'usuari, el seu directori personal, hi copia el contingut de `/etc/skel`, demana la contrasenya i la informació GECOS (nom real, telèfons, etc.).

L'opció `--ingroup devs` és imprescindible: sense ella, `adduser` crearia un grup propi amb el mateix nom de l'usuari i l'hi assignaria com a grup principal, que no és el que demana l'enunciat.

### 3.2 Creació amb `useradd`

```bash
sudo useradd -m -d /home/marc_sys -s /bin/bash -g sysadmin -G devs -c "Marc Soler" marc_sys
sudo useradd -m -d /home/auditor -s /bin/bash -g auditor -c "Usuari Auditor" auditor
```

![Creació de marc_sys i auditor amb useradd](/img/55.png)

**Explicació:** a diferència d'`adduser`, la comanda `useradd` és de baix nivell i no és interactiva: no demana res i només fa exactament allò que li indiquem amb les opcions. Per això cal especificar-ho tot:

| Opció | Funció |
|-------|--------|
| `-m` | Crea el directori personal i hi copia `/etc/skel`. Sense aquesta opció l'usuari existeix però no té `/home`. |
| `-d` | Indica la ruta del directori personal. |
| `-s` | Defineix la shell de l'usuari. |
| `-g` | Assigna el grup **principal**. |
| `-G` | Assigna els grups **secundaris** (separats per comes si n'hi ha més d'un). |
| `-c` | Camp GECOS, on hi posem el nom real. |


### 3.3 Assignació de contrasenyes

```bash
sudo passwd marc_sys
sudo passwd auditor
```

![Assignació de contrasenyes](/img/777.png)

Els usuaris creats amb `adduser` ja tenien contrasenya perquè l'script la demana durant el procés. Als creats amb `useradd` cal assignar-la manualment, ja que sense contrasenya el compte queda bloquejat i no s'hi pot iniciar sessió.

### 3.4 Caducitat forçada per a `pau_dev`

```bash
sudo chage -d 0 pau_dev
sudo chage -l pau_dev
su - pau_dev
```

![Forçar el canvi de contrasenya amb chage](/img/222.png)

**Explicació:** `chage` (*change age*) gestiona la caducitat de les contrasenyes. L'opció `-d 0` posa la data de l'últim canvi a 0, és a dir, l'1 de gener de 1970. Com que aquesta data és anterior a qualsevol període de validesa, el sistema interpreta que la contrasenya ha caducat i obliga l'usuari a canviar-la en el primer inici de sessió. L'opció `-l` mostra tota la informació de caducitat del compte.

### Comprovació

En consultar `chage -l pau_dev` els camps de caducitat indiquen **«se debe cambiar la contraseña»**, i en iniciar sessió amb `su - pau_dev` apareix el missatge `You are required to change your password immediately (administrator enforced)`, de manera que el sistema no deixa continuar fins que no se'n defineix una de nova.

---

## Tasca 4. Inspecció del sistema i fitxers de configuració

### Objectiu


Analitzar la informació que guarda el sistema sobre els usuaris i entendre el paper de cada fitxer de configuració.

### Comandes utilitzades

```bash
id marc_sys
grep pau_dev /etc/passwd
sudo grep pau_dev /etc/shadow
ls -l /etc/shadow
grep SKEL /etc/adduser.conf
grep SKEL /etc/default/useradd
```

![Inspecció dels fitxers del sistema](/img/1000.png)

### Pregunta 1: Quins UID, GID i grups té assignats `marc_sys`?

La sortida de `id marc_sys` és:

```
uid=1005(marc_sys) gid=1002(sysadmin) groups=1002(sysadmin),1001(devs)
```

| Camp | Valor |
|------|-------|
| **UID** | 1005 |
| **GID** (grup principal) | 1002, corresponent a `sysadmin` |
| **Grups** | `sysadmin` (1002) com a principal i `devs` (1001) com a secundari |

Això confirma que les opcions `-g sysadmin` i `-G devs` del `useradd` s'han aplicat correctament. La pertinença a dos grups permet que `marc_sys` accedeixi tant als recursos del departament de sistemes com als de desenvolupament.


### Pregunta 2: Quina diferència hi ha entre `/etc/passwd` i `/etc/shadow`? Quins permisos té `/etc/shadow` i per què?

A **`/etc/passwd`** hi trobem:

```
pau_dev:x:1001:1001:pau,1,1,1,1:/home/pau_dev:/bin/bash
```

Els set camps són: nom d'usuari, indicador de contrasenya, UID, GID, informació GECOS, directori personal i shell. És informació **pública i no sensible**, i per això el fitxer és llegible per tots els usuaris: molts programes necessiten consultar-lo per traduir un UID a un nom d'usuari.

La **`x`** del segon camp és la clau: indica que la contrasenya no es guarda aquí, sinó a `/etc/shadow`.

A **`/etc/shadow`** hi trobem:

```
pau_dev:$y$j9T$v8t6.bmlS/...:20740:0:99999:7:::
```

Aquí hi ha el **hash** de la contrasenya i tota la informació de caducitat. El prefix `$y$` identifica l'algoritme **yescrypt**, el que utilitza Ubuntu actualment. Els números següents indiquen la data de l'últim canvi (20740 dies des de l'1/1/1970), el mínim de dies entre canvis (0), el màxim (99999) i els dies d'avís abans de caducar (7).

Els permisos són:

```
-rw-r----- 1 root shadow 1452 /etc/shadow
```

És a dir, **640**, amb propietari `root` i grup `shadow`. Només root hi pot escriure, i només root i els membres del grup `shadow` el poden llegir. La resta d'usuaris no hi tenen cap accés.

**Per què aquesta protecció?** Tot i que les contrasenyes estan xifrades i no es poden desxifrar directament, un atacant que obtingués els hash podria intentar trencar-los *offline* mitjançant força bruta o atacs de diccionari, provant milions de combinacions sense deixar rastre al sistema. Històricament els hash es guardaven al mateix `/etc/passwd`, que és llegible per tothom, i precisament per aquest risc es van separar en un fitxer amb permisos restrictius.

### Pregunta 3: Quin és el directori plantilla i on es configura?


El directori plantilla és **`/etc/skel`** (abreviatura de *skeleton*). Tot el seu contingut es copia automàticament al directori personal de cada usuari nou en el moment de crear-lo.

La configuració es defineix en dos fitxers diferents segons l'eina que fem servir:

| Eina | Fitxer de configuració | Directiva |
|------|------------------------|-----------|
| `adduser` | `/etc/adduser.conf` | `SKEL=/etc/skel` |
| `useradd -m` | `/etc/default/useradd` | `SKEL=/etc/skel` |

A la captura es veu que en tots dos casos les línies apareixen comentades amb `#`. Això és normal: són els valors predeterminats que el programa aplica igualment, i les línies només serveixen de documentació. Si volguéssim fer servir una altra ruta com a plantilla, hauríem de descomentar la línia i canviar-ne el valor.

---

## Tasca 5. Manteniment, bloqueig i eliminació

### Objectiu

Gestionar el cicle de vida dels comptes: bloquejar temporalment un usuari que marxa de vacances, desbloquejar-lo quan torna i eliminar definitivament el compte d'un treballador que deixa l'empresa.

### Comandes utilitzades

```bash
sudo usermod -L pau_dev
sudo grep pau_dev /etc/shadow

sudo usermod -U pau_dev
sudo grep pau_dev /etc/shadow

sudo userdel -r auditor
ls /home
grep auditor /etc/passwd
```


![Bloqueig, desbloqueig i eliminació d'usuaris](/img/30.png)

### Explicació

**Bloqueig (`usermod -L`).** Després d'executar la comanda, la línia de `/etc/shadow` passa de `pau_dev:$y$j9T$...` a `pau_dev:!$y$j9T$...`. El sistema no esborra el hash, sinó que hi afegeix un signe **`!`** al davant. Aquest caràcter fa que cap contrasenya introduïda pugui generar mai aquesta cadena, de manera que l'autenticació sempre falla. Com que el hash original es conserva intacte, el bloqueig és completament reversible i l'usuari recuperarà la seva contrasenya d'abans.

**Desbloqueig (`usermod -U`).** Elimina el `!` i la línia torna exactament al seu estat original. Comparant les dues captures es veu que són idèntiques excepte per aquest caràcter.

**Eliminació (`userdel -r`).** L'opció `-r` (*remove*) elimina el compte juntament amb el seu directori personal i la seva bústia de correu de `/var/mail`. El missatge `auditor mail spool (/var/mail/auditor) not found` **no és un error**: simplement informa que aquest usuari no tenia cap bústia creada, cosa habitual en un servidor sense servei de correu configurat.

### Comprovació

- Amb el compte bloquejat, el `!` apareix clarament a `/etc/shadow`.
- Després del desbloqueig, el `!` desapareix i la línia recupera el format original.
- Després de l'eliminació, `ls /home` ja no mostra el directori `auditor` i `grep auditor /etc/passwd` no retorna cap resultat, cosa que confirma que el compte ha desaparegut del sistema.

---

## Tasca 6. Personalització de l'entorn d'usuari

### Objectiu

Instal·lar la shell `fish` i assignar-la com a shell per defecte de l'usuària `laura_dev`, comprovant que el canvi es reflecteix a `/etc/passwd` i que funciona en iniciar sessió.

### 6.1 Instal·lació del paquet

```bash
sudo apt update
sudo apt install fish -y
```


![Actualització de la llista de paquets](/img/25.png)

![Instal·lació de fish](/img/26.png)

**Explicació:** `apt update` descarrega la llista actualitzada de paquets disponibles des dels repositoris configurats a `/etc/apt/sources.list`. És imprescindible executar-lo abans d'instal·lar res, perquè sense una llista actualitzada APT podria intentar descarregar versions que ja no existeixen als servidors.

`apt install fish -y` instal·la el paquet i les seves dependències (`fish-common`, `bzip2`, `libpcre2-32-0`, etc.). L'opció `-y` respon automàticament que sí a la confirmació.

### 6.2 Canvi de shell

```bash
which fish
sudo usermod -s /usr/bin/fish laura_dev
grep laura_dev /etc/passwd
su - laura_dev
echo $SHELL
status
```

![Canvi de shell i sessió amb fish](/img/100.png)

![Canvi de shell i sessió amb fish](/img/101.png)

**Explicació:** primer executem `which fish` per saber la ruta exacta on s'ha instal·lat el binari (`/usr/bin/fish`). Aquest pas és important: si a `usermod -s` hi indiquéssim una ruta inexistent, l'usuari no podria iniciar sessió.

L'opció `-s` de `usermod` modifica el setè camp de la línia de l'usuari a `/etc/passwd`, que és precisament el que defineix quina shell s'executa en iniciar sessió. Com a alternativa es podria fer servir la comanda `chsh`, que fa el mateix però pensada perquè l'usuari pugui canviar-se la seva pròpia shell.

### Comprovació

- A `/etc/passwd`, la línia de `laura_dev` acaba ara amb `/usr/bin/fish` en lloc de `/bin/bash`.
- En iniciar sessió amb `su - laura_dev` apareix el missatge de benvinguda de fish i el *prompt* canvia a `laura_dev@sox-abc ~>`, amb colors i format propis.
- `echo $SHELL` retorna `/usr/bin/fish` i `status` confirma `This is a login shell`.

**Nota sobre fish:** és una shell moderna amb autocompletat intel·ligent basat en l'historial i comprovació de sintaxi en temps real, cosa que l'ha fet popular entre desenvolupadors. Cal tenir present, però, que **no és compatible amb l'estàndard POSIX**, de manera que els scripts escrits per a bash o sh no s'hi executen sense adaptar-los. Per això no es recomana com a shell per a scripts del sistema, encara que sigui còmoda per a ús interactiu.


---

## Conclusions

En aquesta activitat s'ha configurat des de zero l'estructura d'usuaris i grups del servidor de TechData S.L., treballant amb les dues famílies d'eines disponibles a Linux:

- **Eines d'alt nivell** (`adduser`, `deluser`): interactives i còmodes, automatitzen la creació del directori personal, la còpia de `/etc/skel` i l'assignació de contrasenya. Són les recomanades per a l'ús manual del dia a dia.
- **Eines de baix nivell** (`useradd`, `userdel`, `usermod`): no interactives i totalment controlables mitjançant paràmetres, cosa que les fa ideals per a scripts i automatització, però obliguen a especificar explícitament cada detall.

També s'ha comprovat la importància del directori plantilla `/etc/skel`, que permet estandarditzar l'entorn de tots els usuaris nous des d'un únic punt, i la separació entre `/etc/passwd` i `/etc/shadow` com a mesura de seguretat fonamental per protegir els hash de les contrasenyes.

Durant el procés s'han resolt dues incidències: l'ambigüitat de l'opció `-r` a la comanda `deluser`, que cal escriure com a `--remove-home`, i la necessitat de desar efectivament el contingut del fitxer de benvinguda perquè `/etc/skel` el propagui als usuaris nous.