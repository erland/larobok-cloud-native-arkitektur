# Kapitel 1: Varför cloud native-modernisering?

## Varför detta kapitel finns

Många större organisationer har byggt sina viktigaste system på en stabil enterprise-stack: Java EE-applikationer, traditionella applikationsservrar, Oracle Database, IBM MQ, etablerade driftprocesser och tydliga förvaltningsmodeller. Den typen av miljö har ofta fungerat väl under lång tid. Den har gett robusthet, transaktionell säkerhet, centraliserad kontroll och förutsägbara driftmönster.

Problemet är sällan att den gamla arkitekturen var fel. Problemet är att den ofta är optimerad för en annan förändringstakt än dagens organisationer behöver.

När verksamheten vill leverera oftare, skala mer flexibelt, automatisera mer, återhämta sig snabbare vid fel och ge produktteam större ansvar räcker det sällan att bara installera en containerplattform. En Java EE-applikation som flyttas oförändrad till en container kan fortfarande ha samma gamla beroenden, samma releaseproblem, samma integrationskopplingar och samma svårigheter kring data.

Det här kapitlet etablerar bokens grundidé: cloud native-modernisering handlar inte om att byta teknik för teknikens skull. Det handlar om att öka organisationens förändringsförmåga utan att tappa kontroll över säkerhet, data, drift och affärskritiska flöden.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- beskriva varför traditionella Java EE-miljöer ofta behöver moderniseras
- skilja mellan containerisering och cloud native-modernisering
- identifiera typiska drivkrafter bakom modernisering på Red Hat-stack
- förstå varför Oracle och IBM MQ bör analyseras som moderniseringsobjekt snarare än ersättas reflexmässigt
- formulera de första arkitekturfrågorna inför en moderniseringsresa

## Utgångsläget: stabilt men trögrörligt

I bokens återkommande scenario använder organisationen Nordisk Handel AB en affärskritisk plattform byggd på Java EE. Plattformen använder Oracle som central databas och IBM MQ som integrationsnav. Systemen har utvecklats under lång tid och hanterar centrala affärsflöden: order, kunder, lager, betalningar och externa integrationer.

Det finns mycket som fungerar:

- transaktioner är väl förstådda
- driftorganisationen kan systemen
- integrationer är beprövade
- support- och förvaltningsprocesser finns
- databasen innehåller historik och affärskritisk logik
- köflöden hanterar viktiga beroenden mellan system

Samtidigt finns tydliga begränsningar:

- releasecykler är långa
- miljöer skiljer sig åt
- deployment kräver många manuella steg
- applikationer är beroende av serverkonfiguration
- flera system delar databas eller schema
- IBM MQ används för fler integrationsmönster än det från början var tänkt för
- felsökning över flera system är tidskrävande
- nya team får svårt att leverera självständigt

Den här typen av nuläge är vanlig. Det är inte ett tecken på misslyckande. Det är ett tecken på att arkitekturen har vuxit fram under lång tid och nu behöver utvecklas.

## Moderniseringens verkliga mål

Ett vanligt misstag är att beskriva modernisering som ett teknikbyte:

- från VM till container
- från Java EE till modern Java-runtime
- från Oracle till PostgreSQL
- från IBM MQ till Kafka eller annan meddelandeplattform
- från traditionell lagring till Ceph
- från serverdrift till OpenShift

Dessa teknikförändringar kan vara relevanta, men de är inte målet i sig. Målet bör formuleras i termer av förmågor.

| Förmåga | Vad den betyder i praktiken |
|---|---|
| Kortare ledtid | Kod kan gå från ändring till produktion snabbare och säkrare |
| Reproducerbara miljöer | Samma applikation beter sig lika i test, staging och produktion |
| Tydligare ansvar | Team vet vad de äger och vad plattformen äger |
| Bättre driftbarhet | System kan observeras, felsökas och återställas |
| Kontrollerad skalning | Applikationer och plattform kan växa utan manuell serverhantering |
| Mindre koppling | System kan ändras utan att många andra system måste ändras samtidigt |
| Bättre säkerhetsstyrning | Policyer och guardrails byggs in i plattformen |
| Lägre förändringsrisk | Modernisering sker stegvis och kan rullas tillbaka |

Om tekniken inte stärker någon av dessa förmågor är det inte självklart att den ska införas.

## Containerisering är inte samma sak som cloud native

Containerisering innebär att en applikation paketeras som en container image och körs i en container runtime. Det kan ge bättre reproducerbarhet och tydligare paketering, men det gör inte automatiskt applikationen cloud native.

En containeriserad Java EE-applikation kan fortfarande:

- kräva manuell serverkonfiguration
- använda lokal disk för viktigt tillstånd
- sakna health checks
- skriva loggar till lokala filer
- vara beroende av statiska hostnamn
- kräva sticky sessions
- vara hårt kopplad till Oracle-schema
- förutsätta IBM MQ-konfiguration i runtime
- sakna tydlig modell för graceful shutdown
- vara svår att skala horisontellt

