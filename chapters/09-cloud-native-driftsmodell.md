# Kapitel 9: Cloud native-driftsmodell

## Varför detta kapitel finns

De tidigare kapitlen har behandlat containerteknik, OpenShift, säkerhet, data, messaging, Elasticsearch och lagring. Alla dessa tekniker påverkar hur applikationer byggs och körs. Men modernisering lyckas inte bara genom rätt teknikval. Den kräver också en driftsmodell som passar den nya arkitekturen.

I en traditionell Java EE-miljö har drift ofta varit centraliserad. Utveckling levererar artefakter, drift installerar, middleware-team hanterar applikationsserver, databasteam hanterar Oracle, integrationsteam hanterar IBM MQ och säkerhetsteam granskar i slutet. I en cloud native-miljö blir ansvaret mer distribuerat. Team behöver äga mer av sin applikations livscykel, samtidigt som plattformen måste erbjuda standarder, guardrails och stöd.

Det här kapitlet beskriver hur en cloud native-driftsmodell kan se ut för en Red Hat- och OpenShift-baserad moderniseringsresa.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- beskriva skillnaden mellan traditionell drift och cloud native-driftsmodell
- förklara ansvarsfördelning mellan applikationsteam, plattformsteam, drift/SRE, säkerhet och arkitektur
- formulera minimikrav för produktionsklara OpenShift-tjänster
- förstå rollen för runbooks, incidentprocesser, supportmodeller och ägarskap
- identifiera risker med otydligt ansvar i plattformsmodernisering
- skapa en första driftsmodell för moderniserade Java EE- och cloud native-tjänster

## Från central drift till delat ansvar

I traditionella miljöer är ansvaret ofta uppdelat efter tekniklager. Ett team äger servern, ett annat databasen, ett tredje integrationen och ett fjärde applikationskoden. Den modellen kan fungera när förändringstakten är låg och miljön är relativt statisk.

I OpenShift förändras detta. Applikationer deployas oftare, instanser är mer dynamiska, konfiguration externaliseras och team kan själva påverka runtime-beteende. Då behöver ansvaret flyttas närmare produkten.

Det betyder inte att utvecklingsteam ska äga allt själva. Det betyder att ansvaret behöver bli tydligare:

- applikationsteam äger kod, konfiguration, applikationsbeteende och runbook
- plattformsteam äger OpenShift, golden paths, guardrails och onboarding
- drift/SRE stödjer operativ modell, incidenter och tillförlitlighet
- säkerhetsteam definierar krav och granskar risker
- arkitekturforum styr större beslut och undantag
- databasteam och integrationsteam äger specialiserade plattformar där de fortfarande behövs

Cloud native-drift är alltså inte “utvecklarna gör allt”. Det är en tydligare modell för delat ansvar.

## Plattformsteamet som produktteam

OpenShift bör inte drivas som ett tekniskt sidoprojekt. Plattformsteamet bör fungera som ett produktteam med interna användare: applikationsteamen.

Det innebär att plattformsteamet behöver förstå sina användares behov och erbjuda:

- onboarding
- dokumentation
- golden paths
- supportkanaler
- standardiserade mallar
- CI/CD-integration
- secrets-modell
- logging- och metrics-standard
- image-policy
- namespace-modell
- utbildning
- incidentstöd
- tydlig roadmap

Plattformsteamet ska inte bara säga nej till felaktig användning. Det ska göra rätt användning enklare än fel användning.

## Applikationsteamets ansvar

När en applikation körs på OpenShift måste applikationsteamet äga mer än koden.

Applikationsteamet bör ansvara för:

- applikationskod
- Containerfile eller motsvarande buildmodell
- runtime-konfiguration
- health checks
- loggning
- metrics
- runbook
- beroendedokumentation
- felhantering
- versionshantering
- rollbackstrategi
- funktionsdrift
- första analys vid incidenter

Det betyder inte att teamet ensamt löser alla problem. Men teamet behöver förstå hur applikationen beter sig i drift.

## Produktionsklarhet

En applikation är inte produktionsklar bara för att den deployar. Produktionsklarhet bör vara en definierad standard.

Minimikrav:

