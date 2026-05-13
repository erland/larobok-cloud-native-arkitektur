# Kapitel 15: Beslutsmodell för teknikval

## Varför detta kapitel finns

Efter fjorton kapitel har boken beskrivit många arkitekturbeslut: Podman, OpenShift, säkerhet, Oracle, PostgreSQL, IBM MQ, event streaming, Elasticsearch, Ceph, stateful workloads, driftsmodell, kontinuitet, strangler pattern, Java-modernisering och observability. Varje område innehåller viktiga tradeoffs.

I en större organisation räcker det inte att enskilda arkitekter eller team gör bra teknikval var för sig. Besluten behöver vara spårbara, jämförbara och möjliga att ompröva. Annars riskerar organisationen att få en ny spretig plattform där varje team väljer egen databas, egen broker, egen secrets-modell och egen observability-lösning.

Det här kapitlet samlar bokens resonemang i en beslutsmodell för teknikval. Fokus ligger på ADR:er, teknikradar, guardrails, undantag, fitness functions och praktisk governance.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- formulera teknikbeslut som spårbara arkitekturbeslut
- använda ADR:er för att dokumentera beslut, tradeoffs och konsekvenser
- skilja mellan standard, rekommendation, experiment och undantag
- skapa en enkel teknikradar för moderniseringsprogrammet
- definiera fitness functions för att följa arkitekturprinciper
- undvika att governance blir antingen för svag eller för tung


## Varför teknikval behöver styras

Cloud native-modernisering ökar ofta teamens frihet. Det är bra, men frihet utan ramar kan leda till högre komplexitet.

Risker:

- flera databaser införs utan tydligt ägarskap
- samma problem löses på olika sätt i olika team
- nya brokers införs utan driftmodell
- Elasticsearch används både som sökindex, databas och integrationsyta
- Ceph används för workloads utan backupmodell
- OpenShift-namespaces skapas utan livscykel
- secrets hanteras olika mellan team
- observability blir ojämn och svår att stödja

Styrning behövs inte för att hindra team, utan för att göra lokala beslut hållbara i helheten.


## Beslut som arkitekturartefakter

Ett arkitekturbeslut bör inte bara leva i mötesanteckningar eller i en persons minne. Det bör vara en artefakt som kan hittas, läsas och omprövas.

Ett bra beslut beskriver vilket problem som skulle lösas, vilka alternativ som övervägdes, vilket alternativ som valdes, varför det valdes, vilka tradeoffs som accepterades, vilka konsekvenser beslutet får, vem som äger beslutet och när beslutet bör omprövas.

Detta är särskilt viktigt i modernisering eftersom dagens övergångslösning lätt blir morgondagens permanentarkitektur.


## ADR: Architecture Decision Record

ADR står för Architecture Decision Record. En ADR är ett kort dokument som beskriver ett arkitekturbeslut och dess konsekvenser.

| Fält | Innehåll |
|---|---|
| Titel | Kort beslutstitel |
| Status | Föreslagen, accepterad, ersatt eller avvecklad |
| Kontext | Varför beslutet behövs |
| Alternativ | Vilka alternativ som övervägdes |
| Beslut | Vad som valdes |
| Konsekvenser | Positiva och negativa följder |
| Ägare | Vem som ansvarar |
| Omprövning | När beslutet bör ses över |

ADR:er bör vara korta nog att skrivas, men tydliga nog att vara användbara. De ska inte ersätta arkitekturdialogen, utan göra resultatet av dialogen spårbart.


## Exempel: ADR för PostgreSQL

**Titel:** Använd PostgreSQL för nya avgränsade relationsbehov.

**Kontext:** Organisationen vill minska slentrianmässig användning av Oracle för nya tjänster, men Oracle behålls för affärskritisk kärndata.

**Alternativ:** fortsätt använda Oracle, använd PostgreSQL för avgränsade nya domäner, eller använd annan databasmodell.

