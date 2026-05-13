# Kapitel 8: Ceph och stateful workloads

## Varför detta kapitel finns

Hittills har boken betonat att containers helst ska vara ersättningsbara och att applikationsinstanser inte ska bära affärskritiskt tillstånd lokalt. Samtidigt består enterprise-arkitektur av mycket mer än stateless API:er. Databaser, köplattformar, Elasticsearch, filflöden, objektlagring och vissa legacy-komponenter behöver lagra data som överlever poddar, noder och uppgraderingar.

Det är här stateful workloads och persistent lagring blir centrala. I Red Hat- och OpenShift-sammanhang är Ceph relevant eftersom det kan ge block-, fil- och objektlagring för plattformen. Men Ceph är inte ett magiskt svar på alla lagringsfrågor. Precis som med Oracle, IBM MQ och Elasticsearch måste arkitekten först förstå datarollen, driftmodellen och riskerna.

Det här kapitlet ger en arkitekturell guide till Ceph, persistent volumes, blocklagring, fillagring, objektlagring och stateful workloads i OpenShift.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- skilja mellan stateless och stateful workloads
- beskriva skillnaden mellan block-, fil- och objektlagring
- förklara rollen för persistent volumes och storage classes i OpenShift
- bedöma när Ceph passar som lagringsplattform
- förstå varför backup, restore, RPO och RTO inte ersätts av persistent volumes
- formulera beslutsstöd för stateful workloads i OpenShift

## Stateless och stateful

En stateless workload lagrar inte affärskritiskt tillstånd i den körande instansen. Om podden försvinner kan en ny startas utan att viktig data går förlorad. Det gör stateless workloads väl lämpade för containerplattformar.

En stateful workload hanterar data som måste bevaras. Exempel:

- databaser
- köplattformar
- event streaming
- Elasticsearch
- filservrar
- cache med särskilda krav
- objektlagring
- vissa batch- eller integrationskomponenter

Stateful workloads är inte förbjudna i OpenShift, men de kräver mer disciplin. En podd kan fortfarande ersättas, men datat får inte försvinna eller bli korrupt.

## Varför lagring är en arkitekturfråga

Lagring uppfattas ibland som en infrastrukturdetalj: applikationen behöver disk, plattformen ger disk. Det är en farlig förenkling.

Lagringsval påverkar:

- prestanda
- tillgänglighet
- backup
- restore
- kostnad
- dataskydd
- kapacitetsplanering
- incidenthantering
- uppgraderingar
- placering av state
- ansvar mellan team

En felvald lagringsmodell kan göra en applikation svår att skala, återställa eller felsöka.

## Block-, fil- och objektlagring

Olika lagringsmodeller har olika egenskaper. Arkitekten bör börja med att förstå vilken semantik applikationen behöver.

| Typ | Beskrivning | Typiska användningsfall |
|---|---|---|
| Blocklagring | Lagring presenteras som blockenhet | Databaser, stateful applikationer |
| Fillagring | Lagring presenteras som filsystem, ibland delat | Legacy-filer, delade kataloger |
| Objektlagring | Data lagras som objekt via API | Dokument, bilder, exporter, arkiv |
| Ephemeral storage | Tillfällig lagring som kan försvinna | Cache, tempfiler, build-artifacts |

Fel lagringstyp kan skapa svårigheter. Att använda delad filyta som integrationsplattform kan bevara gamla kopplingar. Att lagra stora dokument i relationsdatabas kan ge onödig belastning. Att använda ephemeral storage för viktig data kan ge dataförlust.

## Persistent volumes och claims

I OpenShift används persistent volumes för att ge workloads lagring som överlever poddens livscykel. Applikationen begär lagring genom en persistent volume claim. Plattformen matchar begäran mot en storage class.

Förenklat:

- **Persistent volume**: faktisk lagringsresurs
- **Persistent volume claim**: workloadens begäran om lagring
- **Storage class**: typ av lagring med definierade egenskaper

Detta ger en bra abstraktion, men den får inte bli för förenklad. Alla storage classes är inte lika. De kan skilja sig i prestanda, backup, replikeringsmodell, kostnad och support.

## Storage classes som arkitekturbeslut

En storage class bör inte bara heta “standard”. Den bör uttrycka vad lagringen är avsedd för.

En mogen storage class-modell bör beskriva:

