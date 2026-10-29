![Solucionari Examen](./source/06_SolucionariExamen/00-portada-solucionari-dhcp.png)
# RA01 · Solucionari del control de DHCP

Aquest document recull els enunciats del control de la RA1 sobre DHCP i Kea, amb respostes raonades. Les adreces i les ordres s'interpreten d'acord amb les dades de cada pregunta.

## 1. Funció de DHCP i avantatges

### Enunciat

Explica la funció principal d'un servidor DHCP i dos avantatges respecte a la configuració IP manual en una xarxa amb més de 100 ordinadors. Què passaria si haguéssim de canviar la porta d'enllaç en els dos escenaris?

### Resposta

Un servidor DHCP assigna automàticament als clients la configuració de xarxa, com ara l'adreça IP, la màscara, la porta d'enllaç i els servidors DNS, durant un període de concessió.

Dos avantatges respecte a configurar cada ordinador manualment són:

1. **Administració centralitzada:** l'administrador configura les opcions en un únic servidor i no ha de repetir els canvis a cada ordinador.
2. **Menys errors i conflictes:** es redueix el risc d'introduir una màscara, una porta d'enllaç o una IP incorrecta, o d'assignar la mateixa IP a dos equips.

Si es canvia la porta d'enllaç, amb configuració manual cal modificar-la en més de 100 ordinadors, un per un. Amb DHCP n'hi ha prou amb canviar l'opció de router al servidor. Els clients rebran el valor nou quan renovin la concessió; fins aleshores, alguns poden continuar utilitzant la porta d'enllaç antiga. En un canvi planificat, es pot forçar la renovació dels clients o preparar-la amb antelació.

## 2. Tipus d'assignació DHCP i reserves

### Enunciat

Descriu breument els tres tipus d'assignació DHCP: dinàmica, automàtica o permanent, i mitjançant reserva. Quines dades necessitaria l'administrador per configurar una reserva i en quin tipus d'equip seria útil?

### Resposta

- **Dinàmica:** el servidor tria una IP disponible d'un pool i la concedeix per un temps limitat. El client la pot renovar i, més endavant, l'adreça es pot assignar a un altre equip.
- **Automàtica o permanent:** el servidor selecciona una adreça disponible i la manté associada al client de manera permanent o indefinida, segons la implementació.
- **Reserva:** l'administrador associa una IP concreta a la identificació d'un client. El client continua obtenint la configuració per DHCP, però rep habitualment la mateixa IP.

