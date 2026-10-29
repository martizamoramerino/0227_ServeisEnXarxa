![Portada: projecte de consultoria DNS](source/08_Consultoria_DNS/00-portada-laboratori-dns.png)

# Projecte de Consultoria DNS – *TechCloud Solutions* (Unsolved)

## 1. Context professional (estudi de cas)

Heu estat contractats com a tècnics en pràctiques a l'empresa de consultoria i serveis IT **TechCloud Solutions**. El vostre tutor d'empresa us assigna el vostre primer projecte per a un client real: l'empresa **Innovació Tecnològica del Maresme S.L.**

Aquest client està experimentant problemes de connectivitat, lentitud en la resolució de dominis i dubtes sobre la configuració dels seus servidors de noms. Com a part de l'acord de consultoria, el vostre equip ha de realitzar dues tasques principals:

1. **Formar el personal del client:** Crear un recurs educatiu per explicar-los com funciona el protocol DNS a nivell teòric.
2. **Diagnosi i auditoria de la xarxa:** Executar una auditoria amb eines CLI en un entorn controlat per analitzar registres, temps de resposta i resolució local.

## 2. Fase teòrica: creació del mapa conceptual

Per assegurar la màxima qualitat abans de presentar-ho al client, els directors tècnics de TechCloud Solutions us demanen que us prepareu un mapa conceptual i una presentació dels conceptes clau següents.

### Conceptes a dominar

- **Què és un DNS:** objectius amb exemples gràfics.
- Com contractar un **nom de domini**.
- **Elements i serveis d'un DNS**.
- **Jerarquia i estructura:** organització en arbre del DNS.
- **Procés de resolució:** diferència entre consultes iteratives i recursives. Rol dels *Root Servers* i servidors autoritatius.
- **Tipus de zones:** zona directa vs. zona inversa. Zona primària vs. zona secundària.
- **Resolució local:** mecanismes de resolució sense servidor centralitzat entre equips de la mateixa xarxa local (protocol **mDNS**).

### Entregable de la fase teòrica

> **Píndola formativa (vídeo):** Graveu una píndola individual i formativa en format vídeo, amb una durada d'entre **5–10 minuts**, on s'expliquin de forma clara, breu i visual tots els conceptes anteriors, adreçats al personal tècnic del client, i el mapa conceptual.

## 3. Fase pràctica: diagnosi de noms (auditoria amb CLI)

### Requisits de l'entorn de treball

- **Sistema operatiu:** màquina virtual amb **Zorin OS** (o distribució Linux equivalent).
- **Configuració de xarxa:**
  - **Interfície 1 (`eth0`/`enp0s3`):** mode **NAT** (per tenir sortida a Internet).
  - **Interfície 2 (`eth1`/`enp0s8`):** mode **Adaptador pont (Bridged)** amb la IP estàtica i els paràmetres assignats pel vostre responsable tècnic.

### Part A. Diagnosi avançada amb `dig` (Linux/macOS)

Executeu les comandes indicades a la consola de Linux i analitzeu els resultats obtinguts. Per mantenir les consultes relacionades, utilitzeu `tecnocampus.cat` per als registres A, NS i SOA. A la consulta inversa, substituïu el marcador per una de les adreces IP que us hagi retornat la primera comanda.

#### Comanda 1: consulta bàsica de registre A

```bash
dig tecnocampus.cat A
```

- **Anàlisi requerida:** Identifiqueu la IP de resposta, el valor del TTL i l'adreça del servidor DNS que ha respost a la consulta.

#### Comanda 2: consulta de servidors de noms (NS)

```bash
dig tecnocampus.cat NS
```

- **Anàlisi requerida:** Quins són els servidors de noms (*Name Servers*) autoritatius per a aquest domini?

#### Comanda 3: consulta detallada SOA

```bash
dig tecnocampus.cat SOA
```

- **Anàlisi requerida:** Quina és la informació del correu de l'administrador (`rname`) i quin és el número de sèrie (`serial`) del domini?

#### Comanda 4: consulta de resolució inversa

```bash
dig -x <IP_obtinguda_a_la_comanda_1>
```

- **Anàlisi requerida:** Quina informació sobre els registres `PTR` s'obté i a quin nom de domini correspon aquesta IP?

### Part B. Comprovació de resolució amb `nslookup` (mode interactiu amb Windows o Linux)

Exploreu l'ús de l'eina `nslookup` en mode interactiu executant la comanda `nslookup` sense arguments des de la terminal fins a veure el prompt (`>`).