Cloud native-modernisering kräver därför mer än ett Containerfile. Den kräver att applikation, plattform och arbetssätt anpassas till en miljö där instanser är ersättningsbara, konfiguration är externaliserad, driftbarhet är inbyggd och ansvar är tydligt.

## Red Hat-stackens roll

I den här boken används Red Hat-stacken som strategisk referenspunkt. Det innebär framför allt:

- Podman för lokal containerförståelse och verifiering
- OpenShift som enterprise-plattform för containerdrift
- Red Hat-orienterade mönster för säkerhet, operators, automation och plattformsguardrails
- en arkitektur där plattformen blir en intern produkt snarare än bara ett kluster

OpenShift är särskilt relevant i större organisationer eftersom plattformen inte bara handlar om att starta containers. Den kan också fungera som ett standardiserat lager för deployment, säkerhet, nätverk, image-policy, secrets, observability och självservice.

Men OpenShift löser inte automatiskt arkitekturproblem. En dåligt designad applikation blir inte väldesignad av att köras på OpenShift. En otydlig datamodell blir inte tydlig av att ligga bakom en route. En integrationsmonolit blir inte eventdriven bara för att man byter transportteknik.

## Oracle och IBM MQ som moderniseringsobjekt

I många moderniseringsdiskussioner behandlas Oracle och IBM MQ som problem som ska bort. Det är för enkelt.

Oracle kan vara ett mycket rimligt val för affärskritiska transaktioner, historik, dataintegritet och etablerad drift. IBM MQ kan vara ett mycket rimligt val för robusta legacy-integrationer och pålitliga meddelandeflöden.

Frågan är inte om teknikerna är bra eller dåliga. Frågan är vilken roll de har fått.

Oracle bör analyseras utifrån frågor som:

- Är databasen primär transaktionskälla eller även integrationsyta?
- Delar flera system samma schema?
- Finns affärslogik i lagrade procedurer?
- Används databasen för rapportering, sök och köliknande beteenden?
- Är Oracle standardval för alla nya behov av gammal vana?

IBM MQ bör analyseras utifrån frågor som:

- Är flödet ett kommando, en händelse, request/reply eller batchliknande integration?
- Finns okända konsumenter?
- Har MQ blivit central integrationsmonolit?
- Behöver nya tjänster direktkopplas till MQ?
- Finns behov där event streaming eller enklare brokers passar bättre?

Modernisering innebär ofta att Oracle och IBM MQ först kapslas in och kompletteras, inte att de omedelbart ersätts.

## Arkitekturproblem i traditionella miljöer

### Delad databas som systemgräns

När flera applikationer läser och skriver i samma databas försvinner tydliga tjänstegränser. Schemaändringar blir riskfyllda, dataägarskap blir otydligt och nya tjänster tvingas förstå gamla interna modeller.

### Central MQ som integrationsnav

En central köplattform kan ge robusthet, men kan också bli ett nav där alla förändringar måste samordnas. Om meddelandekontrakt är otydliga skapas hård semantisk koppling även om transporten är asynkron.

### Serverkonfiguration som dold arkitektur

Datasources, JMS-resurser, certifikat, bibliotek och miljöparametrar kan ligga i applikationsservern snarare än i applikationens explicita leveransmodell. Det gör miljöer svåra att återskapa.

### Driftbarhet som eftertanke

Loggar, metrics, health checks och tracing läggs ofta till när problem uppstår. I en distribuerad miljö behöver dessa egenskaper finnas från början.

### Otydligt ansvar

När utveckling, drift, middleware, databas, säkerhet och arkitektur äger olika delar utan gemensam modell blir förändring långsam. Cloud native kräver tydligare ansvar, inte mindre ansvar.

## Beslutsguide: är modernisering motiverad?

| Fråga | Om svaret är ja | Möjlig slutsats |
|---|---|---|
| Är releaseledtiden ett verksamhetsproblem? | Team väntar länge på produktion | Automatisering och plattformsstandarder kan ge värde |
| Skiljer sig miljöer åt? | Test och produktion beter sig olika | Containerisering och externaliserad konfiguration är relevant |
| Är felsökning över systemgränser svår? | Incidenter tar lång tid | Observability bör prioriteras |
| Är databasen integrationsyta? | Många system delar schema | Dataägarskap och inkapsling behövs |
| Är MQ nav för all integration? | Alla flöden passerar samma tekniska mönster | Messaging bör klassificeras per mönster |
| Saknas tydlig ägare för plattformen? | Team uppfinner egna lösningar | OpenShift bör produktifieras |
| Är driftkrav otydliga? | Ingen vet vad produktionsklar betyder | Minimikrav och golden paths behövs |
| Är legacy-risk hög? | Big bang vore farligt | Strangler pattern och samexistens är lämpligt |

## Beslutsguide: containerisera, kapsla in eller modernisera?