| Område | Krav |
|---|---|
| Ägarskap | Team, kontaktväg och ansvarig produkt eller tjänst |
| Build | Reproducerbar image från godkänd bas |
| Deployment | Standardiserad deploymentmodell |
| Konfiguration | Externaliserad och dokumenterad |
| Secrets | Hanteras via godkänd mekanism |
| Health | Readiness och liveness där det är relevant |
| Logging | Loggar till standard output och central insamling |
| Metrics | Grundläggande tekniska och tjänstespecifika mätetal |
| Runbook | Dokumenterad felsökning och åtgärd |
| Beroenden | Databas, MQ, API:er och externa system dokumenterade |
| Rollback | Känd strategi för återställning |
| Säkerhet | Minsta privilegium och godkänd exponeringsmodell |

Produktionsklarhet bör vara en del av golden path, inte en manuell checklista som dyker upp i slutet.

## Runbooks

En runbook är en praktisk instruktion för hur en tjänst felsöks och hanteras i drift. Den behöver inte vara lång, men den måste vara användbar.

En runbook bör innehålla:

- vad tjänsten gör
- vem som äger tjänsten
- hur man ser om tjänsten fungerar
- vanliga fel och åtgärder
- viktiga dashboards
- relevanta loggfrågor
- beroenden
- hur man stoppar och startar om
- hur rollback görs
- hur köer, index eller batchjobb hanteras
- när ärendet ska eskaleras
- kontaktvägar

En runbook som ingen använder är dokumentation. En runbook som används vid incidenter är operativ förmåga.

## Incidenthantering

I en cloud native-miljö kan incidenter uppstå i samspelet mellan applikation, plattform, nätverk, data, messaging och externa beroenden. Incidentprocessen behöver därför stödja tvärfunktionell felsökning.

Viktiga frågor:

- vem tar första larmet?
- vem äger kundpåverkan?
- när kopplas plattformsteam in?
- när kopplas databasteam eller integrationsteam in?
- hur används korrelations-id?
- var finns runbook?
- hur dokumenteras tidslinje?
- hur görs post-incident review?
- hur förs lärdomar tillbaka till golden paths?

Incidenter ska inte bara lösas. De ska förbättra plattformen och arbetssättet.

## SRE-tänkande

SRE, Site Reliability Engineering, handlar om att behandla driftbarhet och tillförlitlighet som ingenjörsproblem. Organisationen behöver inte införa en full SRE-funktion direkt, men flera principer är värdefulla:

- definiera SLO:er för viktiga tjänster
- mät felbudget eller risknivå
- automatisera återkommande manuellt arbete
- prioritera tillförlitlighet där affärsvärdet kräver det
- göra incidentlärande systematiskt
- förbättra observability
- minska operativ toil

Toil är repetitivt manuellt arbete som inte skapar långsiktigt värde. En modern driftsmodell bör aktivt minska toil genom automation och bättre plattformsmönster.

## Supportmodell

När många team använder OpenShift behövs en tydlig supportmodell. Annars riskerar plattformsteamet att bli en informell helpdesk för allt.

Supportmodellen bör skilja mellan:

| Typ av fråga | Primärt ansvar |
|---|---|
| Applikationsfel | Applikationsteam |
| Plattformskluster | Plattformsteam |
| Bas-image eller buildmall | Plattformsteam i samverkan med säkerhet |
| Databasproblem | Databasteam eller datatjänstägare |
| MQ- eller brokerproblem | Integrationsteam eller plattformsägare |
| Nätverks- och routeproblem | Plattformsteam med nätverksstöd |
| Säkerhetsavvikelse | Säkerhetsteam med berörda ägare |
| Oklar rotorsak | Gemensam incidentprocess |

Supportmodellen ska vara dokumenterad innan bred onboarding.

## Ansvarsmatris

En enkel ansvarsmatris hjälper organisationen att undvika gråzoner.

| Område | Applikationsteam | Plattformsteam | Drift/SRE | Säkerhet | Arkitekturforum |
|---|---|---|---|---|---|
| Kod | Äger | Stödjer via standarder | Observerar påverkan | Granskar principer | Ej normalt |
| Image | Bygger enligt standard | Tillhandahåller bas | Följer driftpåverkan | Definierar krav | Godkänner avvikelser |
| Deployment | Äger manifest eller GitOps-konfiguration | Tillhandahåller mallar | Följer stabilitet | Granskar risk | Sätter mönster |
| Secrets | Konsumerar säkert | Tillhandahåller mekanism | Stödjer rotation | Sätter policy | Hanterar undantag |
| Observability | Instrumenterar | Samlar signaler | Använder vid drift | Definierar auditkrav | Sätter minimikrav |
| Incident | Deltar och äger appanalys | Deltar vid plattformsrisk | Leder eller stödjer | Deltar vid säkerhet | Följer lärdomar |
| Data | Äger tjänstens data | Stödjer plattform | Stödjer drift | Sätter dataskyddskrav | Beslutar större mönster |