- lagringstyp
- prestandaprofil
- replikeringsmodell
- backupstöd
- restoreförväntan
- tillgänglighet
- kostnad
- tillåtna workloads
- ägare
- supportmodell
- eventuell dataklassning

Exempel:

| Storage class | Passar för | Kommentar |
|---|---|---|
| block-standard | Mindre databaser och stateful tjänster | Kräver backupmodell |
| block-premium | Prestandakänsliga databaser | Dyrare och mer styrd |
| file-shared | Legacy-behov av delad katalog | Bör vara restriktiv |
| object-standard | Dokument och exportfiler | Kräver metadataansvar |
| ephemeral | Tillfälliga filer | Ej för affärskritisk data |

När storage classes saknar tydliga egenskaper kommer team att välja på känsla.

## Cephs roll

Ceph är en distribuerad lagringsplattform som kan tillhandahålla flera typer av lagring. I Red Hat- och OpenShift-sammanhang kan Ceph vara relevant för block-, fil- och objektlagring beroende på implementation och plattformsval.

Ceph kan ge organisationen:

- gemensam lagringsplattform
- stöd för flera lagringstyper
- integration med OpenShift
- skalbar lagringsgrund
- möjlighet till objektlagring
- ett alternativ till separata speciallösningar

Men Ceph innebär också ansvar:

- lagringskompetens
- övervakning
- kapacitetsplanering
- uppgraderingsstrategi
- incidenthantering
- prestandatest
- backup- och restoremodell
- tydlig supportkedja

Ceph ska behandlas som strategisk plattformsförmåga, inte som “bara disk”.

## När Ceph passar

Ceph passar ofta när:

- OpenShift behöver integrerad lagring
- flera lagringstyper behövs
- organisationen vill standardisera plattformsnära lagring
- objektlagring är relevant för applikationer
- plattformsteamet kan äga eller samäga lagringsförmågan
- kapacitets- och driftmodell finns
- workload-kraven är kända och testade

Ceph passar sämre när:

- organisationen saknar lagringskompetens
- kraven är mycket specialiserade
- befintlig extern lagring redan löser behovet bättre
- stateful workloads är få och enkla
- backup och restore inte är definierat
- plattformsteamet redan är överbelastat
- Ceph införs bara för att det ingår i stacken

Teknikens närvaro i ekosystemet är inte tillräckligt skäl. Användningsfallet måste motivera driftansvaret.

## Databaser på OpenShift-lagring

Databaser är ett vanligt stateful användningsfall. Men att en databas kan köras i OpenShift betyder inte att den alltid bör göra det.

Innan en databas körs på plattformsnära lagring bör följande vara klart:

- vilken storage class används?
- är prestandan testad med realistisk last?
- finns backup?
- har restore testats?
- finns RPO och RTO?
- vem patchar databasen?
- vem hanterar incidenter?
- finns monitoring?
- hur görs uppgradering?
- finns stöd för failover?
- är detta produktionsmönster godkänt?

För mindre nya tjänster kan PostgreSQL på OpenShift vara rimligt om driftmodellen är mogen. För affärskritisk Oracle-kärna kan extern etablerad drift vara bättre under lång tid.

## Objektlagring

Objektlagring passar ofta bättre än databas eller delad filyta för dokument, bilder, exporter och stora binära objekt.

Användningsfall:

- kunddokument
- orderbilagor
- rapportfiler
- exporter
- integrationsfiler
- arkiv
- uppladdade dokument
- datautdrag för analys

Objektlagring kräver dock metadata. Själva filen kan ligga som objekt, men affärsinformation om objektet behöver ofta ligga i en databas.

Frågor:

- vem äger objektet?
- vilken metadata behövs?
- hur hanteras åtkomst?
- hur länge ska objektet sparas?
- hur raderas objekt?
- är objektet känsligt?
- behövs versionshantering?
- hur kopplas objektet till affärsdata?

Objektlagring löser fillagring, men inte informationsmodellering.

## Fillagring och legacy

Delade filytor är vanliga i äldre miljöer. Batchjobb, integrationer och applikationer kan skriva och läsa från gemensamma kataloger.

I containerplattformar bör delad fillagring användas försiktigt. Den kan vara nödvändig som övergång, men riskerar att bevara gamla kopplingar.

Risker:

