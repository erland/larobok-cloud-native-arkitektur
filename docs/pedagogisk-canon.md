# Pedagogisk canon

## Pedagogisk profil
- Språk: Svenska.
- Nivå: Erfaren.
- Läsarprofil: Lösningsarkitekter och erfarna Java EE-utvecklare.
- Ton: Professionell, pragmatisk och beslutsorienterad.
- Repetitionstakt: Kort repetition när tidigare begrepp används i ett nytt sammanhang.

## Återkommande scenario
Organisationen Nordisk Handel AB driver en affärskritisk Java EE-plattform med traditionell applikationsserver, Oracle Database och IBM MQ. Målet är att stegvis modernisera mot OpenShift, Podman, containeranpassade databastjänster, modern messaging, Elasticsearch och Ceph.

## Arkitekturprinciper
- Modernisering ska ske stegvis, inte som big bang.
- Containerisering är inte samma sak som cloud native.
- State ska hanteras explicit och inte gömmas i applikationscontainrar.
- Plattformen ska erbjuda guardrails snarare än obegränsad frihet.
- Teknikval ska motiveras med tradeoffs, inte med trendvärde.
- Legacy-komponenter kan samexistera med nya komponenter under övergångsperioder.

## Kapitelkomponenter
Varje kapitel bör innehålla:
- Varför området är viktigt.
- Arkitekturproblem i traditionella miljöer.
- Hur cloud native förändrar spelplanen.
- Beslutsguide.
- Tradeoffs.
- Referensmönster.
- Anti-patterns.
- Vanliga fallgropar.
- Arkitektens checklista.
- Koppling till moderniseringscaset.
- Nästa steg.

## Versions- och faktaval
Exakta produktversioner ska verifieras mot officiell dokumentation när kapitel skrivs. Boken ska undvika onödigt versionsberoende där arkitekturprinciper räcker.


## Uppdatering efter Kapitel 1

### Introducerade huvudbegrepp
- Cloud native: arkitektur- och driftmodell som utnyttjar automatisering, resiliens, observerbarhet och skalbarhet.
- Containerized: applikation som är paketerad och körd i container utan att nödvändigtvis vara cloud native.
- Guardrails: plattformsregler och standarder som ger team självservice inom säkra ramar.
- Stateful workload: arbetslast där tillstånd behöver bevaras och hanteras explicit.
- Moderniseringsdrivare: organisatoriska och tekniska skäl att förändra plattform, arbetssätt eller arkitektur.

### Etablerade arkitekturprinciper
- Modernisering ska börja med önskad förändringsförmåga, inte med teknikval.
- OpenShift ska behandlas som enterprise-plattform, inte som en ny serverhall.
- Oracle och IBM MQ ska analyseras efter användningsfall, inte ersättas reflexmässigt.
- Första moderniseringsvåg bör vara avgränsad, mätbar och riskbalanserad.

## Uppdatering efter Kapitel 9

### Introducerade huvudbegrepp
- Sekundärt sökindex: index optimerat för sök och läsning utan att äga primär sanning.
- Primär sanning: datakälla som äger affärskritiskt korrekt tillstånd.
- Reindexering: process för att återskapa ett index från primär källa eller händelseflöde.
- Retention: regler för hur länge data ska behållas.
- CDC: Change Data Capture, mönster för att fånga förändringar från databas.
- Eventual consistency: modell där sekundära vyer blir konsistenta över tid.

### Etablerade arkitekturprinciper
- Elasticsearch ska normalt användas som sekundärt index, inte primär affärsdatabas.
- Index ska ha ägare, schema, retention och reindexeringsstrategi.
- Observability-data och affärssök kan ha olika krav och bör inte blandas okritiskt.
- Eventual consistency ska dokumenteras när sökindex uppdateras asynkront.


## Uppdatering efter Kapitel 10

### Introducerade huvudbegrepp
- Blocklagring: lagring som exponeras som blockenhet, ofta för databaser och stateful workloads.
- Fillagring: lagring som exponeras som filsystem, ofta för delade filer eller legacy-behov.
- Objektlagring: API-baserad lagring för objekt, dokument och binära filer.
- Persistent volume: plattformsresurs för lagring som överlever poddens livscykel.
- Storage class: klassificering av lagring med egenskaper som prestanda, kostnad och skyddsnivå.
- RPO/RTO: mål för dataförlust respektive återställningstid.