**Comandes clau del mode interactiu:**

- `set type=TIPO`: Defineix el tipus de registre a consultar (`A`, `AAAA`, `MX`, `NS`, `SOA`, `TXT` o `ALL`).
- `server IP_o_NOM`: Canvia el servidor DNS al qual es realitzarà la consulta.
- `exit`: Surt del mode interactiu.

#### Comanda 5: consulta bàsica no autoritativa

1. Dins del prompt de `nslookup`, configureu: `set type=A`.
2. Escriviu el domini de consulta: `tecnocampus.cat`.

- **Anàlisi requerida:** Quina resposta dona la consola? Per què indica textualment que la resposta és **no autoritativa** (*Non-authoritative answer*)?

#### Comanda 6: consultes autoritatives directes

1. Canvieu el servidor de consultes escrivint `server IP`, on la IP sigui la del primer servidor de noms autoritatiu obtingut a la comanda 2 per al domini `tecnocampus.cat` (per exemple, `server a9-66.akam.net` o la seva IP corresponent).
2. A continuació, configureu `set type=A` i consulteu de nou el domini `tecnocampus.cat`.

- **Anàlisi requerida:** Quines diferències s'observen en la resposta obtinguda respecte a la comanda 5? Comproveu si l'indicador "no autoritatiu" ha desaparegut.

### Part C. Resolucions locals i protocol mDNS

Comproveu el funcionament de la resolució local sense servidor DNS dedicat (útil per a xarxes locals petites):

1. Verifiqueu el nom d'equip (*hostname*) de la vostra màquina Zorin OS.
2. Des d'un altre equip de la mateixa xarxa local (o xarxa bridged), executeu la resolució local mitjançant el domini `.local`:

```

```

- **Anàlisi requerida:** Descriviu com funciona el protocol **mDNS** (*Multicast DNS*), quin port utilitza (UDP 5353) i quin avantatge aporta en entorns on no es disposa d'un servidor DNS configurat.

## 4. Lliurables del projecte

Pugeu a la plataforma d'aprenentatge (Moodle) una carpeta o un arxiu comprimit amb el nom `Cognoms_Nom_Practica_DNS_Comandes.zip` que contingui:

1. **Enllaç al vídeo (fase teòrica):** document amb l'enllaç (YouTube, Vimeo o Google Drive) a la píndola formativa (10–15 minuts).
2. **Document `guia.md`**, que inclogui:
   - Captures de pantalla clares de l'execució de les **6 comandes** (4 de `dig` + 2 de `nslookup`) i de les proves de **mDNS**.
   - Respostes detallades a tots els punts d'anàlisi de la pràctica.
   - Configuració de les dues interfícies de xarxa de l'equip Zorin OS (`ip a` / `ifconfig`).

## 5. Criteris d'avaluació i qualificació

L'avaluació d'aquesta activitat de consultoria es realitzarà sobre un total de **10 punts**, distribuint la qualificació segons els blocs de treball següents:

- **Píndola formativa en vídeo (30% – 3 punts):** Avaluació de la claredat expositiva, el rigor tècnic en l'explicació dels conceptes de la fase teòrica (jerarquia, registres, zones, TTL, SOA i mDNS), l'adequació del llenguatge al client i el compliment de la durada establerta (entre 10 i 15 minuts).
- **Auditoria de diagnosi amb CLI (40% – 4 punts):** Execució correcta de les 4 comandes amb l'eina `dig` i les 2 proves en mode interactiu amb `nslookup`. Es valorarà la capacitat d'analitzar les respostes obtingudes, la identificació correcta dels servidors autoritatius, els registres i la justificació tècnica de les respostes no autoritatives.
- **Proves de resolució local mDNS (20% – 2 punts):** Demostració pràctica de la resolució de noms en xarxa local sense servidor centralitzat i explicació teòrica del funcionament del protocol mDNS i el seu port de treball.
- **Format, documentació i presentació (10% – 1 punt):** Qualitat de la guia lliurada en Markdown (`guia.md`), amb estructura clara, títols i format coherents; inclusió de les captures requerides, configuració correcta de xarxa i eines, i redacció formal. Per demostrar les competències digitals i aportar traçabilitat, incloeu-hi un apartat de fonts i eines consultades: si feu servir IA, indiqueu quina eina heu utilitzat i afegiu l'enllaç a la conversa o al resultat (si es pot compartir) i a les fonts verificables; si no feu servir IA, afegiu els enllaços a les fonts consultades.
