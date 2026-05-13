# Kapitel 5: Dataarkitektur och modernisering från Oracle

## Varför detta kapitel finns

I många Java EE-baserade enterprise-miljöer är Oracle mer än en databas. Den är ofta transaktionsmotor, integrationsyta, rapportkälla, batchplattform, historiskt arkiv och ibland även platsen där affärslogik har hamnat över tid. Därför är databasmodernisering sällan ett enkelt produktbyte.

När organisationer börjar använda containers och OpenShift uppstår ofta frågan om Oracle ska flyttas, ersättas eller kompletteras. Frågan är rimlig, men bör inte besvaras för snabbt. En affärskritisk Oracle-miljö kan vara rätt att behålla under lång tid, samtidigt som nya tjänster kan behöva andra datalösningar.

Det här kapitlet fokuserar på dataarkitektur: hur man analyserar Oracle-miljön, när Oracle bör behållas, när PostgreSQL eller andra alternativ passar, och hur man undviker att skapa en ny databasmonolit i modernare form.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- skilja mellan databasprodukt, dataroll och dataägarskap
- analysera Oracle som primär transaktionskälla, integrationsyta och rapportkälla
- bedöma när Oracle bör behållas, kapslas in, kompletteras eller ersättas
- förstå när PostgreSQL passar för nya avgränsade domäner
- identifiera risker med delad databas och direkt schemaåtkomst
- formulera en beslutsmodell för databaser i OpenShift-orienterad arkitektur

## Oracle som arkitekturcentrum

Oracle har ofta fått en central roll av goda skäl. Databasen har varit stabil, kraftfull och välkänd. Organisationen har byggt drift, backup, behörighet, rapportering och kompetens kring den.

Med tiden kan Oracle-miljön dock få flera roller samtidigt:

- primär transaktionsdatabas
- gemensam datamodell för flera system
- integrationsyta mellan applikationer
- plats för lagrad affärslogik
- rapporteringskälla
- batchplattform
- arkiv
- sökunderlag
- källa för exportfiler

När en databas får så många roller blir den inte bara en teknisk komponent. Den blir ett arkitekturcentrum. Det gör modernisering svår eftersom varje förändring riskerar att påverka många system.

## Databasprodukt är inte samma sak som dataroll

Ett vanligt misstag är att börja med frågan: “ska vi byta Oracle mot PostgreSQL?” Den bättre första frågan är: “vilken dataroll pratar vi om?”

| Dataroll | Syfte | Möjliga tekniker |
|---|---|---|
| Primär transaktionsdatabas | Äger affärskritisk sanning | Oracle, PostgreSQL |
| Sekundär läsmodell | Optimerar läsning utan att äga sanning | PostgreSQL, replikerad modell |
| Sökindex | Sökning, filtrering och relevans | Elasticsearch |
| Cache | Snabb åtkomst till återskapbar data | Cachelösning |
| Analytiskt lager | Rapportering och historik | Data warehouse, lakehouse |
| Objektlagring | Dokument, filer och binära objekt | S3-kompatibel lagring, Ceph |
| Eventlogg | Händelser och återspel | Event streaming |

Modernisering blir lättare när varje databehov placeras i rätt roll. Oracle kan vara rätt för en roll och fel för en annan.

## Primär sanning

Primär sanning är den datakälla som äger korrekt affärstillstånd. Om flera system har kopior behöver organisationen veta vilken källa som gäller.

Exempel:

- Oracle äger orderns transaktionella sanning.
- Elasticsearch äger inte ordern, utan ett sökindex över orderdata.
- Ett rapportlager äger inte operativ status, utan en analytisk vy.
- En cache äger inte data, utan kan återskapas.

Om detta inte är tydligt uppstår dubbla sanningar. Då vet ingen säkert vilken källa som ska korrigeras vid fel.

## Delad databas som kopplingsproblem

I traditionella Java EE-miljöer delar flera applikationer ofta databas eller schema. Det kan ha varit effektivt initialt, men skapar stark koppling.

Problem med delad databas:

- schemaändringar påverkar flera system
- dataägarskap blir otydligt
- intern tabellstruktur blir externt kontrakt
- säkerhetsmodellen blir grov
- testning blir svår
- nya tjänster tvingas förstå gammal datamodell
- transaktionsgränser blir otydliga
- modernisering blockeras av okända beroenden

Delad databas bör därför behandlas som ett moderniseringsproblem även om Oracle som databasprodukt behålls.

## Direkt schemaåtkomst

Direkt schemaåtkomst innebär att flera applikationer läser eller skriver direkt i samma schema. Det är ofta ett av de största hindren för cloud native-modernisering.

Direkt schemaåtkomst bör inte alltid stoppas över en natt. Det kan vara affärskritiskt och svårt att ändra. Men nya tjänster bör normalt inte få direkt åtkomst till legacy-schema som standard.

Alternativ:

- API framför legacy-funktion
- läsmodell för rapportering eller sök
- eventdriven replikering
- kontrollerad adapter
- datakontrakt
- strangler pattern runt viss funktion

Målet är att stegvis flytta från delad intern datamodell till tydliga kontrakt.

## När Oracle bör behållas

Oracle bör ofta behållas när den hanterar affärskritisk kärndata och migreringsrisken är hög.

Behåll Oracle när:

- datamodellen är stor och central
- transaktionskraven är höga
- befintlig driftmodell är mogen
- backup och restore är etablerade
- regulatoriska krav är kopplade till nuvarande lösning
- många system är beroende av data
- alternativ inte ger tydligt affärsvärde
- modernisering kan ske genom inkapsling först

Att behålla Oracle är inte ett misslyckande. Det kan vara det mest ansvarsfulla beslutet i en stegvis moderniseringsresa.

## När Oracle bör kapslas in

Inkapsling är ofta första steget före ersättning. Det innebär att nya och förändrade applikationer inte får direkt beroende till Oracle-tabeller, utan går via ett kontrollerat gränssnitt.

Kapsla in Oracle när:

- flera system delar schema
- nya tjänster behöver data från legacy
- databasen används som integrationsyta
- organisationen vill minska koppling utan datamigrering
- affärslogik i databasen behöver exponeras kontrollerat
- rapportering belastar transaktionssystemet

Inkapsling kan ske via API, service, adapter, eventflöde eller sekundär läsmodell.

## När Oracle bör kompletteras

Oracle behöver inte ersättas för att nya databehov ska lösas bättre. Ofta är komplettering rätt väg.

Komplettera Oracle när:

- sökbehov passar bättre i Elasticsearch
- dokument och filer passar bättre i objektlagring
- nya domäner har enklare relationsbehov
- rapportering bör avlasta transaktionssystemet
- eventflöden behövs för flera konsumenter
- cache kan minska belastning på kärnsystem

Komplettering kräver dock datagovernance. Annars uppstår spretiga datalager utan tydlig primär sanning.

## När Oracle kan ersättas

Ersättning är rimlig när användningsfallet är avgränsat och värdet är tydligt.

Ersätt Oracle när:

- domänen är tydligt avgränsad
- datamängden är hanterbar
- beroenden är kartlagda
- migrering kan testas
- rollback eller parallellkörning är möjlig
- PostgreSQL eller annan lösning uppfyller kraven
- organisationen har driftmodell för alternativet
- affärsnyttan motiverar risken

Ersättning bör normalt ske domänvis eller funktionsvis, inte som ett stort produktbyte.

## PostgreSQL som alternativ

PostgreSQL är ofta ett bra alternativ för nya avgränsade tjänster med relationsbehov. Det kan ge lägre tröskel, tydligare teamägarskap och bättre passform för cloud native-arbetssätt.

PostgreSQL passar ofta när:

- tjänsten har egen datamodell
- teamet äger schema och migrationer
- transaktionskraven är normala
- koppling till Oracle inte behövs
- driftmodell finns
- backup och restore är löst
- lösningen inte kräver Oracle-specifika funktioner

PostgreSQL passar sämre när:

- data är starkt beroende av Oracle-schema
- affärslogik ligger i Oracle-procedurer
- teamet saknar databasdriftmodell
- hög tillgänglighet och restore inte är etablerade
- valet görs bara för att “komma bort från Oracle”

