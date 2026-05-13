# Kapitel 6: Messaging och eventdrivna mönster

## Varför detta kapitel finns

I många Java EE-baserade enterprise-miljöer har IBM MQ varit lika central som Oracle. Den har hanterat robusta integrationer, asynkrona flöden, koppling mot legacy och kommunikation mellan system som inte kan eller bör vara tillgängliga samtidigt.

När organisationen går mot OpenShift och cloud native uppstår därför en viktig fråga: ska IBM MQ behållas, kapslas in, kompletteras eller ersättas?

Svaret är sällan enkelt. IBM MQ kan fortfarande vara rätt val för affärskritiska legacy-flöden. Samtidigt passar inte traditionell köhantering för alla moderna integrationsbehov. Vissa flöden är kommandon, andra är domänhändelser. Vissa behöver exakt en mottagare, andra behöver flera konsumenter. Vissa behöver återspel och historik, andra behöver bara robust arbetskö.

Det här kapitlet ger en beslutsmodell för messaging och eventdrivna mönster i moderniseringsresan.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- skilja mellan meddelandekö, kommando, event, publish/subscribe och event streaming
- analysera när IBM MQ bör behållas, kapslas in, kompletteras eller ersättas
- förstå varför eventdriven arkitektur kräver mer än asynkron transport
- identifiera risker med integrationsmonolit
- resonera om idempotens, retry, dead-letter och ordering
- formulera en beslutsmodell för messaging i en OpenShift-orienterad målarkitektur

## IBM MQ som integrationsgrund

IBM MQ har ofta införts för att lösa verkliga problem:

- system behöver kommunicera asynkront
- mottagare kan vara otillgängliga under kortare perioder
- meddelanden behöver levereras tillförlitligt
- legacy-system kräver beprövade integrationsmönster
- driftorganisationen behöver stabil och supportad köhantering
- transaktionsnära flöden behöver kontrolleras

I en sådan roll kan IBM MQ vara mycket värdefullt. Problemet uppstår när samma teknik börjar användas för alla integrationsbehov, oavsett mönster.

Med tiden kan en central MQ-plattform bli:

- integrationsnav för nästan alla system
- dold processmotor
- plats för otydliga kontrakt
- ersättning för API:er
- transport för både kommandon och händelser utan tydlig skillnad
- gräns mellan gammalt och nytt
- teknisk flaskhals för förändring

Då är utmaningen inte bara produkten. Utmaningen är integrationsarkitekturen.

## Kö är inte samma sak som eventdriven arkitektur

All asynkron kommunikation är inte eventdriven arkitektur.

En meddelandekö används ofta för att skicka ett arbete eller kommando till en mottagare. Ett event beskriver något som redan har hänt. Skillnaden är viktig eftersom den påverkar ansvar, kontrakt och konsistens.

| Mönster | Beskrivning | Typisk fråga |
|---|---|---|
| Message queue | Meddelande läggs på kö och konsumeras av mottagare | Vem ska utföra detta arbete? |
| Command message | Meddelande som ber mottagaren göra något | Utför denna operation |
| Domain event | Händelse som beskriver något som hänt | Vad har hänt i domänen? |
| Publish/subscribe | Flera prenumeranter kan ta emot publicerat meddelande | Vilka behöver veta detta? |
| Event streaming | Händelser lagras i logg och kan läsas eller återspelas | Vem vill reagera nu eller senare? |
| Request/reply | Avsändare skickar begäran och väntar på svar | Hur får jag ett svar asynkront? |

Om organisationen inte skiljer mellan dessa mönster riskerar den att bygga otydliga flöden där ingen vet om meddelandet är en order, en händelse eller en teknisk signal.

## Kommandon och händelser

Ett kommando uttrycker en önskan: gör något. En händelse uttrycker ett faktum: något har hänt.

Exempel på kommando:

- reservera lager
- skapa faktura
- skicka orderbekräftelse
- annullera betalning

Exempel på händelse:

- order skapad
- betalning reserverad
- lagerreservation misslyckades
- kund uppdaterad

Kommandon har ofta en tydlig mottagare. Händelser kan ha flera konsumenter. Producenten av en händelse bör normalt inte behöva veta exakt vilka som lyssnar, men händelsens semantik och kontrakt måste ändå vara tydliga.

