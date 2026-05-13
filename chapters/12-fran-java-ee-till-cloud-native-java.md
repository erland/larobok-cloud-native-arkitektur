# Kapitel 12: Från Java EE till cloud native Java

## Varför detta kapitel finns

Många organisationer som moderniserar mot OpenShift börjar med Java EE-applikationer som burit verksamheten under lång tid. De innehåller ofta central affärslogik, är beroende av Oracle, använder IBM MQ och är driftsatta i en traditionell applikationsservermodell.

Det betyder inte att de är dåliga applikationer. Det betyder att de är byggda för en annan driftmodell än cloud native. En traditionell Java EE-applikation kan förutsätta långlivade servrar, serverkonfigurerade datasources, JNDI-resurser, JMS-objekt, lokala loggfiler, sessionsstate och manuella releaseflöden.

Det här kapitlet visar hur Java EE kan vara startpunkten för en stegvis modernisering. Målet är inte att skriva om allt direkt, utan att avgöra vilken moderniseringsnivå som är rimlig för varje applikation.


## Lärandemål

Efter kapitlet ska läsaren kunna:

- skilja mellan containeriserad Java EE och cloud native Java
- identifiera serverbundna beroenden i Java EE-applikationer
- bedöma om en applikation ska containeriseras, anpassas eller byggas om
- förstå krav på konfiguration, health checks, logging och shutdown
- resonera om sessioner, state, runtime footprint och skalning
- formulera en praktisk plan för stegvis Java-modernisering


## Java EE som startpunkt

Java EE gav länge en stabil modell för transaktioner, datasources, säkerhet, JMS, paketering och drift. Många affärskritiska system har fungerat väl tack vare den modellen. Därför bör Java EE inte beskrivas som ett problem i sig.

Det som behöver analyseras är vilka antaganden applikationen har gjort om sin runtime:

- datasources finns i applikationsservern
- JMS- och MQ-resurser är serverkonfigurerade
- certifikat, truststores och bibliotek finns lokalt på servern
- loggar skrivs till serverns filsystem
- sessioner hålls i serverminne
- driftteam ändrar miljöspecifik konfiguration
- applikationen deployas som WAR eller EAR
- servern är långlivad och byts sällan

I OpenShift behöver dessa antaganden göras explicita. Annars flyttas bara den gamla servermodellen in i en container.


## Tre moderniseringsnivåer

| Nivå | Beskrivning | När den passar |
|---|---|---|
| Containeriserad befintlig runtime | Traditionell applikationsserver paketeras i image | Snabb övergång, komplex legacy, låg förändringsaptit |
| Anpassad Java-applikation | Konfiguration, health, logging, shutdown och state moderniseras | Appen ska leva vidare i OpenShift |
| Cloud native Java | Applikationen designas för plattformsdrift, observability och skalning | Nya tjänster eller avgränsade strangler-delar |

Det är inte alltid bäst att hoppa direkt till nivå tre. En affärskritisk applikation kan först behöva containeriseras, därefter anpassas, och först senare brytas upp eller byggas om.


## Containeriserad Java EE

Att köra en befintlig Java EE-runtime i container kan vara ett legitimt första steg. Det kan skapa reproducerbar paketering, tydligare runtime-versioner och bättre kontroll över image-livscykeln.

Men containerisering är inte samma sak som modernisering. Om applikationen fortfarande kräver manuell serverkonfiguration, lokal state, osynliga beroenden och gamla releaseprocesser har organisationen främst bytt paketeringsformat.

Containeriserad Java EE passar när applikationen är viktig men svår att ändra, när omskrivning vore för riskfylld, eller när målet är att först synliggöra beroenden. Det bör däremot finnas en beslutad plan för vilka problem som ska åtgärdas senare.


## Anpassad Java-applikation

Mellanläge två är ofta mest praktiskt. Applikationen skrivs inte om helt, men den görs mer plattformsduglig.

Typiska anpassningar:

- externaliserad konfiguration
- secrets via plattformens mekanismer
- loggar till standard output
- readiness och liveness
- graceful shutdown
- non-root-kompatibilitet
- tydligare sessionstrategi
- resource requests och limits
- dokumenterade beroenden till Oracle, MQ och externa API:er
- grundläggande metrics och korrelations-id

Denna nivå ger ofta stor effekt utan att all affärslogik behöver rivas upp.


## Cloud native Java

Cloud native Java innebär att applikationen designas för dynamisk plattformsdrift från början. Den är inte bara körbar i container, utan anpassad för att vara observerbar, konfigurerbar, skalbar och ersättningsbar.

Kännetecken:

- liten och tydlig runtime
- externaliserad konfiguration
- tydlig health-modell
- stateless där det är möjligt
- idempotenta integrationsflöden
- metrics, loggar och tracing
- kontrollerad resursprofil
- deklarativ deployment
- automatiserad build och test
- tydligt ägarskap för driftbeteende

För nya tjänster bör detta vara normalmålet. För äldre Java EE-system är det ofta ett stegvis mål.


## Serverbunden konfiguration

En av de största utmaningarna är konfiguration som historiskt legat i applikationsservern. Exempel är datasources, JMS connection factories, MQ-destinationer, security realms, JAAS-konfiguration, certifikat, system properties, delade bibliotek, thread pools och transaction manager-inställningar.

I en OpenShift-modell behöver teamet bestämma vad som hör hemma i image, vad som hör hemma i deployment, vad som ska vara ConfigMap, vad som ska vara Secret och vad som ska ägas av plattformen.

Det viktiga är att miljöskillnader inte ska vara dolda serverändringar.


## Externaliserad konfiguration

Externaliserad konfiguration innebär att samma image kan användas i flera miljöer. Miljöspecifika värden tillförs vid deployment.

Konfiguration kan omfatta databas-URL, MQ-endpoints, externa API:er, loggnivåer, feature flags, timeouts, retry-inställningar, certifikatvägar och cacheinställningar. Secrets ska hanteras separat från vanlig konfiguration. Ett lösenordsbyte ska inte kräva ny image.

Detta är en central skillnad mot många traditionella Java EE-miljöer där servern bar mycket av miljöns identitet.


## Health checks

OpenShift behöver veta om applikationen lever och om den är redo att ta emot trafik.

| Typ | Syfte | Exempel |
|---|---|---|
| Liveness | Är processen vid liv? | Starta om om applikationen hängt sig |
| Readiness | Kan applikationen ta emot trafik? | Ta bort podden från trafik tills den är redo |
| Startup | Behöver appen längre starttid? | Skydda långsamt startande legacy-runtime |

Readiness ska vara meningsfull men inte för aggressiv. Om den beror på varje extern komponent kan korta störningar skapa onödig instabilitet. Om den bara svarar OK säger den för lite.


## Logging och observability

Traditionella Java EE-applikationer skriver ofta till loggfiler på servern. I OpenShift bör applikationen skriva till standard output så att plattformen kan samla in loggarna.

Bra loggning bör innehålla tjänstnamn, miljö, version, korrelations-id, loggnivå, teknisk kontext och relevanta fel. Den ska inte innehålla secrets eller onödiga personuppgifter.

För moderniserade Java-applikationer bör loggar kompletteras med metrics och tracing där det behövs. Observability är inte ett separat tillägg, utan en del av produktionsklarheten.


## Graceful shutdown

När OpenShift stoppar eller flyttar en podd behöver applikationen kunna avsluta kontrollerat. Detta är särskilt viktigt för Java EE-applikationer som hanterar transaktioner, MQ-konsumtion eller långkörande jobb.

Frågor:

- slutar applikationen ta emot ny trafik?
- får pågående request avslutas?
- stoppas MQ-konsumenter kontrollerat?
- bekräftas meddelanden först efter lyckad behandling?
- stängs databasanslutningar?
- avbryts batchjobb säkert?
- finns risk för halvbearbetade transaktioner?
- hur lång termination grace period behövs?