- filer blir dolda integrationskontrakt
- katalogstruktur saknar ägare
- samtidiga skrivningar skapar fel
- rensning glöms bort
- åtkomst blir för bred
- filsystemet blir informell databas
- modernisering försenas

Om fillagring behövs bör den ha tydligt syfte, ägare och avvecklingsplan.

## Backup, restore, RPO och RTO

Persistent volume är inte backup. En volume kan överleva att en podd försvinner, men den skyddar inte mot:

- raderad data
- korrupt data
- applikationsbuggar
- felaktiga batchjobb
- ransomware-liknande påverkan
- större plattformsfel
- mänskliga misstag

Därför behövs backup och restore.

RPO, recovery point objective, beskriver hur mycket data organisationen kan acceptera att förlora. RTO, recovery time objective, beskriver hur snabbt återställning behöver ske.

| Begrepp | Fråga |
|---|---|
| Backup | Finns kopia av data? |
| Restore | Kan data faktiskt återställas? |
| RPO | Hur mycket data får gå förlorad? |
| RTO | Hur snabbt måste tjänsten återställas? |
| Retention | Hur länge sparas backup eller data? |

En backup som aldrig återställningstestats är inte en verifierad återställningsförmåga.

## Kapacitet och kostnad

Lagring växer ofta snabbare än förväntat. Loggar, index, dokument, köer och databaser kan alla skapa snabb tillväxt.

Kapacitetsstyrning bör omfatta:

- initial uppskattning
- tillväxtmodell
- kvoter
- larm
- kostnadsfördelning
- retention
- arkivering
- rensningsprocess
- regelbunden uppföljning

Självservice utan kostnadssynlighet kan snabbt skapa okontrollerad lagringstillväxt.

## Stateful workloads och observability

Stateful workloads behöver särskilda signaler.

| Komponent | Viktiga signaler |
|---|---|
| Databas | lagringsnivå, latency, anslutningar, lås, backupstatus |
| Elasticsearch | shardhälsa, indexeringsfördröjning, query latency, disk |
| Broker/MQ | ködjup, dead-letter, consumer rate |
| Event streaming | consumer lag, topic-tillväxt, disk |
| Ceph | kapacitet, latency, rebalansering, fel, diskhälsa |
| Objektlagring | antal objekt, storlek, fel, åtkomstmönster |

Om stateful komponenter saknar observability kommer incidenter ofta upptäckas sent.


## Särskild avgränsning

Det här kapitlet gör inte Ceph till standardval för all lagring. Poängen är snarare att ge arkitekten ett språk för att skilja mellan lagringsbehov. Vissa workloadstyper passar bra på plattformsnära lagring, medan andra bör ligga kvar i etablerade externa miljöer tills organisationen har tillräcklig driftmognad.

En bra tumregel är att varje stateful beslut ska kunna beskriva tre saker: vilken data som skyddas, hur den återställs och vem som äger livscykeln. Om någon av dessa tre saknas är beslutet inte färdigt.

## Beslutsguide: välj lagringstyp

| Scenario | Rekommenderad lagring |
|---|---|
| Databas med persistent disk | Blocklagring |
| Dokument, bilder eller exportfiler | Objektlagring |
| Legacy kräver delad katalog | Fillagring som kontrollerat övergångsmönster |
| Temporära filer | Ephemeral storage |
| Loggar | Central loggplattform, inte lokal disk |
| Cache som kan återskapas | Cachetjänst eller ephemeral storage |
| Elasticsearch | Blocklagring med tydlig kapacitetsmodell |
| Backup | Separat backupmål och restore-process |

## Beslutsguide: när ska workloaden vara stateful i OpenShift?

| Fråga | Om ja | Slutsats |
|---|---|---|
| Finns testad backup och restore? | Ja | Ett grundkrav är uppfyllt |
| Finns definierat RPO/RTO? | Ja | Återställningskrav är kända |
| Är prestanda testad? | Ja | Lägre produktionsrisk |
| Finns ägare för drift? | Ja | Ansvar är tydligt |
| Finns monitoring? | Ja | Operativ kontroll finns |
| Krävs specialiserad databasdrift? | Ja | Extern drift kan vara bättre |
| Är workloaden affärskritisk? | Ja | Kräver särskild ADR och riskanalys |
| Är detta bara testdata? | Ja | Enklare modell kan räcka |

## Tradeoffs

