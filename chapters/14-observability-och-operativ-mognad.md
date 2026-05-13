# Kapitel 14: Observability och operativ mognad

## Varför detta kapitel finns

När en traditionell Java EE-miljö moderniseras mot OpenShift, distribuerade tjänster, eventflöden, sekundära index och flera datalager ökar behovet av insyn. I en servercentrerad miljö kunde felsökning ofta börja på en känd server, i en känd logg och i ett relativt statiskt flöde. I en cloud native-miljö är instanser kortlivade, trafik distribuerad och fel kan uppstå i samspelet mellan flera komponenter.

Observability är därför inte ett tillägg som kan införas sist. Det är en grundförutsättning för att kunna drifta, felsöka och vidareutveckla en moderniserad arkitektur.

Det här kapitlet beskriver hur loggar, metrics, tracing, SLI:er, SLO:er, runbooks och operativ mognad behöver hänga ihop för att OpenShift-baserade tjänster ska vara produktionsdugliga.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- förklara skillnaden mellan traditionell övervakning och observability
- beskriva rollerna för loggar, metrics och tracing
- förstå varför korrelations-id är centralt i distribuerade flöden
- formulera minimikrav för operativ mognad i OpenShift
- resonera om SLI, SLO och larm
- identifiera vanliga anti-patterns kring loggning, larm och felsökning

## Från övervakning till observability

Traditionell övervakning svarar ofta på frågan: “är systemet uppe?” Observability försöker svara på en bredare fråga: “varför beter sig systemet som det gör?”

I en moderniserad miljö räcker det inte att veta att en podd kör eller att en server svarar. Arkitekten behöver kunna förstå vilka tjänster som ingick i ett affärsflöde, var tid spenderades, vilket meddelande som fastnade, vilken konsument som halkar efter, vilken databasfråga som dominerar svarstiden, om sökindexet ligger efter primär källa, om en release ökade felprocenten och om användarupplevelsen påverkas.

Observability handlar om att bygga system som går att förstå i drift, även när felet inte är känt i förväg.

## De tre grundsignalerna

| Signal | Syfte | Exempel |
|---|---|---|
| Loggar | Beskriver händelser och fel | Start, fel, affärshändelser, varningar |
| Metrics | Mätvärden över tid | Svarstid, felprocent, ködjup, CPU, minne |
| Tracing | Visar flöden över tjänstegränser | Ett orderflöde genom API, tjänster, MQ och databas |

De tre signalerna ersätter inte varandra. Loggar hjälper vid detaljerad felsökning. Metrics visar trender och larm. Tracing visar sammanhanget mellan komponenter.

## Loggar

Loggar är ofta första platsen utvecklare och drift tittar på vid problem. I OpenShift bör applikationer skriva loggar till standard output så att plattformen kan samla in dem.

Bra loggar har tydlig tidsstämpel, loggnivå, tjänstnamn, miljö, version, korrelations-id, relevant teknisk kontext, relevant affärskontext utan känsliga uppgifter, felmeddelande med tillräcklig detalj, inga secrets och inga onödiga personuppgifter.

Dåliga loggar är antingen för tysta eller för pratiga. För lite loggning gör felsökning svår. För mycket loggning skapar kostnad, brus och compliance-risk.

## Metrics

Metrics används för att förstå systemets hälsa över tid. De lämpar sig för dashboards, larm och trendanalys.

Viktiga metrics i moderniserade enterprise-miljöer:

- request rate
- latency
- error rate
- saturation
- CPU och minne
- ködjup
- consumer lag
- antal misslyckade meddelanden
- databasanslutningar
- indexeringsfördröjning
- antal retries
- antal dead-letter-meddelanden
- deploymentfrekvens
- rollbackfrekvens

För arkitekten är det viktigt att metrics kopplas till affärsflöden, inte bara infrastruktur. CPU kan vara relevant, men säger inte ensam om kunder kan lägga order.

## Tracing

Tracing visar hur ett anrop eller flöde rör sig genom flera komponenter. I en distribuerad arkitektur är detta ofta avgörande.

Ett orderflöde kan exempelvis passera API-gateway eller route, ordertjänst, databas, betalningstjänst, MQ eller event stream, lagerkonsument, Elasticsearch-indexering och notifieringstjänst.

Utan tracing kan varje komponent se frisk ut isolerat, samtidigt som helhetsflödet är långsamt eller felaktigt. Tracing gör det möjligt att se var flödet bromsar, var fel uppstår och vilka beroenden som är inblandade.

## Korrelations-id

Korrelations-id är ett gemensamt id som följer ett flöde genom flera system. Det gör det möjligt att koppla ihop loggar, meddelanden och traces.

Principer:

- skapas tidigt i flödet
- skickas vidare i HTTP-headers, meddelanden och events
- loggas av alla komponenter
- används i felsökning och support
- ska inte innehålla känslig information
- ska vara stabilt genom flödet

Utan korrelations-id blir felsökning i distribuerade system ofta manuell och tidskrävande.

## SLI, SLO och larm

Service Level Indicator, SLI, är ett mätetal för tjänstens beteende. Service Level Objective, SLO, är målnivån för mätetalet.

| SLI | SLO |
|---|---|
| Andel lyckade orderförfrågningar | 99,9 % per månad |
| Svarstid för ordersök | 95 % under 500 ms |
| Indexeringsfördröjning | 99 % under 60 sekunder |
| Consumer lag | Under definierad tröskel |
| Fel i betalningsflöde | Under definierad felbudget |

Larm bör kopplas till användarpåverkan eller risk för användarpåverkan. Larm på varje teknisk detalj skapar larmtrötthet.

## Operativ mognad i OpenShift

En OpenShift-applikation bör inte betraktas som produktionsklar bara för att den deployar. Den behöver uppfylla operativa minimikrav:

1. Loggar till standard output.
2. Korrelations-id i relevanta flöden.
3. Readiness och liveness där det är relevant.
4. Grundläggande metrics.
5. Dashboard för tjänstens hälsa.
6. Larm för användarpåverkande fel.
7. Dokumenterad runbook.
8. Ägare och kontaktväg.
9. Känd rollback-strategi.
10. Dokumenterade beroenden.
11. Spårbar releaseversion.
12. Felhantering för köer och asynkrona flöden.

Detta är inte “extra driftarbete”. Det är vad som gör tjänsten möjlig att äga över tid.

## Observability för stateful och asynkrona komponenter

För databaser, köer, event streaming, Elasticsearch och Ceph behövs särskilda signaler.

| Komponent | Viktiga signaler |
|---|---|
| Databas | anslutningar, lås, långsamma frågor, replikering, backupstatus |
| MQ/broker | ködjup, felköer, consumer rate, retry, dead letters |
| Event streaming | consumer lag, topic-tillväxt, felaktiga events |
| Elasticsearch | indexeringsfördröjning, query latency, shardhälsa, lagringsnivå |
| Ceph/lagring | kapacitet, latency, fel, rebalansering, diskhälsa |
| OpenShift | poddrestarts, resursbrist, schedulingproblem, deploymentstatus |

Detta visar varför observability måste täcka både applikation och plattform.

## Observability för moderniseringsprogrammet

Observability gäller inte bara enskilda tjänster. Moderniseringsprogrammet behöver också mätas. Annars blir det svårt att veta om arbetet faktiskt förbättrar förändringsförmågan.

Exempel på programnivå-metrics:

- antal tjänster med runbook
- antal tjänster med korrelations-id
- antal tjänster med definierade SLO:er
- antal incidenter där rotorsak hittades inom acceptabel tid
- andel flöden med dokumenterade beroenden
- antal oägda larm
- antal köer utan dead-letter-process
- antal produktionssatta tjänster utan rollbackstrategi
- antal legacy-flöden som saknar observability
- tid från incidentstart till första tekniska hypotes

Dessa mätetal ska inte användas för att skapa rapporteringsritualer. De ska visa var organisationen behöver stärka driftbarhet och ansvar.

## Dashboards

Dashboards är användbara när de hjälper team att fatta beslut. De är mindre användbara när de bara visar många grafer.

En bra dashboard bör svara på:

- fungerar tjänsten just nu?
- påverkas användare?
- är felet i applikation, beroende eller plattform?
- har något förändrats efter senaste release?
- finns risk att kapaciteten tar slut?
- finns köer eller index som halkar efter?
- vilka länkar går till runbook och loggar?

Dashboards bör byggas runt tjänster och affärsflöden, inte bara runt tekniska komponenter.

## Larm

Ett larm ska leda till handling. Om ingen vet vad som ska göras när larmet går är larmet inte färdigt.

Ett bra larm har tydlig ägare, tydlig påverkan, tröskel som betyder något, länk till runbook, rimlig prioritet, möjlighet att tysta vid planerat arbete, koppling till SLO eller risk och låg andel falsklarm.

Larmtrötthet uppstår när många larm inte kräver åtgärd. Då riskerar teamen att missa de viktiga larmen.

## Beslutsguide: vad ska observeras?

| Scenario | Prioriterad observability |
|---|---|
| Synkront API | latency, error rate, request rate, tracing |
| Asynkront MQ-flöde | ködjup, dead letters, retry, korrelations-id |
| Event streaming | consumer lag, eventfel, schemafel |
| Sökindex | indexeringsfördröjning, query latency, reindexeringsstatus |
| Databas | långsamma frågor, anslutningar, lås, backup |
| Batchjobb | start, sluttid, duration, fel, antal bearbetade poster |
| Stateful workload | lagring, backup, restore, replikering |
| Affärskritiskt flöde | end-to-end SLI, tracing och larm |

