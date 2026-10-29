![Portada: projecte de consultoria DNS](source/08_Consultoria_DNS/00-portada-laboratori-dns.png)

# 08 · Projecte de consultoria DNS: diagnosi i auditoria (Solved)

> **MP 0227 · Serveis de xarxa · 2n SMX · RA2**
>
> Versió resolta per al professorat, amb orientacions i respostes raonades. Les respostes DNS són dinàmiques: feu les consultes al vostre entorn i documenteu els valors que hi observeu, no només els exemples d'aquesta guia.

## Context professional

Heu estat contractats com a tècnics en pràctiques a l'empresa de consultoria i serveis IT **TechCloud Solutions**. El vostre tutor us assigna el primer projecte per a un client real: **Innovació Tecnològica del Maresme S.L.**

El client té problemes de connectivitat, lentitud en la resolució de dominis i dubtes sobre la configuració dels servidors de noms. Com a part del projecte, l'equip ha de:

1. **Formar el personal del client:** explicar el funcionament del DNS amb un recurs educatiu clar i visual.
2. **Diagnosticar i auditar la xarxa:** utilitzar eines de línia d'ordres en un entorn controlat per analitzar registres, respostes DNS i resolució local.

> **Nota de coherència de l'enunciat:** en diferents apartats de la proposta inicial apareixen dues durades per al vídeo i dos formats per a la guia. En aquesta versió s'unifica la durada en **5–10 minuts** i s'estableix el lliurament de la guia en format Markdown: **`guia.md`**.

## Objectius

En acabar la pràctica, hauràs de poder:

- Explicar què és DNS, com s'organitza i com es registra un domini.
- Diferenciar una consulta recursiva d'una consulta iterativa i identificar el rol dels servidors arrel i autoritatius.
- Interpretar registres A, NS, SOA i PTR, inclosos el TTL, el serial i el camp `rname`.
- Explicar la diferència entre zona directa/inversa i primària/secundària.
- Comparar una resposta d'un resolutor recursiu amb una resposta d'un servidor autoritatiu.
- Demostrar la resolució local mDNS i explicar-ne l'abast i el port.

## Preparació de l'entorn

### Requisits

- Màquina virtual amb **Zorin OS** o una distribució Linux equivalent.
- **Interfície 1 (`eth0`/`enp0s3`):** xarxa en mode **NAT** per tenir sortida a Internet.
- **Interfície 2 (`eth1`/`enp0s8`):** xarxa en mode **Adaptador pont (Bridged)**, amb la IP estàtica i els paràmetres que us hagi assignat el responsable tècnic.
- Un segon equip a la mateixa xarxa local per provar mDNS.

Els noms reals de les interfícies poden ser diferents. No inventeu l'adreça estàtica, el gateway ni el servidor DNS: utilitzeu els valors assignats al vostre laboratori.

### Comprovacions inicials

```bash
ip -br address
ip route
resolvectl status
hostnamectl --static
```

`ip -br address` mostra les adreces de les interfícies; `ip route` permet comprovar les rutes i la ruta per defecte, i `resolvectl status` mostra els DNS efectius del sistema quan s'utilitza `systemd-resolved`.

Si `dig` no està instal·lat, a Zorin/Ubuntu es pot instal·lar amb:

```bash
sudo apt update
sudo apt install dnsutils
```

## Fase teòrica · Mapa conceptual de DNS

Prepareu un mapa conceptual i una presentació breu dels conceptes següents.

### Conceptes que cal dominar

#### Què és DNS?

El **DNS** (*Domain Name System*) és un sistema distribuït i jeràrquic que permet consultar informació associada als noms de domini. Per exemple, una aplicació pot consultar quines adreces IPv4 té associades `tecnocampus.cat` i, amb una resposta, intentar connectar-se al servei corresponent.

DNS no transporta la pàgina web ni garanteix que el servei funcioni: resol el nom i retorna dades. La connexió al servei és una fase diferent.

#### Com es contracta un nom de domini?

1. Es comprova si el nom està disponible al registre del domini de nivell superior desitjat, com ara `.cat` o `.com`.
2. Es tria un **registrador acreditat** i es registra el domini durant el període ofert.
3. Es faciliten les dades de contacte i es compleixen les condicions del registre corresponent.
4. Es decideix on s'allotjarà la zona DNS i s'hi creen els registres necessaris.
5. Es publiquen al registrador els servidors de noms (NS) de la zona.

