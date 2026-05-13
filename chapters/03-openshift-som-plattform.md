# Kapitel 3: OpenShift som plattform

## Varför detta kapitel finns

De tidigare kapitlen etablerade varför cloud native-modernisering behövs och hur containerteknik med Podman och OCI gör applikationens paketering mer explicit. Nu flyttas fokus från enskilda containers till plattformen där de ska köras, styras och förvaltas.

OpenShift är inte bara en plats där containers startas. I en enterprise-miljö bör OpenShift betraktas som en intern applikationsplattform: en produkt som ger utvecklingsteam standardiserade sätt att bygga, deploya, exponera, säkra och observera applikationer. Plattformen ska göra rätt väg enkel, samtidigt som den skyddar organisationen från spretighet, säkerhetsrisker och ohållbar drift.

För lösningsarkitekter är OpenShift därför en arkitekturfråga lika mycket som en driftfråga. Hur namespaces modelleras, hur deployment sker, hur routes exponeras, hur secrets hanteras och hur operators används påverkar både teamens autonomi och organisationens kontroll.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- beskriva OpenShift som intern enterprise-plattform snarare än enbart Kubernetes-kluster
- förklara rollen för project, namespace, deployment, service, route, ConfigMap, Secret och operator
- resonera om plattformsguardrails och självservice
- identifiera vilka plattformsförmågor som bör finnas innan bred onboarding
- bedöma när en workload är rimlig att flytta från lokal containerverifiering till OpenShift
- formulera en referensmodell för applikationer på OpenShift

## Från container till plattform

En container som fungerar lokalt med Podman är bara första steget. När den ska köras på OpenShift tillkommer frågor som inte syns fullt ut lokalt:

- vilken namespace-struktur ska användas?
- vilka rättigheter behöver teamet?
- hur tillförs konfiguration?
- hur hanteras secrets?
- hur exponeras tjänsten internt eller externt?
- vilka resource requests och limits gäller?
- hur sker rolling update?
- vilka health checks används?
- hur samlas loggar och metrics?
- vilka nätverkspolicyer gäller?
- vem ansvarar för drift och incidenter?

Det är här skillnaden mellan containerteknik och plattformsarkitektur blir tydlig. OpenShift ger mekanismerna, men organisationen måste bestämma standarderna.

## OpenShift som intern produkt

Ett vanligt misstag är att införa OpenShift som ett tekniskt installationsprojekt. Klustret installeras, några exempelapplikationer körs och sedan förväntas utvecklingsteam börja använda plattformen. Resultatet blir ofta att varje team skapar egna mönster.

En bättre modell är att behandla OpenShift som en intern produkt.

Det innebär att plattformen behöver:

- tydlig målgrupp
- onboardingprocess
- dokumenterade golden paths
- supportmodell
- standardiserade bas-images
- rekommenderade deploymentmönster
- tydliga säkerhetskrav
- standard för secrets och konfiguration
- observability från start
- undantagsprocess
- kapacitets- och kostnadsmodell

Teamen ska inte behöva uppfinna plattformen själva. De ska kunna använda den genom tydliga, säkra och förvaltade mönster.

## Centrala OpenShift-begrepp

### Project och namespace

OpenShift project är en användarvänlig representation av namespace. Ett namespace är en logisk avgränsning för resurser, åtkomst, policyer och ansvar.

Namespace-modellen är ett tidigt arkitekturbeslut. Den påverkar säkerhet, kostnad, livscykel och teamägarskap.

Vanliga modeller:

| Modell | Passar när | Risk |
|---|---|---|
| Namespace per team och miljö | Team äger flera tjänster | Många namespaces att hantera |
| Namespace per applikation och miljö | Tydlig applikationslivscykel | Administrativ overhead |
| Namespace per domän | Domänteam äger flera tjänster | Kräver mogen domänmodell |
| Delade namespaces | Tidig pilot eller små miljöer | Otydligt ansvar och svag isolering |

För Nordisk Handel AB är namespace-strukturen viktig eftersom organisationen har flera team, flera miljöer och affärskritiska system. En enkel pilot kan börja med få namespaces, men målmodellen bör stödja tydligt ägarskap.

### Deployment

Deployment beskriver önskat tillstånd för en applikation: vilken image som ska köras, hur många instanser som ska finnas, vilka resurser som behövs och hur uppdateringar ska ske.