PostgreSQL ska inte bli en ny central monolit. Det ska användas med tydligt dataägarskap.

## Databas i OpenShift eller utanför?

En återkommande fråga är om databaser ska köras i OpenShift. Svaret beror på driftmodell, krav och organisationens mognad.

| Alternativ | Passar när | Risk |
|---|---|---|
| Databas utanför OpenShift | Befintlig drift är mogen och kraven är höga | Mer nätverks- och livscykelkoppling |
| Databas i OpenShift | Plattformen har mogen stateful-drift | Backup och restore underskattas |
| Managed databas | Organisationen accepterar tjänstemodell | Kontroll, compliance och nätverk måste analyseras |
| Lokal databas i container | Test, demo eller temporära miljöer | Ofta olämpligt för produktion |

Frågan är inte om det går tekniskt. Frågan är om organisationen kan drifta databasen ansvarsfullt.

## Schemahantering och migrationer

Cloud native-modernisering kräver bättre disciplin kring schemaförändringar. I traditionella miljöer kan databasändringar ha hanterats i separata releasepaket eller av DBA-team. När team får mer ansvar behöver schemaändringar bli spårbara och testbara.

Principer:

- schemaändringar versioneras
- migrationer testas i pipeline
- bakåtkompatibilitet planeras
- rollbackstrategi diskuteras
- dataändringar skiljs från strukturändringar
- ägare för schema är tydlig
- releaseordning mellan applikation och databas hanteras

Schemaförändringar är ofta svårare att rulla tillbaka än applikationskod. Det kräver särskild försiktighet.

## Datagovernance

När organisationen introducerar fler datatekniker ökar behovet av datagovernance.

Datagovernance bör svara på:

- vem äger datamängden?
- vad är primär sanning?
- vilka kopior finns?
- hur uppdateras kopior?
- vilken retention gäller?
- vilka data är känsliga?
- vem får läsa data?
- hur hanteras radering?
- hur testas backup och restore?
- hur dokumenteras dataflöden?

Utan datagovernance riskerar organisationen att byta en central databasmonolit mot många små otydliga dataproblem.

## Beslutsguide: behåll, kapsla in, komplettera eller ersätt

| Situation | Rekommenderad riktning |
|---|---|
| Affärskritisk kärndata i Oracle | Behåll initialt |
| Flera system delar schema | Kapsla in och minska direktåtkomst |
| Nya tjänster behöver enkel relationsdata | Överväg PostgreSQL med tydligt ägarskap |
| Sök och filtrering belastar Oracle | Komplettera med Elasticsearch |
| Dokument lagras som BLOB av gammal vana | Överväg objektlagring |
| Rapportering påverkar OLTP-prestanda | Skapa sekundär läsmodell eller analytiskt lager |
| Oracle används som integrationsyta | Ersätt gradvis med API, event eller adapter |
| Domän är avgränsad och migrerbar | Överväg ersättning efter analys |

## Beslutsguide: databasval för ny tjänst

| Fråga | Om ja | Möjlig riktning |
|---|---|---|
| Äger tjänsten sin egen datamodell? | Ja | PostgreSQL kan vara rimligt |
| Krävs Oracle-specifika funktioner? | Ja | Oracle eller omdesign |
| Behövs stark koppling till legacy-schema? | Ja | Kapsling före ny databas |
| Är data primär sanning? | Ja | Välj databas med tydlig driftmodell |
| Är data sekundär läsmodell? | Ja | Replikerad SQL-modell eller Elasticsearch |
| Är data dokument eller filer? | Ja | Objektlagring |
| Behövs sök och relevans? | Ja | Elasticsearch som sekundärt index |
| Saknas backup/restore? | Ja | Inte produktionsredo |

## Tradeoffs