Matrisen behöver anpassas lokalt, men principen är att varje område ska ha en tydlig primär ägare.

## GitOps och deklarativ drift

I OpenShift-miljöer används ofta deklarativa mönster där önskat tillstånd beskrivs i filer. GitOps innebär att Git blir källan för önskat tillstånd och att automation synkroniserar plattformen mot detta.

Fördelar:

- spårbarhet
- granskningsbarhet
- reproducerbarhet
- enklare rollback
- mindre manuell klickdrift
- tydligare koppling mellan kod och deployment

Risker:

- secrets hanteras fel i Git
- team förstår inte synkroniseringsmodellen
- akut felsökning görs direkt i klustret och skrivs över
- för många repositories skapar oöversiktlighet
- ansvar mellan app- och plattformsrepository blir otydligt

GitOps kan vara kraftfullt, men kräver utbildning och governance.

## Release och rollback

Cloud native-drift kräver säkra release- och rollbackmönster. En deployment ska inte vara en unik manuell händelse, utan en repeterbar process.

Viktiga frågor:

- hur versioneras image?
- hur kopplas image till kod och build?
- hur görs promotion mellan miljöer?
- hur hanteras databasändringar?
- hur rullas applikationen tillbaka?
- vad händer med meddelanden under rollback?
- påverkas Elasticsearch-index?
- finns feature flags?
- kan release göras stegvis?

Rollback är ofta svårare när data eller integrationer ändras. Därför måste rollbackstrategin utformas före produktion.

## Change management i ny form

Traditionell change management bygger ofta på manuell granskning före förändring. I en modernare modell flyttas mycket kontroll in i automation, policyer och standardiserade pipelines.

| Traditionell kontroll | Cloud native-kontroll |
|---|---|
| Manuell change review | Automatiserade policyer och godkända pipelines |
| Serverinstallation | Deklarativ deployment |
| Dokumenterad men manuell rollback | Versionerad och testad rollback |
| Sen säkerhetsgranskning | Image scanning och policy-as-code |
| Personberoende driftrutin | Runbook och golden path |
| Central kö för ändringar | Självservice inom guardrails |

Målet är snabbare och säkrare förändring, inte okontrollerad förändring.

## Mognadsnivåer för driftsmodell

En organisation behöver inte nå full cloud native-mognad direkt. Det är mer realistiskt att arbeta stegvis.

| Nivå | Kännetecken | Nästa steg |
|---|---|---|
| Start | Plattform finns, men arbetssätt är otydligt | Definiera ägarskap och produktionsklarhet |
| Grund | Golden path och runbookmall finns | Standardisera support och incidentprocess |
| Stabil | Team kan deploya inom guardrails | Inför SLO:er och bättre automatisering |
| Mogen | Observability, GitOps och policyer används konsekvent | Minska toil och förbättra kontinuerligt |

Denna mognadsmodell hjälper arkitekten att undvika allt-eller-inget-tänkande. Det viktiga är att varje steg stärker organisationens förmåga att drifta säkert och ändra snabbare.

## Beslutsguide: är tjänsten produktionsklar?

| Fråga | Om nej |
|---|---|
| Finns tydligt ägande team? | Tjänsten bör inte produktionssättas |
| Finns runbook? | Incidenthantering blir beroende av personer |
| Finns health checks? | Plattformen kan inte styra livscykel väl |
| Finns central loggning? | Felsökning blir svår |
| Finns grundläggande metrics? | Driftstatus blir oklar |
| Är secrets korrekt hanterade? | Säkerhetsrisk |
| Är beroenden dokumenterade? | Incidenter tar längre tid |
| Finns rollbackstrategi? | Release blir riskfylld |
| Finns supportmodell? | Ansvar blir otydligt |
| Är undantag dokumenterade? | Risk accepteras osynligt |

## Beslutsguide: vem ska äga vad?

| Situation | Primärt ägarskap |
|---|---|
| Applikationskod och fel i affärslogik | Applikationsteam |
| Plattformens klusterhälsa | Plattformsteam |
| Golden path för Java-tjänster | Plattformsteam med arkitektur |
| Produktionsincident med oklar rotorsak | Gemensam incidentprocess |
| Secrets-policy | Säkerhetsteam med plattformsteam |
| Databasbackup | Databasteam eller datatjänstägare |
| MQ-flödeskontrakt | Integrationsteam och berört applikationsteam |
| OpenShift-route och ingresspolicy | Plattformsteam med säkerhet |
| ADR för avvikande teknikval | Arkitekturforum |
| SLO för affärstjänst | Produkt-/applikationsteam med drift/SRE |

