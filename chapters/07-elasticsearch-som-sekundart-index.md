# Kapitel 7: Elasticsearch som sekundärt index

## Varför detta kapitel finns

I tidigare kapitel har vi skiljt mellan primär sanning, sekundära läsmodeller och eventdrivna flöden. Elasticsearch blir relevant just i den skärningspunkten. Många enterprise-system använder relationsdatabaser som Oracle både för transaktioner och för sök, filtrering, rapportnära frågor och operativ insyn. Det fungerar ofta länge, men kan skapa tung belastning, svåra SQL-frågor och hård koppling mellan sökbehov och transaktionsmodell.

Elasticsearch kan vara ett starkt verktyg för sök och indexering. Det kan avlasta Oracle, förbättra användarupplevelsen och ge snabb filtrering över stora datamängder. Men tekniken blir farlig om den behandlas som primär transaktionsdatabas eller otydlig integrationsplattform.

Det här kapitlet visar hur Elasticsearch kan användas som sekundärt index i en moderniserad arkitektur utan att skapa nya dubbla sanningar.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- förklara skillnaden mellan primär sanning och sekundärt index
- identifiera när Elasticsearch passar för sök, filtrering och analysnära läsning
- förstå varför Elasticsearch normalt inte bör vara primär transaktionsdatabas
- välja mellan direkt indexering, eventdriven indexering, CDC och batch
- resonera om eventual consistency, reindexering, retention och åtkomst
- formulera en beslutsmodell för Elasticsearch i en OpenShift-orienterad målarkitektur

## Elasticsearch roll i målarkitekturen

Elasticsearch är optimerat för indexering och sökning i dokumentliknande data. Det passar väl när användare eller system behöver hitta, filtrera, sortera eller analysera data på sätt som skiljer sig från den primära transaktionsmodellen.

Vanliga användningsfall:

- fritextsökning i produktkatalog
- sökning i orderhistorik
- filtrering av ärenden
- dokument- och metadataindex
- operativa vyer över händelser
- loggsökning och felsökning
- analysnära läsmodeller
- snabb åtkomst till sammanställda vyer

I bokens målarkitektur används Elasticsearch främst som sekundärt index. Det innebär att Elasticsearch inte äger affärskritisk primär sanning. Indexet är en härledd modell som kan byggas upp igen från en primär källa eller från händelseflöden.

## Primär sanning och sekundärt index

Primär sanning är den källa som äger korrekt affärstillstånd. Ett sekundärt index är en optimerad kopia eller härledd modell.

| Roll | Beskrivning | Exempel |
|---|---|---|
| Primär sanning | Äger korrekt affärstillstånd | Oracle för ordertransaktioner |
| Sekundärt index | Optimerar sök och läsning | Elasticsearch för orderhistoriksökning |
| Sekundär läsmodell | Härledd vy för särskilt läsbehov | PostgreSQL eller indexerad modell |
| Observability-index | Teknisk sökbar driftdata | Loggar och traces |
| Analysmodell | Aggregerad eller historiserad data | Rapport- eller analyslager |

Ett sekundärt index får gärna vara snabbt och flexibelt. Det ska däremot inte vara den enda platsen där kritisk affärsdata finns.

## När Elasticsearch passar

Elasticsearch passar bra när användningsfallet handlar om att hitta och filtrera data snarare än att äga transaktioner.

Det passar ofta för:

- sökning över många fält
- fritextsökning
- facetterad filtrering
- snabb order- eller produktlookup
- indexering av dokumentmetadata
- operativ sökning över händelser
- sökfunktioner som belastar Oracle för mycket
- läsmodeller där viss fördröjning accepteras

Elasticsearch passar särskilt bra när läsmodellen skiljer sig från skrivmodellen. Oracle kan äga ordern, medan Elasticsearch ger kundtjänst ett snabbt sätt att hitta ordern via kund, datum, artikel, status eller fritext.

## När Elasticsearch inte passar

Elasticsearch passar sämre när kraven handlar om stark transaktionalitet, korrekt primärt tillstånd eller relationsintegritet.

Var försiktig när behovet är:

- affärskritisk primärdatabas
- komplexa transaktioner
- strikt referentiell integritet
- finansiell bokföring
- korrigering av primärdata
- stark konsistens mellan flera objekt
- datalager där varje skrivning måste vara omedelbart korrekt