**Beslut:** PostgreSQL får användas för nya avgränsade tjänster när teamet äger datamodellen, backup och restore är definierat och direkt beroende till Oracle-schema saknas.

**Konsekvenser:** Team får bättre autonomi, men organisationen behöver driftmodell, schemahantering och datagovernance.


## Exempel: ADR för Elasticsearch

**Titel:** Elasticsearch används som sekundärt index, inte primär affärsdatabas.

**Kontext:** Kundtjänst behöver snabb ordersökning utan att belasta Oracle.

**Alternativ:** fortsätt söka direkt i Oracle, skapa sekundärt index i Elasticsearch, eller skapa separat relationsbaserad läsmodell.

**Beslut:** Elasticsearch används som sekundärt index för ordersökning. Oracle är primär sanning. Indexet ska kunna reindexeras.

**Konsekvenser:** Sök blir snabbare och Oracle avlastas, men eventual consistency och indexeringsfördröjning måste accepteras och mätas.

Denna typ av ADR förhindrar att Elasticsearch senare oavsiktligt börjar användas som primär databas.


## Teknikradar

En teknikradar visar vilka tekniker och mönster organisationen rekommenderar, utvärderar, begränsar eller avråder från.

| Ring | Betydelse |
|---|---|
| Adopt | Rekommenderad standard för lämpliga användningsfall |
| Trial | Får användas i kontrollerade pilotfall |
| Assess | Utvärderas, men inte för produktion utan beslut |
| Hold | Ska undvikas eller kräver särskilt undantag |

Exempel för Nordisk Handel AB:

| Teknik eller mönster | Ring | Kommentar |
|---|---|---|
| OpenShift för nya containeriserade tjänster | Adopt | Målplattform |
| Podman för lokal containerverifiering | Adopt | Utvecklings- och verifieringsstöd |
| PostgreSQL för avgränsade nya domäner | Trial | Kräver driftmodell |
| Oracle för kärntransaktioner | Adopt | Behålls där den äger primär sanning |
| Direkt schemaåtkomst från nya tjänster | Hold | Kräver undantag |
| IBM MQ för kritiska legacy-flöden | Adopt | Behålls initialt |
| Event streaming för domänhändelser | Trial | Kräver kontrakt och governance |
| Elasticsearch som sekundärt index | Adopt | Inte primärdatabas |
| Ceph för plattformsnära lagring | Trial | Kräver stateful driftmognad |
| Secrets i Git | Hold | Ej tillåtet |

Teknikradarn ska inte vara statisk. Den bör omprövas regelbundet.


## Standard, rekommendation, experiment och undantag

Alla beslut ska inte ha samma styrka. En bra governance-modell skiljer mellan olika nivåer.

| Nivå | Betydelse |
|---|---|
| Standard | Normalval som ska användas om inget särskilt skäl finns |
| Rekommendation | Föredraget val, men alternativ kan vara rimliga |
| Experiment | Kontrollerad utvärdering med tydlig ägare och slutdatum |
| Undantag | Avvikelse från standard som kräver motivering |
| Förbud eller hold | Ska inte användas utan särskilt beslut |

Detta gör styrningen mer nyanserad. Allt behöver inte vara antingen fritt eller förbjudet.


## Undantagsprocess

Undantag är ibland nödvändiga. Problemet är inte att undantag finns, utan att de blir osynliga eller permanenta.

Ett bra undantag bör ha skäl, ägare, riskbeskrivning, tidsgräns, kompensationsåtgärder, plan för omprövning, dokumenterad ADR och beskrivning av påverkan på drift och säkerhet.

Exempel: En legacy-applikation får tillfälligt köra med sticky sessions, men beslutet har slutdatum och en plan för att minska sessionsberoende.


## Fitness functions

En fitness function är ett testbart kriterium som visar om arkitekturen följer en princip. Det kan vara automatiserat eller manuellt granskat.