## Tradeoffs

| Val | Fördel | Risk |
|---|---|---|
| Mer ansvar hos applikationsteam | Snabbare lärande och bättre ägarskap | Kräver kompetens och stöd |
| Mer central drift | Tydlig specialisering | Flaskhals och mindre teamautonomi |
| Stark golden path | Enklare onboarding | Kan kännas begränsande |
| Fri plattformsanvändning | Flexibilitet | Spretighet och supportproblem |
| GitOps | Spårbar och deklarativ drift | Kräver mognad och disciplin |
| Manuell drift | Snabbt i enskilda fall | Svårt att skala och reproducera |
| Full SRE-modell direkt | Tydlig tillförlitlighetsstyrning | Kan vara för stort steg |
| Stegvis SRE-principer | Praktiskt införande | Risk för otydlig ambitionsnivå |

## Anti-patterns

### “You build it, you run it” utan stöd

Att säga att team äger drift utan att ge plattform, utbildning, runbooks, observability och support är inte empowerment. Det är ansvarsförskjutning.

### Plattformsteam som biljettkö

Om varje deployment, route eller secret kräver manuell handläggning blir OpenShift en ny flaskhals.

### Produktionsklarhet som muntlig bedömning

Om produktionsklarhet inte är definierad blir bedömningen personberoende och inkonsekvent.

### Runbook som aldrig testas

En runbook som inte används vid övning eller incident är ofta ofullständig.

### Otydligt ägarskap för delade komponenter

Databaser, brokers, index och operators kräver ägare. Annars blir incidenter svåra.

## Vanliga fallgropar

- Att införa OpenShift utan supportmodell.
- Att onboarda team utan golden paths.
- Att sakna tydlig ansvarsmatris.
- Att inte definiera produktionsklarhet.
- Att sakna runbooks för kritiska tjänster.
- Att inte öva incidenter.
- Att göra manuella ändringar direkt i klustret utan spårbarhet.
- Att inte koppla deployment till version och change.
- Att sakna rollbackstrategi för databasändringar.
- Att låta plattformsteamet äga applikationsproblem.

## Arkitektens checklista

Innan driftsmodellen godkänns bör arkitekten kunna svara på:

- Vilka team använder plattformen?
- Vad äger applikationsteamet?
- Vad äger plattformsteamet?
- Vad äger drift/SRE?
- Hur hanteras säkerhetsfrågor?
- Hur ser supportmodellen ut?
- Vad betyder produktionsklar?
- Finns runbookmall?
- Finns incidentprocess?
- Finns rollbackprincip?
- Hur används GitOps eller annan deklarativ modell?
- Hur hanteras manuella nödförändringar?
- Hur dokumenteras undantag?
- Hur förs incidentlärande tillbaka till plattformen?
- Hur mäts driftsmodellens mognad?

## Koppling till moderniseringscaset

Nordisk Handel AB inser att den tekniska plattformen inte räcker. Om teamen fortsätter arbeta enligt den gamla modellen kommer OpenShift bara bli en ny leveransyta med gamla väntetider.

Organisationen inför därför en första cloud native-driftsmodell:

- plattformsteamet äger OpenShift som produkt
- applikationsteam äger sina tjänsters produktionsbeteende
- varje tjänst behöver runbook
- pilotteam får stöd i golden path för Java-tjänst
- incidenter hanteras gemensamt över teamgränser
- produktionsklarhet definieras som standard
- ADR krävs för större avvikelser
- release och rollback ska vara spårbara

Detta gör att moderniseringen blir ett arbetssätt, inte bara en teknisk migrering.

## Snabb sammanfattning

- Cloud native-drift bygger på tydligt delat ansvar.
- Plattformsteamet bör fungera som produktteam.
- Applikationsteam behöver äga mer av tjänstens produktionsbeteende.
- Produktionsklarhet ska vara definierad och repeterbar.
- Runbooks, incidentprocesser och supportmodell är centrala.
- GitOps och deklarativ drift kan ge spårbarhet men kräver governance.
- Driftsmodellen är en del av arkitekturen.

## Nästa steg

Nästa kapitel behandlar disaster recovery, backup och kontinuitet. Där fördjupas frågan om hur organisationen återställer applikationer, data och plattform efter allvarliga fel.
