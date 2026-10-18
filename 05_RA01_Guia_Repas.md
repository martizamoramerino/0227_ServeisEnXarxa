![Flux Pràctica](./source/05_RA01_Guia_Repas/00-portada-guia-repas.png)

# RA01 · Guia de repàs per a l'examen

Guia de síntesi per estudiar els conceptes principals treballats fins ara i repassar les comprovacions i ordres utilitzades a les pràctiques. La base teòrica són les lliçons [01 · Repàs de xarxes TCP/IP](01_Repas_TCP_IP_Guide_Activity.md) i [02 · Introducció al servei DHCP](02_Teoria_DHCP_Guide.md). La part aplicada recull procediments de les guies de [configuració de Kea](03_A_DHCP_Kea_Guia.md) i [l'activitat pràctica amb Kea i Zorin](03_B_DHCP_Kea_Activitat.md).

> [!IMPORTANT]
> Les adreces IP dels exemples són didàctiques. A la pràctica, identifica les interfícies reals i substitueix els valors per la xarxa que t'hagi assignat el docent. No barregis la IP de la interfície NAT amb la de la xarxa interna.

## Com aprofitar aquesta guia

1. Estudia primer els conceptes de xarxes i adreçament.
2. Assegura't que pots calcular una subxarxa i justificar els seus límits.
3. Repassa DHCP i explica DORA, les concessions i el disseny del servei.
4. Utilitza l'apartat pràctic com a recordatori d'ordres: abans d'executar-ne una, explica què vols comprovar.
5. Tapa les definicions i intenta respondre les preguntes de síntesi del final.

---

## 1. Fonaments de xarxes TCP/IP

### 1.1 Capes i identificadors

TCP/IP agrupa les funcions de comunicació en capes. Cada capa resol una part diferent del trajecte:

| Capa o funció | Exemples | Pregunta que resol |
|---|---|---|
| Aplicació | HTTP(S), DNS | Quin servei o quina informació necessitem? |
| Transport | TCP, UDP | A quin procés o servei es lliuren les dades? |
| Internet | IPv4, IPv6, ICMP | A quina adreça IP s'envia el paquet i com s'encamina? |
| Accés a la xarxa | Ethernet, Wi-Fi | Com es transmet una trama dins de l'enllaç local? |

No confonguis aquests identificadors:

- **MAC:** identifica una interfície a l'enllaç local. S'utilitza en les trames Ethernet; no és l'adreça que permet encaminar paquets entre xarxes.
- **IP:** identifica l'adreçament lògic de la interfície dins d'una xarxa IP i permet encaminar paquets.
- **Port:** identifica el servei o procés de transport. Per exemple, HTTPS acostuma a utilitzar el port de destinació 443.

En un trajecte, la IP identifica el destí de xarxa, el port identifica el servei i la MAC permet lliurar la trama al següent dispositiu de l'enllaç local. Els ports van de `0` a `65535`.

<!-- IMATGE 01 — PROMPT: Crea una infografia vectorial original, horitzontal i llegible sobre l'encapsulament TCP/IP. Mostra un missatge que baixa per les capes d'aplicació, transport, Internet i accés a la xarxa a l'emissor, i puja per les capes al receptor. Etiqueta HTTP/DNS, TCP/UDP i ports, IP, Ethernet/Wi-Fi i MAC; indica que MAC és d'enllaç local i que IP s'utilitza per encaminar entre xarxes. Textos en català, sense imitar captures reals. Fitxer suggerit: source/05_RA01_Repas/01-pila-tcp-ip.png -->
![Piles TCP i IP](./source/05_RA01_Guia_Repas/01-pila-tcp-ip.png)

### 1.2 Adreçament IPv4 i IPv6

- **IPv4** té 32 bits, representats en quatre octets decimals entre 0 i 255, per exemple `192.168.10.20`.
- **IPv6** té 128 bits i s'escriu en grups hexadecimals separats per dos punts. IPv6 no utilitza broadcast.
- Una adreça IP, tota sola, **no indica la mida de la xarxa**: cal conèixer la màscara o el prefix CIDR.

Rangs privats IPv4 que cal reconèixer:

| Rang privat | Interval |
|---|---|
| `10.0.0.0/8` | `10.0.0.0` – `10.255.255.255` |
| `172.16.0.0/12` | `172.16.0.0` – `172.31.255.255` |
| `192.168.0.0/16` | `192.168.0.0` – `192.168.255.255` |

Per tant, no tot el rang `172.x.x.x` és privat. `127.0.0.1` és una adreça de bucle local (*loopback*). Les classes A, B i C són una classificació històrica: la mida real d'una xarxa actual la determina el prefix configurat, no la classe aparent de la IP.

### 1.3 Màscara, prefix i càlcul de subxarxes

La màscara separa els bits de xarxa (bits `1`) dels bits d'equip (bits `0`). El prefix indica quants bits corresponen a la xarxa:

| Prefix | Màscara | Bits d'equip | Adreces totals |
|---:|---|---:|---:|
| `/24` | `255.255.255.0` | 8 | 256 |
| `/26` | `255.255.255.192` | 6 | 64 |
| `/28` | `255.255.255.240` | 4 | 16 |

En els exercicis IPv4 convencionals:

- **Adreces totals:** `2^h`, on `h = 32 − prefix`.
- **Adreces de host habitualment assignables:** `2^h − 2` (no es compten l'adreça de xarxa ni el broadcast).
- **Adreça de xarxa:** resultat de l'operació AND entre IP i màscara; els bits d'equip són zero.
- **Broadcast:** mateixa part de xarxa i tots els bits d'equip a u.
- **Primer i últim host:** xarxa + 1 i broadcast − 1.

**Mètode dels blocs:** identifica l'octet on la màscara no és `255` ni `0`; calcula `256 − valor de la màscara` per obtenir la mida del bloc. Busca el múltiple immediatament inferior o igual a l'octet de la IP. Aquest límit inicia la xarxa; el final del bloc determina el broadcast.

**Exemple:** `10.254.96.104/22` té màscara `255.255.252.0`. Al tercer octet, la mida del bloc és `256 − 252 = 4`; el 96 inicia el bloc `96–99`. Per tant:

| Xarxa | Hosts assignables | Broadcast |
|---|---|---|
| `10.254.96.0/22` | `10.254.96.1` – `10.254.99.254` | `10.254.99.255` |

Si et donen un nombre de dispositius, compta totes les interfícies que necessiten IP i tria el nombre mínim de bits `h` que compleix `2^h − 2 ≥ dispositius`.

<!-- IMATGE 02 — PROMPT: Genera una infografia didàctica original per calcular una xarxa IPv4 /22. Mostra IP 10.254.96.104, màscara 255.255.252.0, càlcul de bloc 256 − 252 = 4 al tercer octet, xarxa 10.254.96.0 i broadcast 10.254.99.255. Marca també el rang assignable 10.254.96.1–10.254.99.254. Comprova amb cura tots els valors; utilitza una recta de blocs clara, textos en català i estil vectorial. Fitxer suggerit: source/05_RA01_Repas/02-calcul-subxarxa.png -->
![Càcul Xarxa](./source/05_RA01_Guia_Repas/02-calcul-subxarxa.png)

### 1.4 IP, màscara, porta d'enllaç i DNS

| Dada | Funció |
|---|---|
| IP | Adreça assignada a la interfície. |
| Màscara o prefix | Determina quines destinacions són a la mateixa subxarxa. |
| Porta d'enllaç (*gateway*) | Router al qual s'envia el trànsit destinat a altres xarxes. Habitualment és dins de la mateixa subxarxa que el client. |
| DNS | Servei que resol noms de domini a adreces IP; no és necessàriament el gateway. |

Per diagnosticar, comprova en ordre: **enllaç → IP i prefix → veí/gateway → rutes → connectivitat per IP → DNS → port i servei**. Una prova correcta de `ping` al gateway no demostra que Internet o DNS funcionin. Si una IP externa respon però el nom no es resol, investiga DNS.

### 1.5 Encaminament, TCP/UDP i NAT

- Si la destinació és a la mateixa subxarxa, l'equip intenta lliurar-hi directament la trama.
- Si és fora de la subxarxa, l'equip envia el paquet al gateway.
- Quan hi ha diverses rutes possibles, s'escull la coincidència de **prefix més llarg**. La ruta `0.0.0.0/0` és la ruta per defecte.
- **TCP** estableix connexió i proporciona lliurament fiable i ordenat del flux. **UDP** envia datagrames sense establir una connexió TCP ni oferir aquestes garanties per si mateix.
- NAT tradueix adreces entre xarxes; **PAT/NAPT** permet que diversos equips comparteixin una IP exterior diferenciant connexions amb ports. NAT no és sinònim de tallafoc.

---

## 2. Teoria essencial de DHCP

### 2.1 Què fa DHCP?

**DHCP** (*Dynamic Host Configuration Protocol*) permet que un client demani configuració de xarxa a un servidor. El servidor pot proporcionar IP, màscara/prefix, gateway, DNS i altres opcions.

| Modalitat | On es configura? | Idea clau |
|---|---|---|
| IP manual | A cada dispositiu client | Cal mantenir els valors individualment. |
| Assignació dinàmica | Al servidor, dins d'un pool | El client rep una adreça durant una concessió limitada. |
| Reserva DHCP | Al servidor, vinculada a la MAC o identificador de client | L'adreça prevista és estable, però el client continua utilitzant DHCP. |

Una màquina pot tenir funcions diferents en interfícies diferents: Ubuntu Server pot ser client DHCP a l'adaptador NAT i alhora servidor DHCP Kea a la xarxa interna. `dhcp4: true` a Netplan configura el comportament de **client** d'aquella interfície; no inicia un servidor DHCP.

DHCP anuncia l'adreça d'un servidor DNS o d'un gateway quan està configurat perquè ho faci, però no crea aquests serveis. Instal·lar Kea tampoc configura automàticament encaminament o NAT.

<!-- IMATGE 03 — PROMPT: Crea un diagrama de xarxa original amb un Ubuntu Server que té dues interfícies: NAT com a client DHCP i xarxa interna com a servidor Kea DHCPv4. Connecta-hi un client Zorin a la xarxa interna. Diferencia clarament les dues funcions DHCP, mostra que Kea només escolta a la interfície interna i indica que anunciar un gateway o DNS no crea aquests serveis. Etiquetes en català, sense captures de pantalla. Fitxer suggerit: source/05_RA01_Repas/03-client-i-servidor-dhcp.png -->
![Client i Servidor DHCP](./source/05_RA01_Guia_Repas/03-client-i-servidor-dhcp.png)

### 2.2 DORA, ports i broadcast

DHCPv4 utilitza **UDP**:

| Sentit | Port UDP d'origen | Port UDP de destinació |
|---|---:|---:|
| Client → servidor | 68 | 67 |
| Servidor → client | 67 | 68 |

Quan el client encara no té IP, pot iniciar la cerca des de `0.0.0.0` i enviar un broadcast local. L'ordre inicial habitual és **DORA**:

| Pas | Missatge | Funció |
|---:|---|---|
| 1 | `DHCPDISCOVER` | El client busca servidors DHCP. |
| 2 | `DHCPOFFER` | Un servidor proposa una IP i opcions. |
| 3 | `DHCPREQUEST` | El client sol·licita una de les ofertes. |
| 4 | `DHCPACK` | El servidor confirma la concessió i les opcions. |

Una oferta no és encara una concessió confirmada: cal veure el `REQUEST` i l'`ACK`. Si responen diversos servidors, el client selecciona una oferta. L'ús de broadcast o unicast depèn de l'estat i del missatge; no assumeixis que tots els `OFFER` o `ACK` han de ser broadcast.

<!-- IMATGE 04 — PROMPT: Dibuixa una seqüència DORA clara entre un client i dos servidors DHCP. Mostra Discover, dues ofertes, Request seleccionant un servidor i ACK d'aquest servidor. Afegeix UDP 68→67 per client a servidor i UDP 67→68 en el sentit contrari. Destaca que l'oferta no confirma per si sola la concessió i no classifiquis tots els missatges com a broadcast obligatori. Textos en català, diagrama original i llegible. Fitxer suggerit: source/05_RA01_Repas/04-sequencia-dora.png -->
![Seqüència DORA](./source/05_RA01_Guia_Repas/04-sequencia-dora.png)

### 2.3 Concessions, renovacions i altres missatges

Una **concessió** (*lease*) autoritza el client a utilitzar la IP durant un temps. Renovar-la no vol dir necessàriament canviar d'adreça.

- **T1 (`renew-timer`):** el client intenta renovar amb el servidor que li va concedir l'adreça.
- **T2 (`rebind-timer`):** si no ha pogut renovar, el client intenta obtenir renovació d'un servidor adequat, habitualment mitjançant broadcast.
- **Venciment (`valid-lifetime`):** si no es renova abans d'aquest moment, la concessió expira.

Exemple teòric d'una concessió de 8 hores iniciada a les 09.00: T1 al 50 % és a les 13.00; T2 al 87,5 % és a les 16.00; el venciment és a les 17.00. A la configuració concreta de Kea, comprova els valors explícits: a l'exemple didàctic `valid-lifetime: 4000`, `renew-timer: 1000` i `rebind-timer: 2000`, de manera que T1 i T2 són el 25 % i el 50 % de la durada.

| Missatge | Funció |
|---|---|
| `DHCPRELEASE` | El client comunica que deixa d'utilitzar la concessió. |
| `DHCPNAK` | El servidor rebutja una petició no vàlida en aquell context. |
| `DHCPDECLINE` | El client indica que la IP proposada sembla estar en ús. |

Si un servidor deixa de respondre, el client no perd immediatament una concessió encara vigent. Una adreça `169.254.x.x` és una pista que el client no té la configuració DHCP esperada; no és una concessió del pool de Kea ni, per si sola, prova quin component ha fallat.

### 2.4 Pools, reserves, relay i protecció

- **Àmbit/subxarxa:** xarxa per a la qual s'apliquen les regles DHCP.
- **Pool:** interval d'adreces que es poden assignar dinàmicament. La capacitat del pool no és la capacitat total de la subxarxa.
- **Reserva:** associació d'una IP a un client identificat per MAC o identificador. Ha de pertànyer a la subxarxa i es manté fora del pool dinàmic per evitar conflictes.
- **DHCP relay:** reenvia les peticions quan clients i servidor són en subxarxes diferents. El broadcast inicial no travessa ordinàriament el router; cal relay i un àmbit adequat per a la xarxa client.
- **Pool esgotat:** no hi ha adreces dinàmiques disponibles; revisa rang, exclusions, reserves i concessions vigents.
- **DHCP starvation:** un atac consumeix adreces del pool amb moltes peticions.
- **Servidor no autoritzat:** pot oferir configuració incorrecta. DHCP snooping als switches diferencia ports de confiança (cap a servidors/relay autoritzats) de ports no confiables on poden connectar clients.

<!-- IMATGE 05 — PROMPT: Crea una infografia original amb dues subxarxes i un router que fa de DHCP relay cap a un servidor central. Mostra el broadcast del client fins al relay, el reenviament cap al servidor i la resposta associada a l'àmbit correcte. Afegeix un requadre separat amb pool dinàmic, IP reservada i una nota que el snooping confia el port del servidor, no els ports dels clients. Etiquetes breus en català, sense afegir configuracions concretes. Fitxer suggerit: source/05_RA01_Repas/05-relay-pool-i-proteccio.png -->
![Relay Pool i Protecció](./source/05_RA01_Guia_Repas/05-relay-pool-i-proteccio.png)

---

## 3. Procediment pràctic i ordres Bash

> Aquest apartat és un resum breu de consulta. Substitueix els marcadors en MAJÚSCULES pels valors reals abans d'executar les ordres. Executa les ordres de modificació només després de confirmar la interfície, el fitxer i el valor que vols canviar.

### 3.1 Identificar la màquina i les interfícies

```bash
whoami
hostname
ip -br link
ip -4 -br addr
ip route
ip neigh
```

- `ip -br link`: interfícies, estat i MAC.
- `ip -4 -br addr`: adreces IPv4 i prefixos per interfície.
- `ip route`: rutes i ruta per defecte.
- `ip neigh`: veïns coneguts a l'enllaç local.

No dedueixis quin adaptador és NAT o xarxa interna només pel nom `enp0s3`/`enp0s8`: contrasta la MAC amb la configuració de VirtualBox.

### 3.2 Comprovar configuració i connectivitat

```bash
resolvectl status
ping -c 4 IP_DEL_GATEWAY
ping -c 4 IP_DEL_SERVIDOR
```

`resolvectl status` mostra la informació DNS disponible amb systemd-resolved. `ping` comprova si hi ha resposta ICMP, no prova per si sol DHCP, DNS ni l'estat d'un servei concret.

Al client Zorin amb NetworkManager:

```bash
nmcli device status
nmcli connection show
nmcli -f GENERAL,IP4,DHCP4 device show INTERFICIE
```

Per renovar una connexió, comprova primer el nom real del perfil. Exemple, només si el perfil s'anomena `Kea-LAB`:

```bash
sudo nmcli connection down "Kea-LAB"
sudo nmcli connection up "Kea-LAB"
```

### 3.3 Canviar i provar Netplan al servidor

Primer consulta els fitxers i la configuració actual:

```bash
ls -l /etc/netplan
sudo netplan get
```

Després de fer una còpia de seguretat i editar el fitxer YAML correcte, valida i prova des de la consola de la VM:

```bash
sudo netplan generate
sudo netplan try
ip -4 -br addr
ip route
```

`generate` detecta errors de generació; `try` aplica temporalment la configuració i permet revertir si no es confirma. Revisa l'adreçament abans de confirmar.

### 3.4 Instal·lar i configurar Kea

Amb connectivitat operativa a NAT:

```bash
sudo apt update
sudo apt upgrade
sudo apt install kea
kea-dhcp4 -v
systemctl list-unit-files 'kea*'
```

La configuració DHCPv4 de Kea és `/etc/kea/kea-dhcp4.conf`. Fes una còpia amb un nom que encara no existeixi abans d'editar:

```bash
sudo cp -a /etc/kea/kea-dhcp4.conf /etc/kea/kea-dhcp4.conf.abans-canvis
sudo nano /etc/kea/kea-dhcp4.conf
```

En el fitxer, comprova sobretot:

1. `interfaces-config.interfaces`: interfície interna real on ha d'escoltar Kea.
2. `subnet4`: adreça de xarxa i prefix correctes.
3. `pools`: rang dins de la subxarxa i sense incloure adreces reservades.
4. `option-data`: opcions `routers` i `domain-name-servers` coherents amb l'escenari.
5. `valid-lifetime`, `renew-timer` i `rebind-timer`: valors en segons i ordenats abans del venciment.
6. `lease-database`: ruta real del fitxer de concessions.
7. `reservations`: MAC correcta i IP lliure dins de la subxarxa, fora del pool.

Valida sempre abans d'aplicar:

```bash
sudo kea-dhcp4 -t /etc/kea/kea-dhcp4.conf
```

Si la prova falla, no reiniciïs el servei: corregeix el missatge i torna a validar. Si és correcta:

```bash
sudo systemctl restart kea-dhcp4-server
sudo systemctl enable kea-dhcp4-server
systemctl status kea-dhcp4-server --no-pager
systemctl is-enabled kea-dhcp4-server
sudo journalctl -u kea-dhcp4-server -b --no-pager -n 50
```

`active` descriu si el servei funciona ara; `enabled` indica si arrencarà automàticament. Si hi ha un error, els registres ajuden a trobar-ne la causa. No canviïs permisos de manera general ni ignoris un error de validació.

### 3.5 Verificar la concessió al client i al servidor

Al client, consulta IP, ruta, opcions DHCP i DNS:

```bash
ip -4 -br addr
ip route
nmcli -f GENERAL,IP4,DHCP4 device show INTERFICIE
resolvectl status
ping -c 4 IP_INTERNA_DEL_SERVIDOR
```

Al servidor, consulta la capçalera i les últimes entrades del fitxer indicat a `lease-database.name`:

```bash
sudo head -n 6 /var/lib/kea/kea-leases4.csv
sudo tail -n 15 /var/lib/kea/kea-leases4.csv
```

Relaciona IP, MAC/identificador de client, temps de concessió i subnet ID. El CSV pot contenir actualitzacions successives d'una mateixa concessió; no comptis files com si fossin clients actius.

Si captures tràfic amb Wireshark, filtra amb `dhcp` (o `bootp`, segons la versió) i comprova l'ordre dels missatges, els ports UDP i les opcions de l'ACK. Anota el que mostra la captura: Offer i ACK no són sempre broadcast.

<!-- IMATGE 06 — PROMPT: Crea un diagrama original de flux de treball pràctic per a DHCP Kea: identificar interfícies → editar Netplan/kea-dhcp4.conf → validar la configuració amb kea-dhcp4 -t → reiniciar i comprovar systemd/logs → verificar IP i ruta del client → relacionar concessió CSV i captura DORA. Utilitza icones simples, fletxes en ordre i etiquetes curtes en català. Remarca que cal aturar-se si falla la validació i que les adreces/interfícies s'han de substituir per les reals del laboratori. No imitis una captura de terminal. Fitxer suggerit: source/05_RA01_Repas/06-flux-practica-kea.png -->
![Flux Pràctica](./source/05_RA01_Guia_Repas/06-flux-practica-kea.png)

### 3.6 Errors habituals i primera comprovació

| Símptoma | Comença comprovant |
|---|---|
| No apareix la interfície esperada | Adaptadors a VirtualBox, MAC i `ip -br link`. |
| El client mostra `169.254.x.x` o no rep IP | Xarxa interna correcta, client en mode DHCP, servei actiu, interfície Kea i pool. |
| Kea no arrenca | `kea-dhcp4 -t`, després `systemctl status` i `journalctl`. |
| El client rep una IP del rang incorrecte | Si és al segment correcte i quin servidor DHCP ha respost; revisa la captura. |
| IP correcta però sense Internet | Gateway anunciat, existència del router i encaminament. DHCP correcte no implica que hi hagi un gateway funcional. |
| IP funciona però el nom no resol | DNS anunciat i accessible; comprova les dades efectives del client. |
| Reserva no s'aplica | MAC de la interfície del client, ubicació de `reservations` i nova negociació. |

---

## 4. Repàs ràpid abans de l'examen

Comprova que pots respondre sense consultar la guia:

- Quina diferència hi ha entre MAC, IP i port?
- Quantes adreces té IPv4 i quina funció té el prefix?
- Com calcules xarxa, broadcast, primer/últim host i capacitat d'un `/24`?
- Quina diferència hi ha entre gateway i servidor DNS?
- Què indiquen les capes TCP/IP i com es diferencia TCP d'UDP?
- Quina ruta s'escull si diverses coincideixen? Què representa `0.0.0.0/0`?
- Què fan NAT i PAT? Per què NAT no és equivalent a un tallafoc?
- Què proporciona DHCP i quins són els quatre passos DORA?
- Quins ports UDP utilitzen client i servidor DHCPv4?
- En què es diferencien una IP manual, una concessió dinàmica i una reserva?
- Què signifiquen T1, T2 i `valid-lifetime`? Quan expira la IP?
- Quina diferència hi ha entre una subxarxa, un pool i una reserva?
- Quan cal un DHCP relay i què aporta?
- Com distingiries un pool esgotat d'una adreça `169.254.x.x` o d'un servidor no autoritzat?
- Quines tres comprovacions cal completar: configuració vàlida, servei actiu i client amb concessió correcta?

Per a cada cas pràctic, presenta l'evidència en aquest ordre: **què esperaves → què has observat → què conclous → quina prova ho demostra**.

## Materials relacionats

- [01 · Repàs de xarxes TCP/IP](01_Repas_TCP_IP_Guide_Activity.md)
- [02 · Introducció al servei DHCP](02_Teoria_DHCP_Guide.md)
- [03 A · Configuració de DHCP amb Kea a Ubuntu Server](03_A_DHCP_Kea_Guia.md)
- [03 B · Activitat guiada de DHCP amb Kea i Zorin](03_B_DHCP_Kea_Activitat.md)
- [04 · Activitat d'avaluació de Kea DHCP](04_DHCP_Repas.md)