Om en applikation skriver direkt till Elasticsearch och behandlar indexet som affärssanning har organisationen ofta skapat ett datagovernance-problem.

## Dataflöden till Elasticsearch

Ett index behöver matas med data. Valet av indexeringsmönster påverkar koppling, latens, robusthet och felsökning.

| Mönster | Beskrivning | Passar när |
|---|---|---|
| Direkt indexering | Applikationen skriver till primärdatabas och index | Enkelt fall med tydligt ansvar |
| Eventdriven indexering | Domänhändelser uppdaterar index asynkront | Flera konsumenter behöver samma förändring |
| Change Data Capture | Databasförändringar fångas från primärdatabas | Legacy kan inte publicera events |
| Batchindexering | Index byggs eller uppdateras periodiskt | Realtid behövs inte |
| Reindexering | Index byggs om från primär källa | Återställning, schemaändring eller kvalitetssäkring |

Det viktigaste är att indexflödet är explicit. Ett sökindex som “bara råkar uppdateras någonstans” blir svårt att felsöka och lita på.

## Direkt indexering

Direkt indexering innebär att applikationen skriver både till primärdatabasen och Elasticsearch. Det kan vara enkelt att förstå men kräver försiktighet.

Fördelar:

- låg latens
- enkel kedja i små system
- tydlig kontroll i samma applikation

Risker:

- dubbel skrivlogik
- olika resultat om ena skrivningen lyckas och andra misslyckas
- hård koppling till indexets schema
- svår rollback
- applikationen får ansvar för synkronisering

Direkt indexering passar bäst när scope är litet och felhantering är tydlig.

## Eventdriven indexering

Eventdriven indexering innebär att primärsystemet publicerar en händelse, till exempel `OrderSkapad` eller `OrderStatusUppdaterad`. En indexeringstjänst konsumerar händelsen och uppdaterar Elasticsearch.

Fördelar:

- lösare koppling
- flera konsumenter kan använda samma händelse
- indexering kan skalas separat
- tydlig relation till domänhändelser
- bättre passform för moderniserade flöden

Risker:

- eventual consistency
- behov av idempotens
- risk för consumer lag
- schema- och eventversionering krävs
- felhantering och dead-letter behövs

Eventdriven indexering är ofta ett bra mål för nya och moderniserade domäner.

## Change Data Capture

Change Data Capture, CDC, innebär att förändringar fångas från databasen och används för att uppdatera indexet. Det kan vara värdefullt när legacy-system inte enkelt kan publicera domänhändelser.

Fördelar:

- mindre kodändring i legacy
- kan användas som övergångsmönster
- gör det möjligt att bygga läsmodeller från befintlig Oracle-data

Risker:

- stärker beroendet till databasschema
- förändringar saknar ibland affärssemantik
- tekniska tabelländringar blir integrationshändelser
- kräver drift och övervakning
- kan vara svårt att tolka komplexa transaktioner

CDC bör ofta ses som ett praktiskt moderniseringsmönster, men inte alltid som slutmål.

## Batch och reindexering

Batchindexering innebär att indexet uppdateras periodiskt. Det passar när data inte behöver vara nära realtid.

Reindexering innebär att indexet byggs om från primär källa eller händelsehistorik. Det är en nödvändig förmåga när:

- indexschema ändras
- data har blivit fel
- Elasticsearch-kluster behöver återskapas
- nya fält ska läggas till
- kvalitet behöver verifieras
- miljöer ska byggas upp från grunden

Ett sekundärt index bör normalt kunna återskapas. Om det inte går att återskapa har indexet i praktiken blivit primärdata.

## Eventual consistency

När indexering sker asynkront uppstår fördröjning. Primärsystemet kan vara uppdaterat innan Elasticsearch visar samma läge.

Detta kallas eventual consistency. För sök är det ofta acceptabelt, men det måste vara förstått.

Frågor att ställa:

- hur färskt måste indexet vara?
- är sekunders fördröjning acceptabel?
- är minuters fördröjning acceptabel?
- vad visar användaren under fördröjning?
- hur upptäcks indexeringsfel?
- kan användaren gå till primärkälla vid osäkerhet?
- finns mätetal för indexeringsfördröjning?