### Etablerade arkitekturprinciper
- Persistent volume är inte backup.
- Storage classes ska ha dokumenterad prestanda, retention, kostnad, backupstöd och ägarskap.
- Ceph ska behandlas som strategisk lagringsplattform med driftmodell, inte som enkel teknisk detalj.
- Delad fillagring ska användas försiktigt och helst som övergångsmönster.
- Objektlagring bör övervägas för dokument, filer och binära objekt.


## Uppdatering efter Kapitel 11

### Introducerade huvudbegrepp
- Strangler pattern: moderniseringsmönster där ny funktionalitet gradvis ersätter delar av ett befintligt system.
- Inkapsling: minskning av direkta beroenden till legacy genom API, adapter eller fasad.
- Samexistens: kontrollerad period där legacy och ny lösning körs parallellt.
- Trafikstyrning: mekanism för att styra användare, funktioner eller trafik mellan gammal och ny lösning.
- Shadow traffic: valideringsmönster där ny lösning får samma input utan att påverka resultatet.
- Dubbel skrivning: riskfyllt mönster där data skrivs till både gammal och ny modell.

### Etablerade arkitekturprinciper
- Stegvis migrering ska föredras framför big bang för affärskritiska system.
- Inkapsling bör ofta föregå ersättning.
- Samexistens ska ha avvecklingsplan.
- Första migreringskandidat ska väljas utifrån affärsvärde, risk och lärande.
- Dataägarskap och integrationssemantik måste vara tydliga före större migrering.


## Uppdatering efter Kapitel 14

### Introducerade huvudbegrepp
- Observability: förmåga att förstå systemets beteende genom signaler som loggar, metrics och tracing.
- Korrelations-id: gemensamt id som följer ett flöde genom flera system.
- SLI: Service Level Indicator, mätetal för tjänstens beteende.
- SLO: Service Level Objective, målnivå för ett SLI.
- Runbook: operativ instruktion för felsökning och åtgärd.
- Larmtrötthet: situation där för många eller dåliga larm gör att viktiga larm ignoreras.

### Etablerade arkitekturprinciper
- Observability ska införas före eller samtidigt med produktion, inte efter incidenter.
- Larm ska vara åtgärdsbara och kopplade till ansvar.
- Kritiska affärsflöden ska kunna följas end-to-end.
- Operativ mognad är en del av produktionsklarhet.


## Uppdatering efter Kapitel 15

### Introducerade huvudbegrepp
- ADR: kort dokument som beskriver arkitekturbeslut, alternativ och konsekvenser.
- Fitness function: mätbart kriterium som visar om arkitekturen rör sig åt önskat håll.
- Teknikradar: modell för att kommunicera vilka tekniker och mönster som rekommenderas, prövas, utvärderas eller bör undvikas.
- Beslutstyp: klassificering av beslut som standard, rekommendation, undantag, experiment eller förbud.
- Governance: styrning som gör teknikval spårbara och konsekventa.

### Etablerade arkitekturprinciper
- Teknikval ska dokumenteras med problem, alternativ, tradeoffs, ansvar och omprövningspunkt.
- Standarder kräver ägare, driftmodell och support.
- Undantag ska ha motivering, ägare och gärna slutdatum.
- Governance ska göra rätt väg enkel, inte skapa onödiga flaskhalsar.


## Uppdatering efter Kapitel 16

### Introducerade huvudbegrepp
- Målarkitektur: styrande riktning för framtida arkitektur, inte statiskt slutdiagram.
- Roadmap: tidsatt plan för stegvis modernisering.
- Golden path: rekommenderat och stödd standardväg för team.
- Moderniseringsförmåga: organisationens förmåga att kontinuerligt förändra arkitektur, plattform och arbetssätt med kontroll.