För arkitekten är deployment ett uttryck för driftmodell. Det bör innehålla eller kopplas till:

- image-version
- antal repliker
- resource requests och limits
- readiness och liveness
- konfiguration
- secrets
- uppdateringsstrategi
- rollbackmöjlighet
- labels och metadata
- ägarskap

En deployment utan health checks, resource limits och ägare är inte en färdig produktionsmodell.

### Service

En service ger en stabil intern åtkomstpunkt till en eller flera poddar. Eftersom poddar kan skapas och försvinna behövs en stabil abstraktion för intern kommunikation.

Arkitektoniskt är service-begreppet viktigt eftersom det påverkar hur tjänster hittar varandra och hur kopplingar etableras. Service-namn och portar bör inte uppstå slumpmässigt. De är en del av applikationens interna kontrakt.

### Route

OpenShift routes används för att exponera HTTP-baserade tjänster. En route kan vara intern, extern eller kopplad till en kontrollerad exponeringsmodell.

Frågor att ställa:

- ska tjänsten vara intern eller extern?
- var termineras TLS?
- krävs API-gateway?
- behövs autentisering vid kanten?
- vilka domännamn används?
- hur hanteras certifikat?
- vilka loggar behövs för inkommande trafik?

Alla tjänster behöver inte exponeras. Ett vanligt misstag är att göra tekniska tjänster åtkomliga externt bara för att det är enkelt.

### ConfigMap och Secret

ConfigMaps används för icke-hemlig konfiguration. Secrets används för känslig information som lösenord, tokens, certifikat och nycklar.

Principen bör vara:

- bygg inte in miljöspecifik konfiguration i imagen
- bygg aldrig in secrets i imagen
- håll konfiguration och kod separerade
- gör skillnad mellan vanlig konfiguration och hemligheter
- dokumentera vilka värden applikationen kräver
- undvik att secrets råkar loggas

För Java EE-applikationer innebär detta ofta att tidigare serverkonfiguration, som datasources och MQ-anslutningar, behöver göras explicit.

### Operator

En operator automatiserar livscykeln för en komplex komponent. Operators kan vara användbara för databaser, messaging, observability, certifikathantering eller annan plattformsfunktionalitet.

Men operator är inte magi. Den ersätter inte ägarskap.

Innan en operator införs bör organisationen veta:

- vem äger operatorn?
- vem uppgraderar den?
- vilka rättigheter kräver den?
- hur hanteras backup?
- hur övervakas komponenten?
- hur testas uppgraderingar?
- vad händer om operatorn fallerar?
- vilken supportmodell gäller?

I en enterprise-miljö bör operators behandlas som plattformsbeslut, inte som enskilda teaminstallationer.

## Guardrails och självservice

OpenShift skapar värde när team kan leverera mer självständigt. Men självservice utan ramar leder till spretighet. Guardrails är styrande ramar som gör rätt väg enkel och fel väg svårare.

Exempel på guardrails:

- godkända bas-images
- krav på non-root-körning
- image scanning
- standardiserade deploymentmallar
- resource requests och limits
- readiness och liveness som minimikrav
- secrets via godkänd mekanism
- nätverkspolicyer
- central logginsamling
- krav på ägare och runbook
- undantag via ADR

Bra guardrails ersätter inte arkitekttänkande. De gör att vanliga beslut kan fattas snabbare och säkrare.

## Golden paths

En golden path är en rekommenderad och stödd väg för ett vanligt scenario. I OpenShift-sammanhang kan det vara en standardiserad väg för att deploya en Java-tjänst med logging, health checks, secrets, CI/CD och observability.

Exempel på golden paths:

- stateless Java API
- Java EE-applikation under övergång
- batchjobb
- intern worker som konsumerar MQ
- söktjänst med Elasticsearch
- tjänst med PostgreSQL
- extern HTTP-exponerad tjänst

Varje golden path bör beskriva:

- när den ska användas
- vilka komponenter den innehåller
- vilka krav som gäller
- vilka undantag som kräver ADR
- vem som ger support
- hur driftbarhet verifieras

Golden paths gör plattformen användbar, inte bara tekniskt möjlig.

## Från lokal verifiering till OpenShift-readiness