| Val | Fördel | Risk |
|---|---|---|
| Behålla Oracle | Stabilitet och låg migreringsrisk | Fortsatt kostnad och koppling |
| Kapsla in Oracle | Minskar direktberoenden | Skapar extra lager att förvalta |
| PostgreSQL för nya tjänster | Tydligare teamägarskap | Kräver driftmodell och kompetens |
| Databas i OpenShift | Självservice och plattformsnärhet | Stateful-drift kan underskattas |
| Extern databas | Mogen drift och tydlig separation | Mer nätverksberoenden |
| Flera datatekniker | Bättre passform per behov | Mer governance och kompetenskrav |
| En databasstandard för allt | Enkel styrning | Dålig passform för specialbehov |

## Anti-patterns

### Byta produkt utan att ändra koppling

Om Oracle ersätts med PostgreSQL men flera system fortfarande delar schema har organisationen inte löst kopplingsproblemet.

### PostgreSQL som ny central monolit

Om alla nya tjänster skriver till samma PostgreSQL-instans och schema återskapas den gamla databasmonoliten i ny form.

### Elasticsearch som primärdatabas

Elasticsearch kan vara utmärkt för sök, men bör normalt inte äga affärskritisk sanning.

### Databasen som integrationsplattform

Att integrera via tabeller kan vara snabbt initialt, men skapar otydliga kontrakt och svår förändring.

### Stateful workload utan restore-test

Backup som aldrig testats är en förhoppning, inte en återställningsförmåga.

## Vanliga fallgropar

- Att börja med databasprodukt före dataroll.
- Att underskatta lagrad affärslogik.
- Att glömma rapport- och batchberoenden.
- Att inte kartlägga vilka system som läser samma schema.
- Att sakna ägare för data.
- Att inte definiera primär sanning.
- Att införa flera datalager utan datagovernance.
- Att köra databas i OpenShift utan driftmodell.
- Att sakna versionerade schemaförändringar.
- Att inte planera avveckling av gamla datavägar.

## Arkitektens checklista

Innan ett databeslut fattas bör arkitekten kunna svara på:

- Vilken dataroll handlar det om?
- Vad är primär sanning?
- Vem äger data?
- Vilka system läser data?
- Vilka system skriver data?
- Finns direkt schemaåtkomst?
- Finns lagrad affärslogik?
- Finns rapport- eller batchberoenden?
- Vilka transaktionskrav finns?
- Vilka tillgänglighetskrav finns?
- Hur sker backup?
- Har restore testats?
- Hur hanteras schemaförändringar?
- Vilken retention gäller?
- Är datalagret lämpligt för OpenShift?
- Behövs ADR?

## Koppling till moderniseringscaset

Nordisk Handel AB analyserar sin Oracle-miljö och upptäcker att databasen har flera roller samtidigt: ordertransaktioner, kundhistorik, rapportering, integrationstabeller och viss affärslogik i lagrade procedurer.

Organisationen beslutar därför att inte ersätta Oracle i första vågen. I stället fattas fyra beslut:

1. Oracle behålls som primär sanning för orderkärnan.
2. Nya tjänster får inte direktläsa Oracle-schema utan ADR.
3. PostgreSQL kan användas för nya avgränsade domäner med tydligt dataägarskap.
4. Sök och rapportering ska gradvis flyttas till sekundära modeller där det ger värde.

Detta gör moderniseringen mer kontrollerad. Oracle behandlas som ett moderniseringsobjekt, inte som ett problem som måste bort direkt.

## Snabb sammanfattning

- Oracle är ofta mer än en databas och måste analyseras efter roll.
- Databasprodukt och dataroll är olika saker.
- Primär sanning måste vara tydlig.
- Delad databas och direkt schemaåtkomst är centrala kopplingsproblem.
- Oracle kan behållas, kapslas in, kompletteras eller ersättas beroende på användningsfall.
- PostgreSQL passar ofta för nya avgränsade relationsbehov med tydligt dataägarskap.
- Databas i OpenShift kräver mogen driftmodell, backup och restore.
- Datagovernance behövs när fler datatekniker införs.

## Nästa steg

Nästa kapitel behandlar messaging och eventdrivna mönster. Där analyseras IBM MQ:s roll, skillnaden mellan kö, kommando och händelse, samt när event streaming eller enklare brokers passar bättre än traditionell köhantering.
