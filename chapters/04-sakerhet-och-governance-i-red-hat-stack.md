# Kapitel 4: Säkerhet och governance i Red Hat-stack

## Varför detta kapitel finns

När en organisation går från traditionell Java EE-drift till OpenShift förändras säkerhetsmodellen. I den äldre miljön låg mycket kontroll ofta i nätverkszoner, brandväggar, serverhärdning, centrala driftprocesser och manuella change-flöden. I en containerplattform blir miljön mer dynamisk. Workloads startas och stoppas, team får mer självservice, konfiguration externaliseras och säkerhetsbeslut behöver ligga närmare applikationens livscykel.

Det betyder inte att säkerhet blir mindre viktig. Tvärtom blir den mer distribuerad. Säkerhet måste byggas in i plattformens standardmönster, image-flöden, RBAC, secrets-hantering, nätverkspolicyer och governance-processer.

Det här kapitlet fokuserar på hur säkerhet och styrning bör utformas i en Red Hat-orienterad stack. Målet är att skapa skydd och kontroll utan att återskapa de gamla flaskhalsarna.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- beskriva hur säkerhetsmodellen förändras i en containerplattform
- förklara principen least privilege i OpenShift-sammanhang
- identifiera centrala beslut kring RBAC, secrets, image-policy och nätverkspolicyer
- förstå hur governance kan stödja självservice utan att skapa anarki
- formulera grundläggande säkerhetskrav för OpenShift-workloads
- känna igen vanliga anti-patterns vid säkerhetsstyrning i moderniseringsprojekt

## Från perimeter till policy

I traditionella enterprise-miljöer har säkerhet ofta byggts kring perimeter. System placeras i zoner, trafik öppnas i brandväggar, servrar härdas och åtkomst kontrolleras via centrala processer. Den modellen försvinner inte helt, men den räcker inte ensam i OpenShift.

I en containerplattform behöver säkerhet uttryckas på flera nivåer:

- vem får skapa och ändra resurser?
- vilka images får köras?
- vilka rättigheter har containern vid runtime?
- vilka secrets får workloaden läsa?
- vilka namespaces får kommunicera?
- vilka tjänster får exponeras externt?
- vilka operators får installeras?
- vilka policyer gäller för loggning och audit?
- hur dokumenteras undantag?

Säkerheten flyttas alltså från att vara enbart omgivande skydd till att också bli policyer och guardrails nära workloaden.

## Least privilege

Least privilege betyder att varje användare, process och komponent bara ska ha den åtkomst som krävs för uppgiften. I OpenShift bör principen tillämpas på både människor, servicekonton, containers, nätverk och plattformskomponenter.

| Område | Exempel på least privilege-fråga |
|---|---|
| Användare | Behöver utvecklaren deploya till produktion eller bara till test? |
| Servicekonto | Behöver workloaden läsa Kubernetes API eller bara köra applikationen? |
| Container | Behöver processen root eller kan den köras non-root? |
| Secrets | Behöver tjänsten alla credentials eller bara sin egen anslutning? |
| Nätverk | Behöver namespace A prata med namespace B? |
| Image-policy | Får teamet köra valfri image eller bara godkända images? |
| Operators | Vem får installera komponenter med höga klusterrättigheter? |

Least privilege ska inte hanteras som en manuell granskning för varje detalj. Det ska byggas in i plattformens standarder.

## RBAC och ansvar

RBAC, role-based access control, styr vad användare och servicekonton får göra. I en enterprise-miljö är RBAC inte bara teknisk konfiguration. Den speglar organisationens ansvarsfördelning.

Frågor att besluta:

- vem får skapa namespaces?
- vem får deploya till test?
- vem får deploya till produktion?
- vem får läsa produktionsloggar?
- vem får läsa secrets?
- vem får ändra routes?
- vem får installera operators?
- vem får ändra resource quotas?
- vem får felsöka direkt i poddar?

Om dessa beslut är otydliga uppstår två risker. Antingen får team för breda rättigheter, vilket skapar säkerhetsrisk. Eller så blir varje ändring beroende av central handpåläggning, vilket återskapar gamla flaskhalsar.

## Servicekonton

En workload i OpenShift kör ofta med ett servicekonto. Servicekontot representerar workloadens identitet mot plattformen. Det ska inte ges mer rättigheter än vad workloaden behöver.

Vanliga fel:

- alla workloads använder default service account
- servicekonton får breda rättigheter “för att det ska fungera”
- samma servicekonto används av flera applikationer
- servicekontots rättigheter dokumenteras inte
- rättigheter tas inte bort när behov försvinner

För de flesta applikationer behövs få eller inga Kubernetes API-rättigheter. Om en applikation behöver mer bör det vara ett medvetet beslut.

## Secrets-hantering

Secrets är ett av de vanligaste riskområdena i containeriserade miljöer. När konfiguration externaliseras måste credentials, tokens, certifikat och nycklar hanteras kontrollerat.

Grundprinciper:

- secrets får inte byggas in i images
- secrets ska inte finnas i källkod
- secrets ska inte ligga i klartext i Git
- secrets ska inte skrivas till loggar
- secrets ska kunna roteras
- secrets ska ha ägare
- applikationer ska bara få läsa de secrets de behöver
- åtkomst till produktionssecrets ska vara begränsad

För Java EE-applikationer är detta ofta en stor förändring. Credentials som tidigare låg i applikationsserverns datasource-konfiguration behöver flyttas till en plattformsanpassad modell.

## Image-policy

Container images är en del av leveranskedjan. Om organisationen inte styr images riskerar den att köra sårbara, okända eller osupporterade komponenter.

En image-policy bör svara på:

- vilka bas-images är godkända?
- hur skannas images?
- vilka sårbarhetsnivåer blockerar deployment?
- får images hämtas från externa registries?
- hur signeras eller verifieras images?
- hur hanteras gamla images?
- hur patchas bas-images?
- vem ansvarar för runtime-versioner?
- hur spåras image till källkod och build?

Image-policy ska helst automatiseras i pipeline och plattform. Om policyn bara finns i dokument kommer den ofta att kringgås.

## Nätverkspolicyer

I en containerplattform kan intern trafik bli mer dynamisk än i traditionella zonmodeller. Utan nätverkspolicyer riskerar många workloads att kunna kommunicera mer fritt än avsett.

Nätverkspolicyer kan användas för att styra:

- vilka namespaces som får kommunicera
- vilka tjänster som får prata med databaser
- vilka workloads som får nå IBM MQ
- vilka tjänster som får exponeras externt
- vilka portar som är öppna
- vilka flöden som är interna

En viktig princip är att inte låta “allt internt är betrott” bli standard. Samtidigt måste policyerna vara begripliga och felsökningsbara, annars kommer team uppleva plattformen som oförutsägbar.

## Governance som stöd, inte hinder

Governance betyder styrning. I modernisering behövs governance för att undvika spretighet, men governance får inte bli en bromskloss.

Bra governance:

- gör rätt väg enkel
- tydliggör standarder
- dokumenterar undantag
- kopplar beslut till ansvar
- automatiserar kontroller där det går
- skiljer mellan krav, rekommendationer och experiment
- omprövar gamla beslut

Dålig governance:

- kräver möte för varje teknisk detalj
- saknar tydliga standarder
- hanterar undantag informellt
- gör team beroende av personliga kontakter
- dokumenterar beslut men följer inte upp dem
- blockerar förbättring utan att erbjuda alternativ

Målet är inte maximal kontroll. Målet är kontrollerad självservice.

## Policy-as-code

Policy-as-code innebär att regler uttrycks maskinläsbart och kan köras automatiserat i pipeline eller plattform. Det kan exempelvis gälla image-policy, labels, security context, resource limits eller krav på health checks.

Exempel på policyer:

- container får inte köras som root
- image måste komma från godkänt registry
- deployment måste ha resource requests
- workload måste ha ägarlabel
- secrets får inte monteras i otillåtna paths
- route får inte exponeras externt utan godkänd annotation
- namespace måste ha quota

Policy-as-code minskar beroendet av manuell granskning och gör regler mer konsekventa. Men policyerna måste vara begripliga och ha en process för undantag.

## Audit och spårbarhet

I enterprise-miljöer räcker det inte att system är säkra. Det måste också gå att visa vad som hänt.

Audit och spårbarhet bör omfatta:

- vem ändrade deployment?
- vilken image kördes?
- vem läste eller ändrade secrets?
- vilken pipeline skapade imagen?
- vilken kodversion motsvarar produktionen?
- vilka undantag har godkänts?
- när ändrades nätverkspolicyer?
- vem godkände extern exponering?
- vilka incidenter har påverkat tjänsten?

Spårbarhet är särskilt viktig när team får mer självservice. Självservice kräver inte mindre kontroll, utan bättre automatiserad och spårbar kontroll.

## Beslutsguide: säkerhetskrav för OpenShift-workloads

| Område | Minimikrav |
|---|---|
| Image | Godkänd bas-image, skannad och spårbar |
| Runtime | Kör non-root om inget tydligt undantag finns |
| Secrets | Hanteras via godkänd mekanism och loggas inte |
| Konfiguration | Externaliserad och separerad från image |
| RBAC | Minsta nödvändiga rättigheter |
| Servicekonto | Dedikerat vid behov, inte breda default-rättigheter |
| Nätverk | Endast nödvändiga flöden öppnas |
| Exponering | Extern route kräver tydligt beslut |
| Logging | Central logginsamling och inga känsliga uppgifter |
| Ägarskap | Team, kontaktväg och runbook finns |
| Undantag | Dokumenteras med ADR eller motsvarande |

## Beslutsguide: när krävs governancebeslut?