Per crear una reserva cal conèixer la MAC (o l'identificador de client) i la IP que es vol reservar; també es pot indicar un nom d'equip. És útil per a dispositius que necessiten una adreça estable, com ara una impressora, un servidor o un equip de xarxa.

## 3. Procés DORA

### Enunciat

Esquematitza el procés DORA durant una assignació inicial d'IP. Indica els quatre missatges, qui els envia i quina funció té cadascun. Explica també per què el DHCPREQUEST inicial es transmet per broadcast.

a. Qui l'envia (client o servidor).  
b. Si s'envia en mode broadcast o unicast (i per què en el cas del REQUEST).  
c. La funció principal del missatge.

### Resposta

| Ordre | Missatge | Emissor → receptor | Enviament habitual en l'assignació inicial | Funció |
|---|---|---|---|---|
| 1 | `DHCPDISCOVER` | Client → servidors DHCP | Broadcast | El client encara no té una IP configurada ni coneix el servidor; busca servidors disponibles. |
| 2 | `DHCPOFFER` | Servidor → client | Broadcast o unicast, segons l'estat i les indicacions del client | El servidor proposa una IP i les opcions de xarxa. |
| 3 | `DHCPREQUEST` | Client → servidors DHCP | Broadcast | El client demana formalment l'oferta seleccionada. |
| 4 | `DHCPACK` | Servidor seleccionat → client | Broadcast o unicast, segons l'estat i les indicacions del client | El servidor confirma la concessió i la configuració. |

El `DHCPREQUEST` inicial es difon per broadcast perquè tots els servidors que hagin ofert una adreça sàpiguen quina oferta ha triat el client. El servidor seleccionat continua el procés; els altres poden retirar les seves ofertes i retornar les adreces als pools. En DHCP, el client utilitza el port UDP 68 i el servidor el port UDP 67.

## 4. Adreça 169.254.88.12

### Enunciat

Un ordinador configurat per obtenir la IP automàticament rep l'adreça `169.254.88.12`.

a. Què ens indica aquesta adreça sobre la configuració de xarxa?  
b. Pot accedir a Internet? Podria comunicar-se amb algun altre ordinador de la xarxa local? Justifica breument la resposta.

### Resposta

És una adreça IPv4 d'enllaç local (APIPA), del rang `169.254.0.0/16`. Normalment indica que el client no ha rebut resposta d'un servidor DHCP i s'ha autoconfigurat amb una adreça local.

En condicions normals no pot accedir a Internet: les adreces d'enllaç local no s'encaminen i no hi ha una porta d'enllaç vàlida obtinguda per DHCP. Pot comunicar-se amb un altre equip del mateix segment local si aquest també té una adreça compatible i hi ha connectivitat a nivell d'enllaç; no podrà comunicar-se així amb equips d'altres subxarxes.

## 5. Comprovacions del servidor i del client

### Enunciat

En una pràctica hem configurat un servidor DHCP amb Kea a Ubuntu Server i un client Zorin OS que ha d'obtenir automàticament la seva configuració IP.

a. Quina comprovació faries a Ubuntu Server per verificar que el servei DHCP està actiu i funcionant?  
b. Quina comprovació faries a Zorin OS per verificar que ha rebut correctament una configuració IP per DHCP? Quina informació esperaries trobar?

### Resposta

**a. A Ubuntu Server**, comprovaria l'estat del servei i els registres:

```bash
sudo systemctl status kea-dhcp4-server
sudo journalctl -u kea-dhcp4-server -n 50 --no-pager
```

El servei hauria d'estar actiu sense errors rellevants. Als registres buscaria l'inici correcte de Kea i les peticions/respostes DHCP del client. Si cal validar la sintaxi del fitxer de configuració:

```bash
sudo kea-dhcp4 -t /etc/kea/kea-dhcp4.conf
```

**b. A Zorin OS**, consultaria la configuració de la interfície i les rutes:

```bash
ip -4 addr
ip route
nmcli device show
```

Esperaria trobar una IP de la subxarxa i del pool configurats, el prefix o la màscara correctes, la porta d'enllaç anunciada i els servidors DNS previstos. La ruta per defecte hauria d'apuntar a la porta d'enllaç correcta.

## 6. Interpretació de la configuració de Kea

### Enunciat

A continuació es mostra un fragment del fitxer de configuració del servidor Kea DHCP (`kea-dhcp4.conf`):

```json
{
  "Dhcp4": {
    "interfaces-config": {
      "interfaces": ["eth0"]
    },
    "valid-lifetime": 3600,
    "renew-timer": 1000,
    "subnet4": [
      {
        "subnet": "192.168.10.0/24",
        "pools": [
          {
            "pool": "192.168.10.50 - 192.168.10.150"
          }
        ],
        "option-data": [
          {
            "name": "routers",
            "data": "192.168.10.1"
          },
          {
            "name": "domain-name-servers",
            "data": "8.8.8.8, 1.1.1.1"
          }
        ],
        "reservations": [
          {
            "hw-address": "00:11:22:33:44:55",
            "ip-address": "192.168.10.10"
          }
        ]
      }
    ]
  }
}
```

Respon les preguntes següents a partir de la configuració:

a. Quin és el rang d'adreces IP dinàmiques que s'assignaran als clients?  
b. Quina porta d'enllaç i quins servidors DNS rebran els equips de la xarxa?  
c. Si connectem un equip amb la MAC `00:11:22:33:44:55`, quina adreça IP obtindrà i per què?  
d. Identifica els valors `renew-timer` i `valid-lifetime`. Explica què representa cadascun i converteix-los a minuts o hores.  
e. La IP reservada `192.168.10.10` es troba fora del rang dinàmic. Per què pot ser una bona decisió organitzar les adreces d'aquesta manera?

### Resposta

**a.** El pool dinàmic va de `192.168.10.50` a `192.168.10.150`, incloses totes dues adreces. Són 101 adreces possibles.

**b.** La porta d'enllaç és `192.168.10.1`. Els servidors DNS són `8.8.8.8` i `1.1.1.1`.

**c.** Rebrà `192.168.10.10`, perquè la MAC coincideix amb la reserva definida. La reserva és fora del pool dinàmic, però pertany a la subxarxa `192.168.10.0/24`.

**d.**

- `renew-timer: 1000`: el client intentarà renovar la concessió al cap de 1.000 segons, és a dir, **16 minuts i 40 segons**.
- `valid-lifetime: 3600`: la concessió és vàlida durant 3.600 segons, és a dir, **1 hora**. Si no es renova, en arribar al venciment el client ha de deixar d'utilitzar aquella IP.

**e.** Deixar les adreces reservades fora del pool dinàmic facilita separar els equips amb IP estable dels clients que reben una IP variable. Així s'evita que el pool assigni dinàmicament una adreça que es vol mantenir fixa per a un dispositiu i es redueix el risc de conflictes.

## 7. Diagnosi de dos casos

### Enunciat

Analitza els dos casos i, per a cadascun, identifica el problema de configuració, proposa una possible causa i indica una comprovació tècnica que faries per verificar-la. Finalment, explica quina acció aplicaries per recuperar la connectivitat.

- **Cas A:** l'ordinador 01 té l'adreça IP `169.254.88.12` amb màscara `255.255.0.0` i no pot accedir a cap recurs de la xarxa ni a Internet.
- **Cas B:** l'ordinador 02 té l'adreça IP `192.168.10.75`, però quan intenta navegar a Internet la connexió falla. En revisar la configuració de xarxa, s'observa que la seva porta d'enllaç és `192.168.10.250`, quan la IP real del router és `192.168.10.1`.

Analitza les dues situacions i argumenta quina és la causa probable de cada fallada i quina acció concreta s'ha de dur a terme per resoldre-ho.

### Resposta

**Cas A**

- **Problema i causa probable:** `169.254.88.12/16` és una adreça APIPA. El client no ha rebut una concessió DHCP; una causa possible és que Kea estigui aturat o que no escolti a la interfície de la xarxa interna. També es podria haver esgotat el pool o haver-hi un problema de connexió entre client i servidor.
- **Comprovació:** a l'Ubuntu Server, revisar `sudo systemctl status kea-dhcp4-server` i `sudo journalctl -u kea-dhcp4-server -n 50 --no-pager`; al client, verificar la interfície i la IP amb `ip -4 addr`. Cal confirmar que servidor i client són al mateix segment i que el pool té adreces disponibles.
- **Acció correctiva:** corregir la interfície, la configuració o la disponibilitat del servei/pool segons el problema trobat, iniciar o reiniciar Kea si correspon i renovar la concessió del client. Quan DHCP funcioni, el client hauria d'obtenir una IP de `192.168.10.0/24` i les opcions de xarxa correctes.

**Cas B**

- **Problema i causa probable:** la IP `192.168.10.75` és coherent amb la subxarxa, però la porta d'enllaç anunciada (`192.168.10.250`) no coincideix amb la IP real del router (`192.168.10.1`). Per tant, el trànsit destinat a Internet s'envia a una adreça incorrecta.
- **Comprovació:** consultar la ruta per defecte amb `ip route` i comprovar la configuració rebuda amb `nmcli device show`. Es pot provar la connectivitat amb `ping 192.168.10.1` i comparar-la amb la porta d'enllaç configurada.
- **Acció correctiva:** corregir a Kea l'opció `routers` perquè anunciï `192.168.10.1`, validar la configuració, aplicar-la reiniciant el servei i renovar la concessió del client. Després, comprovar que la ruta per defecte apunta al router correcte i tornar a provar la connectivitat.