En applikation som passerar lokal Podman-verifiering är inte automatiskt redo för OpenShift. Nästa steg är att verifiera att den passar plattformens krav.

OpenShift-readiness bör omfatta:

| Område | Frågor |
|---|---|
| Image | Är imagen godkänd, skannad och spårbar? |
| Security | Kör processen non-root och med rimliga rättigheter? |
| Konfiguration | Är miljöspecifik konfiguration externaliserad? |
| Secrets | Hanteras credentials via godkänd mekanism? |
| Health | Finns readiness och liveness? |
| Resurser | Finns requests och limits? |
| Logging | Går loggar till central insamling? |
| Nätverk | Är intern och extern åtkomst definierad? |
| State | Finns persistent behov dokumenterat? |
| Ägarskap | Finns team, runbook och supportväg? |

Detta bör vara en standardiserad check, inte en ad hoc-granskning varje gång.

## OpenShift och Java EE

Java EE-applikationer kan köras på OpenShift på flera sätt. Det finns inte ett enda rätt svar.

| Strategi | Beskrivning | Passar när |
|---|---|---|
| Containeriserad befintlig runtime | Applikationsserver paketeras i image | Snabb övergång och komplex legacy |
| Anpassad Java-runtime | Applikationen anpassas för lättare driftmodell | Applikationen ska leva vidare länge |
| Ny cloud native-tjänst | Funktion byggs om eller nyutvecklas | Tydlig domän och affärsnytta |

OpenShift kan stödja alla dessa strategier, men de ska inte blandas ihop. En containeriserad traditionell runtime kan vara ett steg på vägen, men bör ha en uttalad livscykel och moderniseringsplan.

## Stateful workloads på OpenShift

OpenShift kan köra stateful workloads, men det kräver särskild försiktighet. Databaser, brokers, Elasticsearch och lagringskomponenter har andra krav än stateless API:er.

Innan stateful workload körs bör följande finnas:

- backup och restore
- testad återställning
- kapacitetsplanering
- storage class med tydlig egenskap
- övervakning
- uppgraderingsstrategi
- incidentrutiner
- ägarskap
- ADR

Frågan är inte “kan detta köras på OpenShift?”. Frågan är “kan vi drifta detta ansvarsfullt på OpenShift?”.

## Beslutsguide: när passar OpenShift?

| Scenario | OpenShift passar väl när | Var försiktig när |
|---|---|---|
| Flera team behöver gemensam plattform | Plattformsteam och guardrails finns | Plattformen saknar produktägarskap |
| Applikationer behöver snabbare deployment | CI/CD och image-policy finns | Releaseprocessen är fortfarande manuell |
| Nya tjänster byggs cloud native | Team kan följa golden paths | Varje team vill uppfinna egen modell |
| Java EE ska moderniseras stegvis | Runtime och konfiguration kan göras explicita | Applikationen är starkt serverbunden |
| Stateful workloads behövs | Backup, restore och driftmodell finns | PVC betraktas som tillräckligt skydd |
| Legacy-integration krävs | Nätverk och secrets är standardiserade | Specialundantag blir normalväg |

## Beslutsguide: vad ska standardiseras först?

| Område | Bör standardiseras tidigt? | Varför |
|---|---|---|
| Namespace-modell | Ja | Påverkar ansvar, säkerhet och kostnad |
| Bas-images | Ja | Påverkar säkerhet och patchning |
| Secrets-hantering | Ja | Svår att ändra i efterhand |
| Logging | Ja | Behövs för felsökning från start |
| Health checks | Ja | Behövs för plattformens livscykelstyrning |
| Resource requests/limits | Ja | Skyddar plattformens kapacitet |
| Stateful workloads | Ja, som beslutsprocess | Hög risk utan driftmodell |
| Service mesh | Inte nödvändigtvis | Bör införas utifrån tydliga behov |
| Full eventplattform | Inte nödvändigtvis | Bör växa från användningsfall |
| Avancerad policyautomation | Senare | Kräver först fungerande standarder |

## Tradeoffs