### Slutlig canon
Boken avslutas med en sammanhållen målarkitektur där OpenShift är produktifierad plattform, Podman används för lokal verifiering, Oracle och IBM MQ hanteras som moderniseringsobjekt snarare än reflexmässiga problem, Elasticsearch används som sekundärt index, Ceph används efter lagringsroll och alla större beslut styrs med ADR, teknikradar och fitness functions.


## Uppdatering efter omskrivet Kapitel 1

### Centrala begrepp introducerade
- Cloud native-modernisering
- Containerisering
- Förändringsförmåga
- Red Hat-stack
- Plattform som produkt
- Lift and shift
- Moderniseringsobjekt

### Etablerade principer
- Modernisering ska utgå från förmågor, inte teknikbyte.
- Containerisering är inte samma sak som cloud native.
- Oracle och IBM MQ ska analyseras efter roll och användningsfall.
- OpenShift ska behandlas som intern plattformsprodukt.
- Modernisering ska ske stegvis och med tydliga avvecklingsplaner.


## Uppdatering efter Kapitel 1 - återskapat

### Centrala begrepp introducerade
- Cloud native-modernisering
- Containerisering
- Förändringsförmåga
- Red Hat-stack
- Plattform som produkt
- Lift and shift
- Moderniseringsobjekt

### Etablerade principer
- Modernisering ska utgå från förmågor, inte teknikbyte.
- Containerisering är inte samma sak som cloud native.
- Oracle och IBM MQ ska analyseras efter roll och användningsfall.
- OpenShift ska behandlas som intern plattformsprodukt.
- Modernisering ska ske stegvis och med tydliga avvecklingsplaner.


## Uppdatering efter Kapitel 2

### Centrala begrepp introducerade
- Container image
- Container
- Container runtime
- Registry
- OCI
- Podman
- Containerfile
- Bas-image
- Rootless containers

### Etablerade principer
- Containerfile ska behandlas som arkitekturdokument.
- Podman är lokal kontrollpunkt, inte produktionsvalidering.
- Samma image bör kunna användas i flera miljöer.
- Rootless-kompatibilitet är en viktig signal för containerhygien.
- Bas-images ska ha tydligt ägarskap och patchmodell.


## Uppdatering efter Kapitel 3

### Centrala begrepp introducerade
- OpenShift som intern plattformsprodukt
- Project och namespace
- Deployment
- Service
- Route
- ConfigMap
- Secret
- Operator
- Guardrails
- Golden path
- OpenShift-readiness

### Etablerade principer
- OpenShift ska betraktas som intern enterprise-plattform, inte bara kluster.
- Självservice kräver guardrails.
- Namespace-modellen är ett tidigt arkitekturbeslut.
- Operators ska behandlas som plattformsbeslut med tydligt ägarskap.
- En lokalt fungerande container är inte automatiskt OpenShift-ready.


## Uppdatering efter Kapitel 4

### Centrala begrepp introducerade
- Least privilege
- RBAC
- Servicekonto
- Secrets-hantering
- Image-policy
- Nätverkspolicy
- Governance
- Policy-as-code
- Audit

### Etablerade principer
- Säkerhet ska byggas in i plattformens golden paths.
- Least privilege ska gälla för användare, servicekonton, runtime, secrets och nätverk.
- Governance ska stödja självservice och dokumentera undantag.
- Policyer bör automatiseras där det är praktiskt, men ha tydlig undantagsprocess.
- Images, secrets och externa exponeringar är centrala kontrollpunkter.


## Uppdatering efter Kapitel 5

### Centrala begrepp introducerade
- Dataroll
- Primär sanning
- Delad databas
- Direkt schemaåtkomst
- Dataägarskap
- PostgreSQL som domänalternativ
- Datagovernance
- Schemahantering
- Restore-test

### Etablerade principer
- Databasbeslut ska börja med dataroll, inte produktnamn.
- Oracle kan behållas, kapslas in, kompletteras eller ersättas beroende på roll och risk.
- Nya tjänster ska inte direktläsa legacy-schema utan arkitekturbeslut.
- PostgreSQL ska användas med tydligt dataägarskap och driftmodell.
- Databas i OpenShift kräver backup, restore och stateful-driftmognad.