| Princip | Fitness function |
|---|---|
| Images ska vara spårbara | Varje deployment refererar version eller digest |
| Tjänster ska vara observerbara | Tjänsten har loggar, metrics och ägare |
| Secrets ska inte ligga i Git | Pipeline blockerar hemligheter |
| Stateful workloads kräver restore | Restore-test finns dokumenterat |
| Elasticsearch är sekundärt | Primär källa och reindexeringsprocess är dokumenterad |
| OpenShift-workloads ska ha resource limits | Deployment utan requests/limits blockeras eller flaggas |
| Nya tjänster ska inte direktläsa Oracle-schema | ADR krävs för direktåtkomst |

Fitness functions gör arkitekturprinciper mer konkreta. De hjälper organisationen att se om målarkitekturen efterlevs i praktiken.


## Beslutsmodell för teknikval

En praktisk beslutsmodell kan bestå av sex steg.

### 1. Definiera användningsfallet

Börja med problemet, inte produkten. Fråga vilket affärsproblem som löses, om behovet handlar om transaktion, sök, messaging, lagring eller observability, om data är primär sanning eller sekundär modell, om flödet är synkront eller asynkront och vilka krav som finns på tillgänglighet, säkerhet och drift.

### 2. Identifiera mönstret

Välj arkitekturmönster före teknik. Exempel är lokal transaktion, saga, sekundärt index, objektlagring, arbetskö, domänhändelse, strangler pattern eller adapter/fasad.

### 3. Jämför alternativ

Minst två realistiska alternativ bör övervägas. Även “behåll nuvarande lösning” är ofta ett alternativ.

### 4. Bedöm konsekvenser

Konsekvenser ska omfatta drift, säkerhet, dataägarskap, backup, restore, observability, kostnad, kompetens, avveckling och governance.

### 5. Dokumentera beslutet

Beslutet dokumenteras med ADR om det påverkar arkitektur, drift eller flera team.

### 6. Följ upp

Beslut behöver följas upp. Om förutsättningar ändras ska beslutet kunna omprövas.


## Beslutsguide: när krävs ADR?

| Situation | ADR krävs? |
|---|---|
| Ny databasprodukt i produktion | Ja |
| Ny broker eller eventplattform | Ja |
| Direkt åtkomst till Oracle-schema | Ja |
| Elasticsearch som ny sökmodell | Ja |
| Stateful workload på OpenShift | Ja |
| Ny extern exponering av affärstjänst | Ofta |
| Avvikelse från secrets-standard | Ja |
| Ny intern stateless tjänst enligt golden path | Normalt nej |
| Mindre konfigurationsändring | Nej |
| Experiment med slutdatum | Ja, lättviktig ADR |


## Beslutsguide: teknikval per problemtyp

| Problem | Bör börja med | Undvik att börja med |
|---|---|---|
| Långsam sökning | Dataroll och primär källa | Produktnamn |
| Ny integration | Kommando, event eller request/reply | Broker-val |
| Ny datalagring | Dataägarskap och RPO/RTO | “Vilken databas gillar teamet?” |
| Legacy-modernisering | Strangler-kandidat | Full ersättning direkt |
| Plattformsexponering | Säkerhets- och route-modell | Snabb extern route |
| Stateful workload | Backup, restore och driftmodell | PVC som enda svar |
| Java-modernisering | Runtime-beroenden | Direkt omskrivning |
| Observability | Affärsflöde och SLI | Bara loggverktyg |


## Governance utan flaskhals

Governance måste vara tillräckligt stark för att ge riktning, men tillräckligt lätt för att inte stoppa teamen.

Bra governance erbjuder golden paths, dokumenterar beslut, gör standarder tydliga, ger snabb undantagshantering, automatiserar kontroller, följer upp lärande och uppdaterar teknikradar.

Dålig governance kräver möte för varje liten sak, saknar tydliga kriterier, skapar informella maktvägar, säger nej utan alternativ, dokumenterar utan uppföljning och gör undantag permanenta.

Målet är att team ska kunna fatta fler bra beslut själva.


## Roller i beslutsmodellen