## Beslutsguide: när är observability tillräcklig?

| Fråga | Om nej |
|---|---|
| Kan teamet se om tjänsten fungerar? | Mer grundläggande metrics behövs |
| Kan teamet se om användare påverkas? | Affärsnära SLI saknas |
| Kan ett flöde följas över tjänster? | Korrelations-id eller tracing saknas |
| Kan asynkrona fel hittas? | Kö-, lag- eller dead-letter-metrics saknas |
| Finns runbook kopplad till larm? | Larmet är inte operativt färdigt |
| Kan releaseeffekt ses? | Versions- och deploymentmetadata saknas |
| Kan datafördröjning ses? | Indexerings- eller replikeringsmetrics saknas |
| Vet någon vem som agerar? | Ägarskap saknas |

## Tradeoffs

| Val | Fördel | Kostnad eller risk |
|---|---|---|
| Mycket detaljerad loggning | Bra felsökning | Hög kostnad och brus |
| Minimal loggning | Lägre kostnad | Svår felsökning |
| Många larm | Fångar fler tekniska avvikelser | Larmtrötthet |
| Få affärsnära larm | Mer relevanta larm | Risk att tekniska problem upptäcks sent |
| Full tracing överallt | Bra insyn | Kostnad och overhead |
| Tracing på kritiska flöden | Bra balans | Mindre täckning |
| Central standard | Jämförbarhet och driftbarhet | Kan kännas begränsande för team |
| Teamunika lösningar | Flexibilitet | Spretighet och svår support |

## Anti-patterns

### Logging som enda felsökningsverktyg

Loggar är viktiga, men räcker inte för distribuerade flöden. Metrics och tracing behövs för sammanhang och trender.

### Larm utan åtgärd

Ett larm som ingen vet hur man agerar på skapar stress men inte bättre drift.

### Dashboard-teater

Många dashboards betyder inte hög operativ mognad. De måste stödja beslut och felsökning.

### Observability efter produktion

Om observability läggs till efter incidenter blir det dyrare och mindre konsekvent.

### Ingen koppling till affärsflöden

Tekniska metrics utan koppling till affärspåverkan gör det svårt att prioritera.

## Vanliga fallgropar

- Att logga secrets eller personuppgifter.
- Att sakna korrelations-id.
- Att mäta infrastruktur men inte affärsflöden.
- Att inte mäta ködjup och consumer lag.
- Att sakna larm för indexeringsfördröjning.
- Att inte följa upp backup- och restore-status.
- Att sakna runbooks.
- Att inte veta vem som äger ett larm.
- Att skapa för många dashboards.
- Att inte testa observability vid incidentövningar.

## Arkitektens checklista

Innan en tjänst betraktas som produktionsklar bör arkitekten kunna svara på:

- Vem äger tjänsten?
- Finns runbook?
- Vilka SLI:er används?
- Finns SLO?
- Vilka larm finns?
- Vem agerar på larmen?
- Finns korrelations-id?
- Finns strukturerad loggning?
- Finns metrics?
- Behövs tracing?
- Hur observeras beroenden?
- Hur observeras asynkrona flöden?
- Hur mäts användarpåverkan?
- Hur följs deployment och rollback upp?
- Har observability testats i incidentövning?

## Koppling till moderniseringscaset

Nordisk Handel AB upptäcker att modernisering utan observability skapar blindhet. Den nya ordersöktjänsten fungerar tekniskt, men när sökresultat är fördröjda behöver teamet kunna se om orsaken är CDC, eventflöde, indexering eller Elasticsearch-frågor.

Organisationen inför därför end-to-end-observability för pilotflödet. Korrelations-id följer från API till event, indexering och sök. Metrics visar indexeringsfördröjning och query latency. Larm skapas bara där det finns tydlig åtgärd.

Detta gör att moderniseringsprogrammet kan fortsätta med större förtroende.

## Snabb sammanfattning

- Observability handlar om att förstå varför system beter sig som de gör.
- Loggar, metrics och tracing behövs tillsammans.
- Korrelations-id är centralt i distribuerade flöden.
- SLI och SLO kopplar teknik till användarpåverkan.
- Larm ska vara åtgärdsbara.
- Stateful och asynkrona komponenter kräver särskilda signaler.
- En tjänst är inte produktionsklar bara för att den deployar.

## Nästa steg

Nästa kapitel behandlar beslutsmodeller för teknikval. Där samlar vi bokens resonemang i en metod för ADR:er, tradeoffs och styrning av teknikval över tid.