## IBM MQ i målarkitekturen

IBM MQ behöver inte tas bort för att organisationen ska bli mer cloud native. Men dess roll bör bli tydligare.

### Behåll IBM MQ när

- flödet är affärskritiskt och stabilt
- legacy-system kräver MQ
- migreringsrisk är hög
- driftkompetens och support är etablerad
- leveranskraven är höga
- affärsnyttan med byte är låg

### Kapsla in IBM MQ när

- nya tjänster inte bör känna till MQ-detaljer
- flödet behöver exponeras via adapter eller API
- meddelandeformat är svårt att ändra
- legacy och OpenShift-tjänster ska samexistera
- organisationen vill minska direktkoppling

### Komplettera IBM MQ när

- flera konsumenter behöver samma händelser
- återspel och historik är viktigt
- enklare arbetsköer inte kräver tung enterprise-messaging
- nya domäner behöver eventdrivna mönster
- MQ används av gammal vana för nya behov

### Ersätt IBM MQ när

- användningsfallet är avgränsat och migrerbart
- flödet inte kräver MQ:s egenskaper
- enklare broker eller event streaming passar bättre
- ny driftmodell finns
- avveckling ger tydligt värde

## Event streaming

Event streaming bygger på att händelser skrivs till en logg där flera konsumenter kan läsa i egen takt. Kafka-liknande plattformar används ofta för detta mönster.

Event streaming passar när:

- flera konsumenter behöver samma händelser
- nya konsumenter ska kunna tillkomma utan producentändring
- historik och återspel är värdefullt
- dataflöden ska användas för analys, sök eller integration
- domänhändelser ska bli explicit del av arkitekturen
- asynkrona flöden behöver skalas över tid

Event streaming passar sämre när:

- det bara finns en mottagare
- meddelanden är kortlivade arbetsuppdrag
- organisationen saknar schema- och topic-governance
- konsumenter inte kan hantera dubbletter
- ordering och kompensation inte är förstådda
- tekniken införs bara för att ersätta MQ produktmässigt

Event streaming är ett arkitekturmönster, inte bara en produktkategori.

## Enklare brokers

För vissa behov är event streaming för tungt. En enklare broker kan passa bättre för arbetsköer, intern asynkron bearbetning eller request/reply-liknande flöden.

En enklare broker passar ofta när:

- behovet är intern arbetskö
- meddelanden är kortlivade
- återspel inte behövs
- bara en eller få konsumenter finns
- utvecklarupplevelse och enkel drift är viktig
- volymen är hanterbar

Men även en enkel broker kräver driftmodell:

- ködjup måste övervakas
- felmeddelanden behöver hanteras
- retry måste styras
- dead-letter queues behövs
- ägare för köer och meddelandekontrakt måste vara tydliga

En “enklare” teknik är inte samma sak som ingen governance.

## Integrationsmonolit

En integrationsmonolit uppstår när system är distribuerade tekniskt men hårt kopplade semantiskt. Det kan hända med IBM MQ, Kafka, RabbitMQ, API:er eller filintegration.

Symptom:

- ingen vågar ändra meddelandeformat
- många okända konsumenter finns
- samma meddelande betyder olika saker för olika system
- orderingberoenden är dolda
- retry skapar affärsfel
- topics eller köer används som databas
- integrationsteam blir flaskhals
- flöden saknar tydliga ägare
- avveckling är nästan omöjlig

Integrationsmonolit är särskilt farlig eftersom den kan se modern ut. Tekniken är distribuerad, men förändringsförmågan är fortfarande låg.

## Kontrakt och schema

Meddelanden och events är kontrakt. De behöver versioneras och ägas.

Ett moget kontrakt bör beskriva:

- namn
- ägare
- syfte
- om det är kommando eller event
- schema
- versionsstrategi
- obligatoriska fält
- semantisk betydelse
- felhantering
- retention
- kompatibilitetsregler
- avvecklingsprocess

För events är semantiken särskilt viktig. Ett event som heter `OrderUpdated` är ofta för otydligt. Vad ändrades? Vem äger händelsen? Ska alla konsumenter reagera? Är det en affärshändelse eller teknisk synkronisering?

## Idempotens