| Val | Fördel | Risk |
|---|---|---|
| Ceph som gemensam lagring | Enhetlig plattformsmodell | Kritisk komponent med hög driftkravbild |
| Extern lagringsplattform | Beprövad drift | Mer integration och beroenden |
| Databas i OpenShift | Självservice och plattformsnärhet | Stateful-drift kan underskattas |
| Databas utanför OpenShift | Stabil etablerad drift | Mer nätverksberoenden |
| Objektlagring för dokument | Bra skalning och livscykel | Kräver metadataansvar |
| Fillagring för legacy | Snabb kompatibilitet | Risk att gammalt mönster lever kvar |
| Kort retention | Lägre kostnad | Mindre historik |
| Lång retention | Mer historik | Högre kostnad och dataskyddsrisk |

## Anti-patterns

### Persistent volume som backup

En persistent volume ersätter inte backup. Den bevarar data över poddens livscykel, men skyddar inte mot logiska fel.

### Allt på samma storage class

Om alla workloads använder samma lagringsklass försvinner viktiga skillnader i prestanda, skydd och kostnad.

### Delad filyta som integrationsplattform

Delade kataloger kan vara praktiska, men skapar ofta otydliga kontrakt och svår felsökning.

### Stateful först, driftmodell sen

Att köra stateful workloads i OpenShift innan backup, restore och monitoring finns skapar onödig produktionsrisk.

### Objekt utan metadata

Objektlagring utan tydlig metadata och livscykel leder till data som är svår att hitta, skydda och radera.

## Vanliga fallgropar

- Att behandla lagring som en teknisk detalj.
- Att inte skilja mellan block, fil och objekt.
- Att sakna RPO och RTO.
- Att inte testa restore.
- Att underskatta lagringskostnad.
- Att sakna storage class-dokumentation.
- Att låta team välja lagring utan riktlinjer.
- Att köra databaser på oprövad storage class.
- Att behålla filbaserad integration för länge.
- Att sakna ägare för data på volymen.

## Arkitektens checklista

Innan en stateful workload godkänns bör arkitekten kunna svara på:

- Vilken typ av data lagras?
- Är data temporär eller persistent?
- Behövs block-, fil- eller objektlagring?
- Vad är primär sanning?
- Vilken storage class används?
- Vilken prestanda krävs?
- Vilken tillgänglighet krävs?
- Hur sker backup?
- Har restore testats?
- Vilket RPO gäller?
- Vilket RTO gäller?
- Vem äger data?
- Vem äger drift?
- Hur mäts kapacitet?
- Hur hanteras retention?
- Är Ceph rätt lösning eller bör extern lagring användas?
- Behövs ADR?

## Koppling till moderniseringscaset

Nordisk Handel AB analyserar sina lagringsbehov efter att ha beslutat att Oracle initialt ska ligga kvar utanför OpenShift. Organisationen ser flera andra lagringsbehov:

- Elasticsearch behöver persistent lagring för sökindex.
- Nya PostgreSQL-behov kan uppstå för avgränsade tjänster.
- Kunddokument och exportfiler bör flyttas mot objektlagring.
- Vissa legacy-flöden använder delade filytor.
- Loggar ska inte skrivas till lokal disk utan samlas centralt.

Plattformsteamet tar fram tre första storage classes och dokumenterar deras syfte. Ceph utvärderas som strategisk lagringsplattform, men bara för use cases där backup, restore, monitoring och ägarskap kan definieras.

Beslutet blir att Ceph är en viktig kandidat, men inte ett standardsvar på alla datafrågor.

## Snabb sammanfattning

- Stateful workloads kräver mer än persistent volume.
- Block-, fil- och objektlagring löser olika problem.
- Ceph kan ge gemensam lagringsförmåga men kräver tydlig driftmodell.
- Persistent volume är inte backup.
- RPO, RTO och restore-test är centrala krav.
- Objektlagring passar ofta bättre än relationsdatabas för dokument och filer.
- Delad fillagring bör användas försiktigt och helst som övergångsmönster.
- Storage classes bör dokumentera prestanda, skydd, kostnad och ägarskap.

## Nästa steg

Nästa kapitel behandlar cloud native-driftsmodell. Där går vi från lagring och state till hur team, plattform och drift organiserar ansvar, support, incidenter, runbooks och produktionsklarhet.