El **registrador** gestiona el registre del domini; el **proveïdor DNS** allotja i serveix la zona. Poden ser empreses diferents.

#### Elements, serveis i registres

- **Client/resolutor local:** rep la petició de l'aplicació i consulta el DNS configurat.
- **Resolutor recursiu:** cerca la resposta i la desa temporalment a la memòria cau segons el TTL.
- **Servidors arrel i del TLD:** orienten la cerca mitjançant referències cap al nivell següent.
- **Servidor autoritatiu:** publica les dades definitives d'una zona.
- **Registres:** A (IPv4), AAAA (IPv6), NS (servidor de noms), SOA (dades de control de zona), MX (correu), CNAME (àlies), TXT (text) i PTR (resolució inversa), entre d'altres.

#### Jerarquia i estructura

El DNS s'organitza com un arbre. Llegit de dreta a esquerra, `www.exemple.cat.` conté:

```text
.                         arrel
└── cat                   domini de nivell superior (TLD)
    └── exemple           domini registrat
        └── www           nom d'amfitrió o subdomini
```

El punt final representa l'arrel i sovint s'omet en l'ús quotidià. Una **zona** és la part de l'espai DNS que administra una organització; no sempre coincideix amb tot el domini.

#### Procés de resolució: recursiva i iterativa

- En una **consulta recursiva**, el client demana al seu resolutor una resposta final o un error. El client sol indicar que vol recursió; el resolutor fa la feina de cerca.
- En una **consulta iterativa**, el servidor consultat retorna la resposta que coneix o una referència a un altre servidor. El resolutor segueix les referències.
- Els **servidors arrel** no contenen la resposta de cada domini: indiquen a quin servidor del TLD cal preguntar.
- Els servidors del **TLD** indiquen els NS autoritatius del domini delegat.
- El servidor **autoritatiu** de la zona retorna els registres publicats per a aquesta zona.
- Si el resolutor ja té una resposta vigent a la memòria cau, la pot retornar sense recórrer tota la jerarquia.

#### Tipus de zones

- **Zona directa:** resol noms cap a adreces o altres dades, per exemple un registre A.
- **Zona inversa:** permet consultar el nom associat a una adreça IP mitjançant un PTR. No és una inversió automàtica del registre A: cal que existeixi el PTR a la zona inversa administrada.
- **Zona primària:** còpia editable de la zona, on s'originen les modificacions.
- **Zona secundària:** còpia de només lectura que obté les dades de la primària mitjançant transferències de zona autoritzades. Augmenta la disponibilitat i reparteix consultes.

#### Resolució local amb mDNS

**mDNS** (*Multicast DNS*) permet que equips d'un mateix enllaç local consultin i anunciïn noms sense dependre d'un servidor DNS central. A Linux és habitual utilitzar Avahi; els noms locals solen tenir el sufix `.local`.

- Utilitza UDP **5353** i adreces multicast (IPv4 `224.0.0.251`, IPv6 `ff02::fb`).
- El seu abast és l'enllaç local; els encaminadors normalment no reenvien aquestes consultes.
- Cal que els equips tinguin mDNS habilitat i que la xarxa permeti el trànsit multicast.
- És útil per descobrir equips i serveis en xarxes petites, però no substitueix el DNS convencional per a noms públics ni travessa xarxes per si sol.

### Mapa conceptual orientatiu

```text
DNS
├── Serveix per consultar informació de noms
│   ├── registres: A, AAAA, NS, SOA, MX, CNAME, PTR...
│   └── resposta temporal: TTL i memòria cau
├── S'organitza jeràrquicament
│   └── arrel → TLD → domini → subdomini
├── Resol noms
│   ├── client → consulta recursiva al resolutor
│   └── resolutor → consultes iteratives a arrel, TLD i autoritatiu
├── S'administra en zones
│   ├── directa / inversa
│   └── primària / secundària
└── Pot resoldre noms a la xarxa local amb mDNS
    └── multicast, UDP 5353, abast d'enllaç (.local)
```

### Entregable teòric

Graveu una **píndola formativa individual de 5–10 minuts** que expliqui de manera clara, breu i visual els conceptes anteriors, adreçada al personal tècnic del client. Incloeu-hi el mapa conceptual. El vídeo no forma part del solucionari d'aquest document.