Idempotens betyder att samma meddelande kan behandlas mer än en gång utan att skapa felaktigt affärsresultat.

Detta är centralt eftersom asynkrona system ofta kan leverera dubbletter. En konsument kan bearbeta ett meddelande men krascha innan bekräftelse. Avsändaren kan göra retry. Nätverksfel kan skapa osäkerhet.

Exempel:

- Att reservera lager två gånger för samma order är fel.
- Att sätta orderstatus till “bekräftad” två gånger kan vara ofarligt.
- Att spara behandlade message-id:n kan göra konsumenten idempotent.

Idempotens bör vara ett designkrav, inte en efterhandsfix.

## Retry och dead-letter

Retry är nödvändigt när fel kan vara tillfälliga. Men retry utan gräns kan förvärra incidenter.

En bra retry-strategi bör definiera:

- vilka fel som ska retried
- hur många försök som görs
- väntetid mellan försök
- när meddelande flyttas till dead-letter
- vem som hanterar dead-letter
- hur man återkör meddelanden
- hur dubbletter undviks
- vilka larm som behövs

Dead-letter queue är inte en soptunna. Det är en operativ kö som måste ha ägare och process.

## Ordering

Vissa flöden kräver att meddelanden hanteras i rätt ordning. Andra gör inte det. Ordering är dyrt om det görs för brett.

Frågor att ställa:

- måste alla meddelanden vara globalt ordnade?
- räcker ordering per order, kund eller konto?
- vad händer om ett senare meddelande kommer först?
- finns versionsnummer?
- kan konsumenten ignorera gamla händelser?
- påverkar ordering skalbarheten?

Ofta räcker ordering per affärsobjekt. Global ordering bör undvikas om den inte är tydligt motiverad.

## Messaging och OpenShift

Messagingplattformar kan köras i eller utanför OpenShift. Beslutet ska baseras på driftmodell, inte bara teknisk möjlighet.

| Alternativ | Passar när | Risk |
|---|---|---|
| IBM MQ utanför OpenShift | Legacy och kritiska flöden är etablerade | Nya tjänster kan bli för hårt kopplade |
| Broker i OpenShift | Plattformen har mönster för drift och observability | Stateful komplexitet underskattas |
| Event streaming i OpenShift | Behov av händelselogg och återspel finns | Kräver governance och kapacitet |
| Managed messaging | Driftbörda ska minskas | Compliance, nätverk och kostnad behöver analyseras |

Oavsett var tekniken körs behöver flödena kontrakt, observability och ägarskap.

## Beslutsguide: välj messagingmönster

| Scenario | Rekommenderat mönster |
|---|---|
| Ett system ska be ett annat göra något | Command message |
| Flera system behöver veta att något hänt | Domain event |
| Händelser behöver återspelas | Event streaming |
| Ett internt arbete ska köas | Message queue eller broker |
| Legacy kräver MQ | Behåll eller kapsla in IBM MQ |
| Synkront svar behövs | API eller request/reply med tydlig konsekvens |
| Sökindex ska uppdateras | Event eller CDC-baserat flöde |
| Batchliknande filöverföring | Filflöde eller objektlagring kan vara bättre |

## Beslutsguide: IBM MQ, broker eller event streaming

| Behov | IBM MQ | Enklare broker | Event streaming |
|---|---|---|---|
| Kritisk legacy-integration | Starkt alternativ | Svagare | Via adapter |
| Intern arbetskö | Möjligt men tungt | Starkt alternativ | Ofta överdrivet |
| Flera konsumenter | Möjligt | Möjligt | Starkt alternativ |
| Återspel | Begränsat | Begränsat | Starkt alternativ |
| Låg komplexitet | Beror på befintlig drift | Starkt | Svagare |
| Domänhändelser | Möjligt | Möjligt | Starkt om governance finns |
| Transaktionsnära flöde | Starkt | Beror på krav | Kräver noggrann design |
| Teamautonomi | Kan begränsas av central plattform | God | God men kräver styrning |

## Tradeoffs