| Situation | Governancebeslut krävs? | Varför |
|---|---|---|
| Ny extern route | Ja | Exponering och säkerhet |
| Ny bas-image | Ja | Säkerhet och support |
| Workload som kräver root | Ja | Runtime-risk |
| Direkt åtkomst till Oracle | Ja | Data- och kopplingsrisk |
| Direkt åtkomst till IBM MQ | Ja, ofta | Integrationsmönster och legacy-koppling |
| Ny operator | Ja | Klusterrättigheter och livscykel |
| Ny stateful workload | Ja | Backup, restore och drift |
| Ny intern stateless tjänst enligt golden path | Nej, normalt inte | Bör kunna självbetjänas |
| Ändring av loggnivå | Nej | Lokal driftparameter |
| Tillfälligt experiment | Ja, lättviktigt | Behöver avgränsning och ägare |

## Tradeoffs

| Val | Fördel | Risk |
|---|---|---|
| Strikt policy från start | Bättre säkerhetsställning | Kan bromsa pilotteam |
| Mjuk start med få policyer | Snabbare lärande | Osäkra mönster kan etableras |
| Centraliserad secrets-hantering | Bättre kontroll | Kräver tydlig process |
| Teamnära secrets-hantering | Snabbare lokala beslut | Risk för spretighet |
| Automatisk policy enforcement | Konsekvent styrning | Kräver bra undantagsprocess |
| Manuell granskning | Flexibel bedömning | Skalar dåligt |
| Hårda nätverkspolicyer | Bättre isolering | Mer felsökning |
| Öppet internt nät | Enkelt initialt | Sämre säkerhetsmodell |

## Anti-patterns

### Cluster-admin som onboardinglösning

Om team får för höga rättigheter för att plattformen är svår att använda är problemet inte behörigheten. Problemet är plattformens användarresa.

### Secrets i Git

Att lägga secrets i Git, även i privata repositories, gör rotation och åtkomstkontroll svår och skapar långvarig risk.

### Policy utan undantagsprocess

Hårda regler utan tydlig undantagsprocess leder ofta till informella genvägar.

### Governance som personlig relation

Om beslut beror på vem man känner snarare än dokumenterade regler blir styrningen ojämlik och svår att skala.

### Säkerhet sist

Om säkerhet granskas först inför produktionssättning blir den en broms. Säkerhetskraven bör vara inbyggda i golden paths från början.

## Vanliga fallgropar

- Att ge breda rättigheter i pilot och sedan aldrig minska dem.
- Att inte skilja på mänskliga användare och servicekonton.
- Att sakna modell för secret rotation.
- Att låta varje team välja egna image-källor.
- Att inte ha standard för externa routes.
- Att skapa nätverkspolicyer utan felsökningsstöd.
- Att införa policy-as-code utan utbildning.
- Att sakna audit för produktionsändringar.
- Att inte dokumentera undantag.
- Att behandla governance som dokument i stället för arbetssätt.

## Arkitektens checklista

Innan plattformen används brett bör arkitekten kunna svara på:

- Vilka roller finns i RBAC-modellen?
- Vem får deploya till produktion?
- Hur hanteras servicekonton?
- Vilka bas-images är godkända?
- Hur skannas images?
- Vilka sårbarheter blockerar deployment?
- Hur hanteras secrets?
- Hur roteras secrets?
- Vilka nätverkspolicyer gäller som standard?
- När krävs ADR?
- Hur hanteras undantag?
- Vilka operators är godkända?
- Hur auditeras ändringar?
- Hur kopplas policyer till golden paths?
- Vem äger governance-processen?

## Koppling till moderniseringscaset

Nordisk Handel AB inser tidigt att OpenShift inte får bli en frizon där varje team gör egna säkerhetsval. Samtidigt vill organisationen inte återskapa gamla change-processer.

Plattformsteamet och säkerhetsteamet tar därför fram en första governance-modell:

- godkända bas-images
- krav på non-root-körning
- standardiserad secrets-hantering
- RBAC per miljö
- extern exponering via beslutad route-modell
- nätverkspolicyer för databas- och MQ-åtkomst
- ADR för avvikelser
- audit för produktionsändringar
- golden path för stateless Java-tjänst

Detta gör att pilotteamet kan arbeta självständigt inom tydliga ramar. Säkerhet blir inte en sen granskning, utan en del av plattformens normala användning.

## Snabb sammanfattning

- Säkerhet i OpenShift bygger på policyer nära workloaden, inte bara perimeter.
- Least privilege bör tillämpas på användare, servicekonton, containers, secrets och nätverk.
- RBAC speglar ansvarsfördelning och måste designas med organisationen i åtanke.
- Secrets och image-policy är centrala styrningsområden.
- Governance ska stödja självservice, inte ersätta den med nya flaskhalsar.
- Policy-as-code kan göra regler konsekventa, men kräver begriplighet och undantagsprocess.
- Säkerhet ska byggas in i golden paths från början.

## Nästa steg

Nästa kapitel går in i dataarkitektur och modernisering från Oracle. Där analyseras när Oracle bör behållas, kapslas in, kompletteras eller ersättas, och hur organisationen kan introducera alternativa datalösningar utan att skapa ny spretighet.