| Val | Fördel | Risk |
|---|---|---|
| Hård standardisering | Enklare drift och säkerhet | Mindre flexibilitet |
| Stor teamfrihet | Snabb innovation | Spretighet och högre driftkostnad |
| Tidig bred onboarding | Snabb adoption | Plattformen kan bli belastad innan den är mogen |
| Begränsad pilot | Kontrollerat lärande | Risk att nyttan upplevs långsam |
| Många golden paths | Täcker fler behov | Mer underhåll |
| Få golden paths | Enklare support | Team kan sakna passande mönster |
| Operators för mycket | Automation | Otydligt ägarskap om de införs okontrollerat |
| Operators restriktivt | Bättre styrning | Mindre självservice för komplexa tjänster |

## Anti-patterns

### OpenShift som ny serverhall

Om OpenShift används för att återskapa gamla servermönster med nya verktyg får organisationen inte ut plattformens värde. Resultatet blir ofta högre komplexitet.

### Namespace utan ägare

Namespaces utan tydligt team, kostnadsansvar och livscykel blir snabbt oöverskådliga.

### Självservice utan support

Att ge team möjlighet att deploya utan dokumentation, golden paths och support skapar frustration snarare än autonomi.

### Operator som genväg

En operator kan automatisera mycket, men den ersätter inte förståelse, backup, restore, uppgraderingsplan och incidentansvar.

### Allt ska in i OpenShift

OpenShift är en viktig målplattform, men allt behöver inte köras där. Vissa databaser, legacy-komponenter eller externa tjänster kan vara bättre placerade utanför.

## Vanliga fallgropar

- Att börja onboarda många team innan secrets, logging och RBAC är standardiserade.
- Att sakna tydlig namespace-modell.
- Att låta varje team välja egen bas-image.
- Att inte definiera resource requests och limits.
- Att underskatta kostnaden för plattformsteamets produktansvar.
- Att blanda test- och produktionsmönster.
- Att inte ha process för undantag.
- Att införa stateful workloads utan restore-test.
- Att sakna gemensam definition av OpenShift-ready.
- Att inte mäta plattformens användbarhet.

## Arkitektens checklista

Innan OpenShift används brett bör arkitekten kunna svara på:

- Vem äger plattformen som produkt?
- Vilka team är första målgrupp?
- Vilken namespace-modell används?
- Vilka bas-images är godkända?
- Hur byggs och skannas images?
- Hur deployas applikationer?
- Hur hanteras ConfigMaps och Secrets?
- Vilka minimikrav gäller för logging?
- Vilka health checks krävs?
- Vilka resource requests och limits gäller?
- Hur exponeras tjänster internt och externt?
- Hur hanteras nätverkspolicyer?
- Vilka operators är godkända?
- Hur beslutas stateful workloads?
- Hur dokumenteras undantag?
- Finns golden paths?
- Finns onboarding och supportmodell?

## Koppling till moderniseringscaset

Nordisk Handel AB beslutar att OpenShift ska införas som en intern plattformsprodukt. Den första versionen riktas inte till alla team samtidigt, utan till några utvalda pilotteam.

Plattformsteamet definierar:

- namespace-modell för team och miljö
- godkända bas-images
- krav på non-root-körning
- standard för ConfigMaps och Secrets
- minimikrav för readiness och liveness
- central logginsamling
- grundläggande resource requests och limits
- onboardingguide
- ADR-process för avvikelser

Den första Java EE-baserade stödapplikationen som verifierades med Podman går vidare till OpenShift-test. Teamet upptäcker att lokal containerverifiering var nyttig, men att OpenShift synliggör nya krav: routes, services, security context, resource limits och central loggning.

Det bekräftar en viktig princip: Podman visar om containern är rimlig. OpenShift visar om den passar i organisationens plattformsmodell.

## Snabb sammanfattning

- OpenShift bör betraktas som intern enterprise-plattform, inte bara kluster.
- Plattformen behöver produktägarskap, onboarding, support och golden paths.
- Project, namespace, deployment, service, route, ConfigMap, Secret och operator är arkitekturbyggblock.
- Självservice kräver guardrails.
- En applikation som fungerar lokalt är inte automatiskt OpenShift-ready.
- Stateful workloads kräver särskild driftmodell.
- OpenShift skapar värde först när tekniken kombineras med tydlig ansvarsfördelning.

## Nästa steg

Nästa kapitel behandlar säkerhet och governance i Red Hat-stacken. Där går vi djupare in i least privilege, RBAC, secrets, nätverkspolicyer, image-policy och hur organisationen styr plattformen utan att skapa nya flaskhalsar.