Eventual consistency är inte ett problem om verksamheten förstår konsekvensen. Det blir ett problem om användarna förväntar sig stark konsistens.

## Indexdesign och ägarskap

Ett Elasticsearch-index behöver ägare. Indexschema, mappings, fält, retention och reindexering får inte vara otydliga tekniska detaljer.

Indexägaren bör ansvara för:

- vilka data som ingår
- vad fälten betyder
- hur index uppdateras
- hur schema ändras
- hur reindexering görs
- vilken retention som gäller
- vem som får läsa data
- vilka SLO:er som gäller
- hur indexeringsfel hanteras

Om flera team skriver till samma index utan gemensamt ägarskap uppstår snabbt datakvalitetsproblem.

## Säkerhet och känsliga data

Elasticsearch kan innehålla känslig information, särskilt när det används för loggar, kunddata, orderhistorik eller dokumentmetadata.

Säkerhetsfrågor:

- finns personuppgifter i indexet?
- finns affärskänslig information?
- vem får söka i indexet?
- behöver fält maskeras?
- hur hanteras radering?
- hur hanteras retention?
- loggas tokens eller credentials?
- skiljer man på observability-data och affärsdata?
- auditeras åtkomst?

Sökbarhet är kraftfullt. Just därför behöver åtkomst och dataklassning vara tydlig.

## Elasticsearch och OpenShift

Elasticsearch kan köras på eller utanför OpenShift beroende på plattformens mognad och krav. Oavsett placering är det en stateful komponent med krav på lagring, kapacitet och drift.

Frågor innan Elasticsearch körs i OpenShift:

- finns godkänd operator eller driftmodell?
- vilken storage class används?
- hur hanteras backup eller reindexering?
- hur övervakas klusterhälsa?
- hur hanteras uppgraderingar?
- hur separeras affärsindex från loggindex?
- hur planeras kapacitet?
- vem äger incidenter?

OpenShift kan förenkla deployment, men tar inte bort behovet av Elasticsearch-kompetens.

## Observability-data och affärsdata

Elasticsearch används ofta för loggar och observability. Det är ett annat användningsfall än affärssök.

| Område | Typiska krav |
|---|---|
| Observability-data | Hög volym, kortare retention, felsökning, teknisk åtkomst |
| Affärssök | Datakvalitet, åtkomstkontroll, tydlig primär källa, verksamhetsnytta |
| Auditdata | Strikt retention, integritet, spårbarhet |
| Analysnära data | Aggregering, historik, ofta separat governance |

Att blanda alla dessa i samma modell kan skapa kostnads-, säkerhets- och driftproblem.

## Beslutsguide: när ska Elasticsearch användas?

| Scenario | Rekommendation |
|---|---|
| Fritextsökning över många objekt | Använd Elasticsearch som sekundärt index |
| Orderhistoriksökning belastar Oracle | Bygg index från primär källa eller events |
| Primär transaktionshantering | Använd inte Elasticsearch som primärdatabas |
| Loggsökning | Använd med tydlig retention och åtkomst |
| Kunddata med känsliga fält | Använd endast med dataklassning och åtkomstkontroll |
| Rapportering med relationslogik | Överväg analytiskt lager eller läsdatabas |
| Realtidsnära operativ vy | Använd eventdriven indexering om fördröjning accepteras |
| Temporär pilot | Definiera ändå ägare, datakälla och avveckling |

## Beslutsguide: välj indexeringsmönster

| Behov | Rekommenderat mönster |
|---|---|
| Ny tjänst publicerar domänhändelser | Eventdriven indexering |
| Legacy Oracle saknar events | CDC som övergångsmönster |
| Data ändras sällan | Batchindexering |
| Index måste kunna återskapas | Reindexering från primär källa |
| Låg latens är viktig | Direkt eller nära realtidsindexering med tydlig felhantering |
| Flera konsumenter behöver förändringar | Event streaming som källa |
| Starkt konsistenskrav | Sök om Elasticsearch verkligen är rätt |

## Tradeoffs