## Fase pràctica · Diagnosi de noms amb CLI

Feu les consultes des de la terminal de Zorin OS i anoteu els resultats obtinguts. Les quatre consultes `dig` utilitzen `tecnocampus.cat` per tenir un exemple comú. La consulta PTR es fa sobre una de les adreces obtingudes a la primera consulta.

### Part A · Diagnosi avançada amb `dig`

#### Comanda 1 · Consulta bàsica del registre A

```bash
dig tecnocampus.cat A
```

**Anàlisi:** identifiqueu les adreces IPv4 de la secció `ANSWER SECTION`, el TTL de cada resposta i el servidor consultat que apareix a `SERVER`.

#### Comanda 2 · Consulta dels servidors de noms (NS)

```bash
dig tecnocampus.cat NS
```

**Anàlisi:** indiqueu quins servidors de noms són autoritatius per a `tecnocampus.cat`.

#### Comanda 3 · Consulta detallada SOA

```bash
dig tecnocampus.cat SOA
```

**Anàlisi:** identifiqueu el camp `rname` (correu del responsable en format DNS) i el número de sèrie (`serial`). Anoteu també el servidor primari i els temps del SOA que apareguin.

#### Comanda 4 · Consulta de resolució inversa

Substituïu l'adreça per una de les adreces IPv4 retornades per la comanda 1:

```bash
dig -x 108.157.113.8
```

**Anàlisi:** identifiqueu el registre PTR retornat i el nom de domini associat a l'adreça consultada. Si no hi ha PTR, indiqueu-ho: una adreça A no implica que existeixi un PTR.

### Part B · Resolució amb `nslookup` en mode interactiu

Obriu el mode interactiu:

```text
nslookup
```

Les ordres útils són:

| Ordre | Funció |
|---|---|
| `set type=A` | Selecciona el tipus de registre A. També es poden provar AAAA, MX, NS, SOA, TXT o ALL. |
| `server NOM_o_IP` | Selecciona el servidor DNS al qual es faran les consultes següents. |
| `exit` | Tanca el mode interactiu. |

#### Comanda 5 · Consulta bàsica no autoritativa

Al prompt `>`:

```text
set type=A
tecnocampus.cat
```

**Anàlisi:** indiqueu les adreces retornades i per què `nslookup` pot marcar la resposta com a **no autoritativa** (*Non-authoritative answer*).

#### Comanda 6 · Consulta directa a un servidor autoritatiu

Trieu un dels NS obtinguts a la comanda 2. Per exemple:

```text
server ns-1071.awsdns-05.org
set type=A
tecnocampus.cat
```

També podeu utilitzar la seva adreça IP, obtinguda amb `dig ns-1071.awsdns-05.org A`, si el vostre `nslookup` no accepta el nom del servidor.

**Anàlisi:** compareu la resposta amb la comanda 5. Comproveu si ha desaparegut l'etiqueta de resposta no autoritativa i expliqueu què indica aquesta diferència.

### Part C · Resolució local mDNS

1. A Zorin OS, consulteu el nom de l'equip:

   ```bash
   hostnamectl --static
   ```

2. Comproveu que el servei mDNS d'Avahi està disponible i en funcionament:

   ```bash
   systemctl status avahi-daemon
   ```

3. Des d'un altre equip de la mateixa xarxa local, consulteu el nom amb `.local`:

   ```bash
   avahi-resolve-host-name NOM_DE_L_EQUIP.local
   ```

   Per exemple, si `hostnamectl --static` mostra `zorin-client`, consulteu `zorin-client.local`. També podeu provar `getent hosts zorin-client.local` si el sistema té configurat NSS per a mDNS.

Si falten les eines d'Avahi, instal·leu-les a Zorin/Ubuntu:

```bash
sudo apt update
sudo apt install avahi-daemon avahi-utils libnss-mdns
```

**Anàlisi:** expliqueu com un equip anuncia o consulta el nom mitjançant multicast, quin port utilitza mDNS (UDP 5353) i quin avantatge aporta quan no hi ha cap servidor DNS local.

> **Límit important de la prova:** els dos equips han de ser al mateix segment de xarxa que admet multicast. En una VM, el mode pont sol ser l'opció adequada per comunicar-se amb altres equips de la LAN; la interfície NAT pot estar en un segment virtual separat. La resolució mDNS no acostuma a travessar encaminadors ni xarxes Wi-Fi amb aïllament de clients.