Graceful shutdown avslöjar ofta om applikationen verkligen passar i en dynamisk plattform.


## Sessioner och state

Många Java EE-applikationer använder HTTP-sessioner. I traditionell serverdrift kan det ha fungerat bra. I OpenShift skapar det frågor om skalning, rolling updates och podd-förlust.

| Strategi | Fördel | Risk |
|---|---|---|
| Minska sessionstillstånd | Bättre skalbarhet | Kräver kodändring |
| Externalisera sessioner | Flera instanser kan dela state | Ny stateful komponent |
| Sticky sessions | Snabb övergång | Bevarar instansberoende |
| Tokenbaserad modell | Mer cloud native | Kräver ändrad säkerhetsmodell |

Sticky sessions kan vara ett övergångsmönster, men bör inte bli omedvetet slutläge.


## Runtime footprint

Runtime footprint beskriver applikationens resursprofil: minne, CPU, image-storlek, starttid och beroenden. I traditionell drift kunde en tung applikationsserver vara acceptabel om servern var långlivad. I OpenShift blir footprint mer synligt eftersom många workloads delar plattform.

Frågor:

- hur mycket minne behövs vid start?
- hur mycket minne behövs under last?
- hur lång är kallstart?
- hur stor är imagen?
- hur många instanser behövs?
- hur påverkas rolling updates?
- är runtime patchbar?
- är resursprofilen rimlig för tjänstens värde?

En stor runtime är inte alltid fel, men den måste vara ett medvetet val.


## Beroenden till Oracle och IBM MQ

Java EE-applikationer är ofta nära kopplade till Oracle och IBM MQ. Modernisering behöver därför hantera dessa beroenden explicit.

För Oracle bör teamet kartlägga JNDI-datasources, direkt schemaåtkomst, lagrade procedurer, transaktionsantaganden, anslutningspooler och schemaförändringar.

För IBM MQ bör teamet kartlägga JMS-resurser, connection factories, destinationer, transaktionskoppling mellan MQ och databas, retry, dead-letter, idempotens och shutdown av konsumenter.

Dessa beroenden är ofta viktigare än valet av Java-runtime.


## Beslutsguide: moderniseringsnivå

| Scenario | Rekommenderad nivå |
|---|---|
| Affärskritisk legacy-app med många serverberoenden | Containerisera försiktigt eller kapsla in först |
| App med externaliserbar konfiguration och få runtime-beroenden | Anpassad Java-applikation |
| Ny avgränsad tjänst | Cloud native Java från start |
| App med starkt sessionsberoende | Anpassa sessionstrategi före skalning |
| App nära avveckling | Minimal modernisering eller lämna kvar |
| Funktion som bryts ut med strangler pattern | Cloud native Java om scope är tydligt |
| App med okänd serverkonfiguration | Kartlägg före containerisering |
| App med MQ-konsumenter | Verifiera shutdown, retry och idempotens |


## Beslutsguide: OpenShift-redo Java-applikation

| Krav | Fråga |
|---|---|
| Image | Byggs den reproducerbart från godkänd bas? |
| Runtime | Är runtime-version och patchmodell tydlig? |
| Konfiguration | Kan samma image användas i flera miljöer? |
| Secrets | Hanteras credentials utanför image och kod? |
| Health | Finns readiness och liveness? |
| Logging | Går loggar till standard output? |
| State | Är sessioner och lokala filer hanterade? |
| Shutdown | Kan processen stoppas kontrollerat? |
| Resurser | Finns requests och limits? |
| Beroenden | Är Oracle, MQ och externa API:er dokumenterade? |
| Observability | Finns metrics och korrelations-id där det behövs? |


## Tradeoffs