| Val | Fördel | Risk |
|---|---|---|
| Elasticsearch som sekundärt index | Snabb sök och avlastning | Synkroniseringsmodell krävs |
| Direkt indexering | Låg latens och enkel start | Dubbel skrivlogik |
| Eventdriven indexering | Lösare koppling | Eventual consistency och mer infrastruktur |
| CDC | Fungerar med legacy | Databasschema blir integrationskälla |
| Batchindexering | Enkel och robust | Fördröjning |
| Lång retention | Mer historik | Kostnad och dataskyddsrisk |
| Kort retention | Lägre kostnad | Mindre felsökningshistorik |
| Ett gemensamt kluster | Enkel start | Blandade krav och störningsrisk |

## Anti-patterns

### Elasticsearch som primärdatabas

Det tydligaste anti-patternet är att låta Elasticsearch äga affärskritisk sanning utan transaktionsmodell och tydligt dataägarskap.

### Index utan reindexeringsstrategi

Om indexet inte kan återskapas blir det svårt att hantera fel, schemaändringar och återställning.

### Logga allt för alltid

Att behålla all loggdata utan retention och dataklassning skapar kostnads- och compliance-risk.

### Sökindex som integrationsplattform

Andra system ska inte läsa från Elasticsearch bara för att data råkar finnas där, om indexet inte är avsett som kontrakt.

### Otydlig eventual consistency

Om användare tror att sökresultatet alltid är omedelbart korrekt kan fördröjningar tolkas som fel.

## Vanliga fallgropar

- Att inte definiera primär källa.
- Att sakna ägare för indexet.
- Att underskatta datavolym.
- Att blanda loggdata och affärsdata utan åtkomstmodell.
- Att inte mäta indexeringsfördröjning.
- Att sakna dead-letter-hantering för indexeringsfel.
- Att lägga känsliga uppgifter i index utan dataklassning.
- Att sakna retentionpolicy.
- Att förutsätta stark konsistens.
- Att inte dokumentera reindexeringsprocess.

## Arkitektens checklista

Innan Elasticsearch införs bör arkitekten kunna svara på:

- Vilket problem löser Elasticsearch?
- Är det sök, logg, analys eller operativ vy?
- Vad är primär sanning?
- Kan indexet återskapas?
- Hur matas indexet?
- Är eventual consistency acceptabelt?
- Vem äger indexschema?
- Hur versioneras index?
- Hur hanteras reindexering?
- Vilken retention gäller?
- Finns personuppgifter eller känsliga data?
- Vem får läsa indexet?
- Hur mäts indexeringsfördröjning?
- Hur hanteras indexeringsfel?
- Ska Elasticsearch köras i OpenShift eller utanför?
- Behövs ADR?

## Koppling till moderniseringscaset

Nordisk Handel AB har länge använt Oracle för både transaktioner och sökning i orderhistorik. Kundtjänst behöver kunna söka snabbt på kund, datum, artikelnummer, status och fritext. Sökfrågorna belastar Oracle och kräver specialanpassade vyer.

Organisationen beslutar att införa Elasticsearch som sekundärt index för orderhistoriksökning. Oracle fortsätter vara primär sanning. Indexet byggs initialt via CDC för legacy-delar och på sikt via domänhändelser när orderflödet moderniseras.

Följande beslut dokumenteras:

- Elasticsearch får inte vara primär databas.
- Indexet ska kunna återskapas.
- Sökresultat får vara fördröjda inom definierad gräns.
- Indexeringsfördröjning ska mätas.
- Kunddata i indexet ska dataklassas.
- Åtkomst till produktionsindex begränsas.
- Reindexering ska testas innan produktion.

Detta ger verksamhetsnytta utan att flytta kärntransaktionerna för tidigt.

## Snabb sammanfattning

- Elasticsearch passar väl för sök, filtrering och sekundära läsmodeller.
- Elasticsearch bör normalt inte äga primär affärssanning.
- Primär källa, indexägare och reindexeringsstrategi måste vara tydliga.
- Eventdriven indexering passar moderniserade flöden, medan CDC kan vara ett övergångsmönster.
- Eventual consistency är ofta acceptabelt för sök men måste kommuniceras.
- Retention, åtkomst och dataklassning är centrala governance-frågor.
- Elasticsearch i OpenShift kräver stateful driftmodell och kapacitetsplanering.

## Nästa steg

Nästa kapitel behandlar Ceph och stateful workloads. Där går vi igenom block-, fil- och objektlagring, persistent volumes, backup och varför stateful drift i OpenShift kräver särskild arkitektonisk disciplin.