## Uppdatering efter Kapitel 6

### Centrala begrepp introducerade
- Message queue
- Command message
- Domain event
- Publish/subscribe
- Event streaming
- Integrationsmonolit
- Idempotens
- Retry
- Dead-letter queue
- Ordering

### Etablerade principer
- Messagingbeslut ska börja med mönsterklassificering, inte produktval.
- IBM MQ kan behållas, kapslas in, kompletteras eller ersättas beroende på flödets roll.
- Eventdriven arkitektur kräver tydliga kontrakt, idempotens och governance.
- Dead-letter queues behöver ägare och process.
- Event streaming ska införas där flera konsumenter, historik eller återspel ger värde.


## Uppdatering efter Kapitel 7

### Centrala begrepp introducerade
- Sekundärt index
- Reindexering
- Direkt indexering
- Eventdriven indexering
- Change Data Capture
- Batchindexering
- Indexägarskap
- Indexeringsfördröjning
- Observability-data jämfört med affärsdata

### Etablerade principer
- Elasticsearch ska normalt användas som sekundärt index, inte primär transaktionsdatabas.
- Index ska ha primär källa, ägare, retention och reindexeringsstrategi.
- Eventual consistency ska göras tydlig för verksamhet och användare.
- CDC kan vara ett övergångsmönster för legacy, men ska inte oreflekterat bli slutmål.
- Affärsdata och observability-data kan kräva olika Elasticsearch-modeller.


## Uppdatering efter Kapitel 8

### Centrala begrepp introducerade
- Stateful workload
- Stateless workload
- Blocklagring
- Fillagring
- Objektlagring
- Persistent volume
- Persistent volume claim
- Storage class
- Ceph
- RPO och RTO

### Etablerade principer
- Persistent volume är inte backup.
- Stateful workloads i OpenShift kräver backup, restore, monitoring och tydligt ägarskap.
- Storage classes ska dokumentera egenskaper, kostnad, prestanda och tillåtna workloads.
- Ceph ska behandlas som strategisk lagringsförmåga, inte som enkel disk.
- Delad fillagring ska vara kontrollerat övergångsmönster, inte ny integrationsplattform.


## Uppdatering efter Kapitel 12

### Centrala begrepp introducerade
- Containeriserad Java EE
- Anpassad Java-applikation
- Cloud native Java
- Serverbunden konfiguration
- Externaliserad konfiguration
- Health checks
- Graceful shutdown
- Runtime footprint
- Sessionstrategi

### Etablerade principer
- Java EE-applikationer ska klassificeras efter moderniseringsnivå.
- Containeriserad Java EE är ett möjligt övergångssteg, inte automatiskt slutmål.
- OpenShift-redo Java kräver externaliserad konfiguration, health checks, logging och shutdown.
- Sessioner, local state och serverbundna resurser måste synliggöras före produktion.


## Uppdatering efter Kapitel 13

### Centrala begrepp introducerade
- Lokal transaktion
- Distribuerad transaktion
- Eventual consistency
- Saga
- Kompenserande transaktion
- Orkestrering
- Koreografi
- Backpressure
- Distribuerad monolit

### Etablerade principer
- Distribuerade transaktioner ska vara medvetna undantag.
- Saga och eventual consistency kräver idempotens, statusmodell och observability.
- Dataägarskap måste definieras före uppdelning i tjänster.
- Orkestrering passar ofta för affärskritiska kärnflöden.
- Koreografi passar bättre för sekundära eller oberoende reaktioner.


## Uppdatering efter tillagd inledning

### Manusstruktur
- `chapters/00-inledning.md` ligger före Kapitel 1.
- Inledningen beskriver bokens syfte, målgrupp, läsanvisning, moderniseringscase, avgränsning, terminologi och grundantaganden.
- Författare är satt till Erland Lindmark i exportmetadata.

### Etablerade principer
- Boken presenteras som en arkitekturhandbok, inte installationsmanual.
- Nordisk Handel AB används som återkommande fiktivt moderniseringscase.
- Svenska används som huvudspråk med vedertagna engelska begrepp där det är lämpligt.