| Roll | Ansvar |
|---|---|
| Applikationsteam | Beskriver behov, äger implementation och driftbeteende |
| Plattformsteam | Definierar golden paths och plattformsstandarder |
| Säkerhetsteam | Definierar säkerhetskrav och granskar risk |
| Datateam/DBA | Bidrar till dataägarskap, backup och databasdrift |
| Integrationsteam | Bidrar till messagingmönster och kontrakt |
| Arkitekturforum | Beslutar större avvikelser och målarkitektur |
| Produktägare | Prioriterar affärsnytta och risk |

En beslutsmodell fungerar bara om rollerna är tydliga.


## Anti-patterns

### Teknikradar som dekoration

Om teknikradarn inte används vid beslut eller uppdateras efter lärdomar blir den bara en presentation.

### ADR som byråkrati

Om ADR:er är för långa eller krävs för allt slutar team skriva dem. De ska vara lätta men användbara.

### Alla får välja allt

Total frihet leder ofta till högre driftkostnad och svagare säkerhet.

### Arkitekturforum som flaskhals

Om alla beslut måste gå via forumet blir governance långsam. Forumet ska fokusera på större beslut och avvikelser.

### Undantag utan slutdatum

Undantag som saknar omprövning blir ofta permanenta standarder.


## Vanliga fallgropar

- Att dokumentera beslut efter att de redan blivit permanenta.
- Att inte inkludera drift och säkerhet i teknikval.
- Att underskatta kompetenskostnad.
- Att välja teknik utifrån trend i stället för användningsfall.
- Att inte definiera ägare för nya plattformskomponenter.
- Att sakna avvecklingsplan för övergångslösningar.
- Att inte ompröva äldre ADR:er.
- Att ha för många standarder utan golden paths.
- Att mäta adoption men inte effekt.
- Att sakna process för experiment.


## Arkitektens checklista

Innan ett större teknikval accepteras bör arkitekten kunna svara på:

- Vilket problem löser valet?
- Vilka alternativ övervägdes?
- Varför valdes detta alternativ?
- Vilka tradeoffs accepteras?
- Vem äger tekniken?
- Vem driftar den?
- Hur säkras den?
- Hur observeras den?
- Hur backas data upp?
- Hur återställs den?
- Vilken kompetens krävs?
- Vilken kostnad tillkommer?
- Vilka team påverkas?
- Finns avvecklingsplan för gammal lösning?
- Behövs ADR?
- När ska beslutet omprövas?


## Koppling till moderniseringscaset

Nordisk Handel AB inför en enkel beslutsmodell. Arkitekturforumet beslutar inte varje detalj, men sätter ramar:

- OpenShift är målplattform för nya containeriserade tjänster.
- Podman används för lokal containerverifiering.
- Oracle behålls för kärntransaktioner.
- PostgreSQL får användas för avgränsade domäner efter ADR.
- IBM MQ behålls för kritiska legacy-flöden.
- Event streaming provas för domänhändelser.
- Elasticsearch används som sekundärt index.
- Ceph utvärderas för lagringsbehov där driftmodell finns.
- Secrets i Git är inte tillåtet.
- Stateful workloads kräver restore-test.

Teknikradarn och ADR:erna blir ett sätt att hålla moderniseringen sammanhängande utan att stoppa teamens framdrift.


## Snabb sammanfattning

- Teknikval behöver styras eftersom lokal frihet annars kan skapa global komplexitet.
- ADR:er gör beslut spårbara och omprövningsbara.
- Teknikradar hjälper organisationen skilja mellan standard, experiment och undantag.
- Fitness functions gör arkitekturprinciper testbara.
- Governance ska stödja självservice, inte skapa nya flaskhalsar.
- Undantag behöver ägare, riskbeskrivning och slutdatum.
- Beslut ska börja med användningsfall och mönster, inte produktnamn.

## Nästa steg

Nästa kapitel sammanfattar boken i en målarkitektur och roadmap. Där knyts alla delar samman till en praktisk riktning för fortsatt modernisering.