| Val | Fördel | Risk |
|---|---|---|
| Behålla Java EE-runtime | Låg kodrisk | Tung runtime och gamla mönster |
| Byta runtime | Bättre footprint | Mer test och migreringsrisk |
| Minimal containerisering | Snabb start | Gamla problem flyttas |
| Anpassa stegvis | Bra balans | Kräver disciplin och roadmap |
| Full omskrivning | Renare målbild | Hög risk och lång ledtid |
| Sticky sessions | Snabb kompatibilitet | Begränsad skalbarhet |
| Stateless design | Bättre plattformspassform | Kan kräva större förändring |


## Anti-patterns

### Containeriserad applikationsserver som slutmål utan beslut

Det kan vara rimligt att containerisera en befintlig runtime, men om det görs utan målbild riskerar organisationen att cementera gamla driftmönster.

### Health endpoint som alltid svarar OK

En health endpoint som inte speglar applikationens verkliga tillstånd ger falsk trygghet.

### Lokal filhantering som dold state

Om applikationen skriver viktig data lokalt i containern uppstår risk vid omstart och flytt.

### Secrets i image eller kod

Credentials i image eller kod skapar säkerhetsrisk och gör rotation svår.

### Omskrivning utan affärsvärde

Att skriva om en stabil applikation utan tydlig nytta kan bli dyrt och riskfyllt.


## Vanliga fallgropar

- Att underskatta serverkonfiguration.
- Att glömma JNDI, datasources och JMS-resurser.
- Att inte testa flera repliker.
- Att inte testa rolling update.
- Att inte testa shutdown av MQ-konsumenter.
- Att behålla sessionsstate utan beslut.
- Att sakna resource limits.
- Att loggar fortfarande skrivs till fil.
- Att runtime patchning saknar ägare.
- Att inte dokumentera beroenden till Oracle och IBM MQ.


## Arkitektens checklista

Innan en Java EE-applikation flyttas eller moderniseras bör arkitekten kunna svara på:

- Vilken runtime används?
- Vilka serverberoenden finns?
- Vilka datasources används?
- Vilka JMS- eller MQ-resurser används?
- Finns delade bibliotek?
- Hur hanteras certifikat?
- Hur externaliseras konfiguration?
- Hur hanteras secrets?
- Hur loggar applikationen?
- Finns health checks?
- Kan flera instanser köras?
- Hur hanteras sessioner?
- Hur stoppas applikationen?
- Hur hanteras Oracle-transaktioner?
- Hur hanteras MQ-konsumtion?
- Vilken moderniseringsnivå är rimlig?


## Koppling till moderniseringscaset

Nordisk Handel AB kartlägger sina Java EE-applikationer och delar in dem i tre grupper:

1. Stödapplikationer som kan containeriseras och anpassas relativt snabbt.
2. Kärnapplikationer som behöver inkapslas och moderniseras stegvis.
3. Nya eller utbrutna funktioner som ska byggas som cloud native Java från start.

För den första stödapplikationen upptäcker teamet att datasources, MQ-resurser och loggning är serverbundna. De gör konfigurationen explicit, flyttar secrets till plattformens modell, inför readiness och liveness och ändrar loggning till standard output.

Organisationen beslutar att ingen Java-applikation får kallas OpenShift-redo utan dokumenterad konfiguration, health checks, logging, shutdown och beroendemodell.


## Snabb sammanfattning

- Java EE är startpunkt, inte automatiskt problem.
- Containeriserad Java EE är inte samma sak som cloud native Java.
- Modernisering kan ske i nivåer: containerisering, anpassning eller cloud native-omdesign.
- Serverbunden konfiguration behöver göras explicit.
- Health checks, logging, graceful shutdown och externaliserad konfiguration är minimikrav.
- Sessioner, state och runtime footprint måste analyseras.
- Oracle- och MQ-beroenden behöver hanteras innan produktion i OpenShift.

## Nästa steg

Nästa kapitel behandlar integration, transaktioner och konsistens. Där går vi djupare in i vad som händer när monolitiska transaktionsgränser ersätts av distribuerade flöden, eventual consistency och kompensation.