## Lliurables

Pugeu a Moodle una carpeta o un arxiu comprimit anomenat:

```text
Cognoms_Nom_Practica_DNS_Comandes.zip
```

Ha de contenir:

1. **Document amb l'enllaç al vídeo** de la fase teòrica (YouTube, Vimeo o Google Drive), compartit amb permisos de visualització.
2. **`guia.md`**, que inclogui:
   - captures reals i llegibles de les quatre consultes `dig`, les dues proves amb `nslookup` i les proves mDNS;
   - respostes raonades als punts d'anàlisi;
   - la configuració de les dues interfícies de Zorin OS, demostrada amb `ip a` o `ifconfig`.

No substituïu les captures per sortides inventades. Si una resposta difereix dels exemples d'aquesta guia, conserveu la sortida real i expliqueu-ne la causa probable (canvi del DNS, memòria cau, TTL o proveïdor).

## Criteris d'avaluació

| Bloc | Pes | Què s'avalua |
|---|---:|---|
| Píndola formativa en vídeo | 30% · 3 punts | Claredat, rigor tècnic, adequació al client, mapa conceptual i durada de 5–10 minuts. |
| Auditoria de diagnosi amb CLI | 40% · 4 punts | Execució de les quatre consultes `dig` i les dues proves `nslookup`; interpretació de registres, autoritatius i respostes no autoritatives. |
| Resolució local mDNS | 20% · 2 punts | Demostració de resolució local i explicació del funcionament, l'abast i el port. |
| Format i documentació | 10% · 1 punt | Qualitat de la guia lliurada en Markdown (`guia.md`), amb estructura, títols i format coherents; captures, configuració de xarxa i eines, i redacció formal. Per aportar traçabilitat i demostrar competències digitals, cal incloure un apartat de fonts i eines consultades: si s'ha utilitzat IA, indicar l'eina i enllaçar la conversa o el resultat (si es pot compartir) i les fonts verificables; si no, enllaçar les fonts consultades. |
| **Total** | **100% · 10 punts** | |

## Solucionari raonat

> Aquest solucionari serveix per entendre què cal observar. No substitueix les evidències del vostre propi laboratori: les respostes de DNS poden canviar segons el moment, el resolutor i la ubicació de la consulta.

### Fase teòrica

- **DNS:** sistema distribuït de consulta de dades associades a noms. La resolució facilita trobar una adreça o un altre registre; no comprova que l'aplicació o el servidor final estiguin disponibles.
- **Contractació:** es registra el nom amb un registrador per al TLD escollit, es decideix quin proveïdor allotjarà el DNS i es publiquen els NS al registre. El registrador i l'allotjament DNS poden ser serveis diferents.
- **Jerarquia:** la cerca segueix arrel → TLD → domini delegat → subdomini o registre. L'arrel i el TLD donen referències; l'autoritatiu publica els registres de la zona.
- **Recursiva/iterativa:** el client demana habitualment una resposta final al resolutor recursiu. Aquest pot consultar de manera iterativa els servidors arrel, TLD i autoritatius, llevat que ja tingui una resposta vigent a la memòria cau.
- **Zones directes i inverses:** la directa resol noms cap a dades com una IP; la inversa utilitza PTR per associar una adreça a un nom. Cal que el responsable de la zona inversa publiqui el PTR.
- **Primària i secundària:** la primària és la còpia d'origen editable; la secundària replica la zona i normalment és de només lectura.
- **mDNS:** els equips del mateix enllaç intercanvien consultes i respostes multicast a UDP 5353, habitualment amb noms `.local`. No cal un DNS central, però sí suport als clients i connectivitat multicast local.

### Resultats de les consultes `dig`

En una consulta real observada per preparar aquesta guia, els resultats van ser:

| Consulta | Exemple de resultat observat | Interpretació |
|---|---|---|
| `dig tecnocampus.cat A` | `108.157.113.8`, `108.157.113.59`, `108.157.113.83` i `108.157.113.90`; TTL observat de 5 segons en aquell moment. | Són adreces IPv4 retornades per al nom. El TTL i les IP poden canviar; anoteu els valors de la vostra consulta. |
| `dig tecnocampus.cat NS` | `ns-1689.awsdns-19.co.uk`, `ns-535.awsdns-02.net`, `ns-130.awsdns-16.com` i `ns-1071.awsdns-05.org`. | Són els quatre NS publicats per a la zona en la consulta observada. L'ordre pot variar. |
| `dig tecnocampus.cat SOA` | Primari `ns-1071.awsdns-05.org`; `rname` `awsdns-hostmaster.amazon.com.`; serial `1`. | En la notació SOA, el primer punt del `rname` separa el nom del responsable del domini de correu; aquí correspon a `awsdns-hostmaster@amazon.com`. |
| `dig -x 108.157.113.8` | PTR `server-108-157-113-8.mad56.r.cloudfront.net.` | Aquesta IP retornava el nom invers del proveïdor CDN, no `tecnocampus.cat`. L'adreça A del web i el PTR no han de coincidir. |

En la mateixa observació, el SOA també mostrava `refresh 7200`, `retry 900`, `expire 1209600` i `minimum 86400` segons. Aquests valors pertanyen al registre SOA; no són el TTL de la resposta A. El TTL que interessa per a cada registre apareix a la columna corresponent de la secció de resposta.

Per llegir la sortida:

- `ANSWER SECTION`: registres que responen la consulta.
- `AUTHORITY SECTION`: informació d'autoritat o referències addicionals; no confongueu aquesta secció amb l'adreça del servidor consultat.
- `SERVER`: servidor DNS al qual `dig` ha enviat la consulta.
- La línia de flags inclou `aa` si la resposta rebuda és autoritativa. Una resposta recursiva correcta pot no tenir aquesta flag.

### Resultats de `nslookup`

#### Consulta bàsica

Amb `set type=A` i `tecnocampus.cat`, el resolutor configurat al sistema retorna una o més adreces A. `Non-authoritative answer` vol dir que el servidor consultat no és l'autoritat de la zona: normalment és un resolutor recursiu que obté la resposta d'un autoritatiu o de la seva memòria cau. **No vol dir que la resposta sigui incorrecta.**

#### Consulta directa a l'autoritat

Quan es selecciona un NS de la comanda 2 i es repeteix la consulta A, el servidor autoritatiu pot respondre directament amb les adreces publicades per la zona. En aquest cas l'etiqueta `Non-authoritative answer` normalment desapareix. La diferència és qui ha respost i si ho ha fet com a autoritat de la zona, no necessàriament el contingut del registre.

L'exemple `a9-66.akam.net` de l'enunciat no s'ha de donar per vigent: cal utilitzar un dels NS que retorni la consulta NS actual. En els resultats observats per aquesta guia, els NS pertanyien a AWS.

### Resolució mDNS

`hostnamectl --static` retorna el nom d'amfitrió configurat, per exemple `zorin-client`. Amb Avahi en funcionament, un altre equip compatible a la mateixa xarxa pot consultar `zorin-client.local` i obtenir una adreça local. El nom es resol amb missatges multicast, no preguntant a un servidor DNS central.

Si no funciona, comproveu, en aquest ordre:

1. que Avahi està instal·lat i actiu als equips necessaris;
2. que heu escrit el nom real seguit de `.local`;
3. que els equips són al mateix segment i que el multicast UDP 5353 no està filtrat;
4. que esteu provant per la interfície pont i no només per una xarxa NAT aïllada;
5. si `getent` falla, si NSS mDNS està instal·lat; `avahi-resolve-host-name` és una prova explícita d'Avahi.

La resolució mDNS local pot funcionar encara que falli la consulta d'un domini públic: són mecanismes i camins de xarxa diferents. Igualment, observar mDNS no demostra que DHCP hagi concedit una adreça ni que el servidor DNS convencional funcioni.

## Fonts de consulta

- [RFC 1034 · Domain Names: Concepts and Facilities](https://www.rfc-editor.org/rfc/rfc1034.html)
- [RFC 1035 · Domain Names: Implementation and Specification](https://www.rfc-editor.org/rfc/rfc1035.html)
- [RFC 6762 · Multicast DNS](https://www.rfc-editor.org/rfc/rfc6762.html)
- [RFC 6763 · DNS-Based Service Discovery](https://www.rfc-editor.org/rfc/rfc6763.html)
- [IANA · Root Zone Database](https://www.iana.org/domains/root/db/)
- [Avahi · Documentació del projecte](https://www.avahi.org/)