| Situation | Rekommenderad första strategi |
|---|---|
| Applikationen är relativt stateless och har få serverberoenden | Containerisera och anpassa för OpenShift |
| Applikationen har många serverberoenden men högt affärsvärde | Kapsla in och modernisera stegvis |
| Applikationen är starkt kopplad till Oracle-schema | Kartlägg dataägarskap före flytt |
| Applikationen är beroende av IBM MQ-flöden | Klassificera messagingmönster före teknikbyte |
| Funktionen är ny och avgränsad | Bygg cloud native från start |
| Funktionen är nära avveckling | Undvik dyr modernisering |
| Funktionen har högt värde men låg teknisk risk | Bra kandidat för första pilot |

## Tradeoffs

| Val | Fördel | Risk |
|---|---|---|
| Snabb containerisering | Ger tidigt lärande och synlig framdrift | Kan flytta gamla problem till ny plattform |
| Djup omskrivning | Kan ge bättre långsiktig målbild | Hög risk, lång ledtid och sen nytta |
| Behålla Oracle initialt | Lägre risk för kärndata | Koppling och kostnad kan bestå |
| Ersätta Oracle för nya domäner | Bättre dataägarskap | Kräver ny driftmodell |
| Behålla IBM MQ för legacy | Stabilitet | Integrationsmonolit kan leva vidare |
| Införa event streaming | Flera konsumenter och återspel | Kräver governance och ny kompetens |
| Produktifiera OpenShift tidigt | Bättre onboarding och styrning | Kräver investering i plattformsteam |
| Ge team full frihet | Snabb lokal innovation | Spretighet och högre driftkostnad |

## Anti-patterns

### Lift and shift till OpenShift

Att flytta applikationer till OpenShift utan att förändra konfiguration, logging, health, ansvar eller beroenden skapar ofta mer komplexitet utan motsvarande nytta.

### Allt ska bli mikrotjänster

Mikrotjänster är inte ett mål i sig. Utan observability, automation, dataägarskap och teammognad kan mikrotjänster skapa mer koppling och fler felmoder.

### Byt teknik före mönster

Att ersätta IBM MQ med Kafka, Oracle med PostgreSQL eller VM med containers utan att förstå mönstret löser sällan grundproblemet.

### Plattform utan produktägarskap

OpenShift utan onboarding, golden paths, supportmodell och guardrails blir ofta bara ett kluster som varje team måste tolka själv.

### Modernisering utan avveckling

Om nya komponenter byggs men gamla aldrig avvecklas får organisationen dubbel komplexitet permanent.

## Vanliga fallgropar

- Att börja med verktyg före målbild.
- Att underskatta data- och integrationsberoenden.
- Att behandla Oracle och IBM MQ som enkla ersättningsproblem.
- Att sakna gemensam definition av produktionsklar applikation.
- Att införa OpenShift utan plattformsteamets produktansvar.
- Att inte budgetera för samexistens och avveckling.
- Att sakna mätetal för modernisering.
- Att inte dokumentera arkitekturbeslut.
- Att fokusera på första deployment snarare än långsiktig driftbarhet.

## Arkitektens checklista

Innan moderniseringsresan startar bör arkitekten kunna svara på:

- Vilken förändringsförmåga saknas idag?
- Vilka affärsproblem ska moderniseringen lösa?
- Vilka applikationer är lämpliga första kandidater?
- Vilka komponenter är affärskritiska och bör hanteras försiktigt?
- Var finns primär sanning?
- Vilka integrationer är kommandon, händelser eller request/reply?
- Vilka delar av miljön är serverbundna?
- Vad ska OpenShift-plattformen erbjuda som produkt?
- Vilka minimikrav gäller för produktionsklara tjänster?
- Hur dokumenteras beslut?
- Hur mäts framgång?
- Hur planeras avveckling av det gamla?

## Koppling till moderniseringscaset

För Nordisk Handel AB leder kapitlets resonemang till en första överenskommelse: moderniseringsprogrammet ska inte definieras som “flytta Java EE till OpenShift”. Det ska definieras som “öka förändringsförmågan för affärskritiska tjänster utan att tappa kontroll över data, integration och drift”.

Organisationen väljer därför att börja med en avgränsad funktion där nyttan är tydlig och risken hanterbar. Oracle och IBM MQ behålls initialt där de är affärskritiska, men deras roller analyseras och dokumenteras. OpenShift införs som intern plattformsprodukt, inte som en ny serverhall.

Detta ger en stabil grund för resten av boken.

## Snabb sammanfattning

- Traditionella Java EE-miljöer är ofta stabila men trögrörliga.
- Cloud native-modernisering handlar om förändringsförmåga, inte teknikbyte i sig.
- Containerisering är bara ett steg; cloud native kräver driftbarhet, ansvar och explicit arkitektur.
- Oracle och IBM MQ ska analyseras efter roll och användningsfall.
- OpenShift behöver produktifieras för att ge verkligt enterprise-värde.
- Modernisering bör ske stegvis, med mätbara mål och tydliga beslut.

## Nästa steg

Nästa kapitel går vidare till containerteknik med Podman och OCI. Där blir containerformatet konkret: image, runtime, Containerfile, rootless containers och hur lokal verifiering kan användas för att upptäcka arkitekturproblem tidigt.