| Val | Fördel | Risk |
|---|---|---|
| Behålla IBM MQ | Stabilitet och låg migreringsrisk | Legacy-koppling kan bestå |
| Kapsla in MQ | Nya tjänster slipper MQ-detaljer | Adapterlager måste förvaltas |
| Införa event streaming | Flera konsumenter och återspel | Högre operativ och semantisk komplexitet |
| Använda enklare broker | Lägre komplexitet för arbetsköer | Kan bli spretig ny standard |
| Standardisera på en teknik | Enklare drift | Fel passform för vissa mönster |
| Välja mönster per behov | Bättre arkitektur | Kräver beslutsmodell |
| Tillåta okända konsumenter | Flexibilitet | Svårare förändring och avveckling |

## Anti-patterns

### Byta IBM MQ mot Kafka utan att ändra design

Om gamla kommandoflöden flyttas till Kafka utan eventmodell, kontrakt och idempotens har organisationen bara bytt transport.

### Allt blir asynkront

Asynkronitet kan öka robusthet, men gör också felsökning och användarupplevelse mer komplex. Vissa flöden passar bättre som synkrona API:er.

### Event som databas

Events kan användas för att bygga läsmodeller, men eventplattformen ska inte bli otydlig primär sanning utan dataägarskap.

### Dead-letter utan ägare

En dead-letter queue utan process och ansvar är bara uppskjuten incidenthantering.

### Okända konsumenter överallt

Lös koppling betyder inte att producenten saknar ansvar för kontrakt. Det betyder att kontraktet är tydligt nog för kontrollerad förändring.

## Vanliga fallgropar

- Att välja produkt före mönster.
- Att inte skilja mellan kommando och event.
- Att sakna idempotenskrav.
- Att sakna retry-strategi.
- Att inte mäta ködjup eller consumer lag.
- Att låta dead-letter växa utan process.
- Att införa event streaming utan schemahantering.
- Att inte definiera ägare för topics och köer.
- Att underskatta orderingproblem.
- Att behålla gamla meddelandeformat utan avvecklingsplan.

## Arkitektens checklista

Innan ett messagingbeslut fattas bör arkitekten kunna svara på:

- Är detta ett kommando, event, request/reply eller arbetskö?
- Vem äger meddelandet?
- Vem äger konsumenten?
- Finns en eller flera konsumenter?
- Behövs återspel?
- Behövs ordering?
- Vilka leveransgarantier krävs?
- Är konsumenten idempotent?
- Hur fungerar retry?
- När hamnar meddelandet i dead-letter?
- Vem hanterar dead-letter?
- Hur versioneras kontraktet?
- Hur observeras flödet?
- Ska tekniken köras i OpenShift eller utanför?
- Behövs ADR?

## Koppling till moderniseringscaset

Nordisk Handel AB kartlägger sina IBM MQ-flöden. Analysen visar att MQ används för flera olika saker: kritiska orderintegrationer, batchliknande filflöden, interna arbetsköer och vissa händelser som flera system vill reagera på.

Organisationen beslutar att inte ersätta IBM MQ i första vågen. I stället införs en klassificering:

- kritiska legacy-flöden behålls på IBM MQ
- nya tjänster får inte direktkoppla sig till MQ utan arkitekturbedömning
- interna arbetsköer kan använda enklare broker om driftmodellen är tydlig
- domänhändelser kring orderstatus blir kandidater för event streaming
- alla nya meddelanden ska ha ägare, kontrakt och versioneringsstrategi

Det viktigaste beslutet är att moderniseringen inte definieras som “bort från IBM MQ”, utan som “rätt integrationsmönster för rätt behov”.

## Snabb sammanfattning

- IBM MQ kan vara fortsatt rätt för kritiska legacy-flöden.
- Kö, kommando, event, pub/sub och event streaming löser olika problem.
- Eventdriven arkitektur kräver tydliga kontrakt, idempotens och governance.
- Event streaming passar när flera konsumenter, historik eller återspel ger värde.
- Enklare brokers kan passa bättre för interna arbetsköer.
- Integrationsmonolit kan uppstå oavsett teknik.
- Messagingmodernisering ska börja med mönsterklassificering, inte produktbyte.

## Nästa steg

Nästa kapitel behandlar Elasticsearch som sekundärt index. Där går vi igenom hur sök, filtrering och analysnära läsmodeller kan avlasta primärdatabaser utan att Elasticsearch blir ny primär sanning.
