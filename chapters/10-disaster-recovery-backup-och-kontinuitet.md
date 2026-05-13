# Kapitel 10: Disaster recovery, backup och kontinuitet

## Varför detta kapitel finns

I de tidigare kapitlen har vi konstaterat att modernisering inte bara handlar om att deploya containers. När applikationer, data, köer, index och lagring flyttas in i eller kopplas till OpenShift förändras också organisationens ansvar för återställning och kontinuitet.

Disaster recovery, backup och kontinuitet blir särskilt viktiga i en miljö där flera tekniker samverkar: Oracle kan ligga utanför OpenShift, PostgreSQL kan finnas för nya tjänster, IBM MQ kan bära legacy-flöden, Elasticsearch kan vara sekundärt index och Ceph kan användas för persistent lagring. Varje del har olika återställningskrav och olika konsekvenser vid fel.

Det här kapitlet hjälper arkitekten att formulera en praktisk kontinuitetsmodell: vad som ska skyddas, hur det återställs, i vilken ordning och med vilket ansvar.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- skilja mellan backup, restore, disaster recovery och kontinuitetsplanering
- förklara RPO och RTO i praktiska arkitekturbeslut
- identifiera vad som behöver återställas i en OpenShift-baserad miljö
- förstå varför persistent volume inte är backup
- resonera om återställningsordning för applikation, data, messaging och index
- formulera en checklista för kontinuitetskrav i moderniseringsprojekt

## Backup, restore och disaster recovery

Backup är en kopia av data eller konfiguration. Restore är förmågan att faktiskt återställa från kopian. Disaster recovery, DR, handlar om att återställa tjänster efter större fel, exempelvis klusterhaveri, datakorruption, regional störning eller allvarlig mänsklig felhantering.

Kontinuitet handlar om verksamhetens förmåga att fortsätta eller återuppta kritiska processer.

| Begrepp | Fråga |
|---|---|
| Backup | Finns en kopia? |
| Restore | Kan kopian användas för återställning? |
| Disaster recovery | Kan tjänsten återställas efter större incident? |
| Kontinuitet | Kan verksamheten fortsätta på acceptabel nivå? |
| RPO | Hur mycket data får förloras? |
| RTO | Hur snabbt måste tjänsten återställas? |

Ett vanligt misstag är att likställa backup med återställningsförmåga. Backup utan testad restore är bara en hypotes.

## RPO och RTO

RPO, recovery point objective, beskriver hur mycket dataförlust som kan accepteras. RTO, recovery time objective, beskriver hur lång tid återställning får ta.

Exempel:

- En orderdatabas kan ha mycket lågt RPO eftersom förlorade order är affärskritiska.
- Ett Elasticsearch-index kan ha högre RPO om det kan återskapas från primär källa.
- En loggplattform kan ha annat RPO beroende på regulatoriska krav.
- En cache kan ha högt RPO eftersom den kan återskapas.
- Ett externt kundgränssnitt kan ha lågt RTO om avbrott påverkar försäljning.

RPO och RTO ska sättas per tjänst eller dataroll, inte generellt för hela plattformen.

## Vad behöver skyddas?

I en OpenShift-orienterad målarkitektur finns flera typer av tillgångar som kan behöva skyddas.

| Tillgång | Exempel | Återställningsstrategi |
|---|---|---|
| Applikationskod | Git-repository | Återskapas från källkod |
| Container images | Registry | Återskapas via build eller restore av registry |
| Deploymentkonfiguration | YAML, Helm, Kustomize, GitOps | Återskapas från Git |
| Secrets | Credentials, certifikat | Restore eller rotation |
| Databaser | Oracle, PostgreSQL | Backup, replikering, restore-test |
| Messaging | MQ, broker, event streaming | Konfiguration, ködata, offset, topics |
| Elasticsearch | Index och mappings | Reindexering eller snapshot |
| Objektlagring | Dokument och filer | Replikering, backup, lifecycle |
| Plattformskonfiguration | Namespaces, RBAC, policies | GitOps eller plattformsbackup |
| Observability-data | Loggar, metrics, traces | Retention och export beroende på krav |

Allt behöver inte skyddas på samma sätt. Vissa saker bör återskapas från kod och konfiguration. Andra kräver backup. Några kan byggas om från primär källa.

## Persistent volume är inte backup

En persistent volume gör att data överlever poddens livscykel. Den skyddar inte automatiskt mot logiska fel.

Exempel på fel som persistent volume inte löser:

- tabeller raderas av misstag
- applikationen skriver korrupt data
- felaktigt batchjobb ändrar många poster
- ransomware-liknande påverkan
- index blir inkonsistent
- volymen raderas
- storage-systemet får större fel
- en uppgradering förstör dataformat

Därför måste varje persistent volume kopplas till en återställningsstrategi, eller tydligt klassas som återskapbar.

## Återskapa eller återställa?

Alla komponenter bör inte återställas från backup. Vissa bör återskapas från källor.

| Komponent | Bäst strategi |
|---|---|
| Applikationsdeployment | Återskapa från GitOps eller pipeline |
| Container image | Återskapa från build eller hämta från registry |
| ConfigMap | Återskapa från deklarativ konfiguration |
| Secret | Återställ eller rotera kontrollerat |
| Primärdatabas | Restore från backup eller replik |
| Sökindex | Reindexera från primär källa om möjligt |
| Cache | Töm och låt återskapas |
| Temporär kö | Beror på affärsbetydelse |
| Eventlogg | Behöver särskild retention och replikering |
| Objekt | Backup eller replikering beroende på krav |

En viktig arkitekturprincip är att minimera mängden unik och oersättlig information i plattformens runtime.

## Återställningsordning

Vid en större incident är ordningen viktig. Om komponenter startas i fel ordning kan återställningen misslyckas eller skapa nya fel.

En möjlig ordning:

1. Grundläggande infrastruktur och nätverk.
2. OpenShift-kluster eller alternativ miljö.
3. Plattformskonfiguration: namespaces, RBAC, policies, storage classes.
4. Secrets och certifikat.
5. Databaser och primära datakällor.
6. Messagingplattformar och eventflöden.
7. Objektlagring.
8. Applikationer och API:er.
9. Sekundära index och läsmodeller.
10. Observability och larm.
11. Extern trafik och användaråtkomst.

Ordningen kan variera, men den ska vara dokumenterad och testad för kritiska flöden.

## Databaser

Databaser är ofta de mest kritiska komponenterna i DR-planen. För Oracle kan organisationen redan ha etablerade processer. För PostgreSQL eller andra nya databaser måste motsvarande krav definieras.

Frågor:

- hur ofta tas backup?
- är backup applikationskonsistent?
- hur återställs databasen?
- hur lång tid tar restore?
- hur verifieras datakvalitet?
- hur testas restore?
- hur hanteras schemaändringar?
- hur hanteras replikering?
- vem äger återställningen?
- hur kopplas restore till applikationernas releaseversion?

Databasbackup är inte bara DBA-fråga. Den påverkar applikationsarkitektur, release och verksamhetskontinuitet.

## Messaging och köer

Messaging är särskilt svårt vid återställning eftersom meddelanden kan representera arbete som är pågående, delvis utfört eller väntande.

Frågor:

- behöver köinnehåll bevaras?
- kan meddelanden återskapas från primär källa?
- vad händer med dubbletter?
- är konsumenter idempotenta?
- finns dead-letter queues?
- hur återstartas konsumenter?
- hur hanteras consumer offsets?
- finns risk att gamla meddelanden behandlas efter restore?
- vilka flöden måste pausas under återställning?

För IBM MQ-flöden kan etablerade rutiner finnas. För nya brokers eller event streaming behöver organisationen skapa motsvarande operativ modell.

## Elasticsearch

Elasticsearch bör ofta vara sekundärt index. Det betyder att återställningsstrategin kan vara reindexering snarare än traditionell backup.

Men detta kräver att:

- primär källa finns kvar
- reindexeringsprocessen är dokumenterad
- indexschema är versionerat
- reindexeringstid är acceptabel
- verksamheten förstår eventuell nedtid eller begränsad funktion
- indexeringsfel kan följas upp

I vissa fall kan snapshots behövas för snabbare återställning. Men även då bör arkitekten fråga om indexet kan byggas om från primär källa.

## Objektlagring och filer

Objekt och filer kan vara affärskritiska. Dokument, bilagor, exporter och arkiv måste skyddas utifrån dataklassning och retention.

Frågor:

- vilka objekt är affärskritiska?
- finns replikering?
- finns backup?
- hur hanteras radering?
- finns versionering?
- hur återställs ett enskilt objekt?
- hur återställs en hel bucket eller motsvarande?
- hur kopplas objekt till metadata i databas?
- hur hanteras inkonsistens mellan objekt och metadata?

Objektlagring löser mycket, men kräver tydlig livscykelstyrning.

## Secrets och certifikat

Secrets är särskilt känsliga i återställning. Att återställa gamla credentials kan skapa säkerhetsrisk, medan att rotera allt kan skapa avbrott om applikationer inte är förberedda.

Strategier:

- restore av secrets från säker backup
- kontrollerad rotation efter incident
- certifikathantering via automatiserad process
- dokumenterad beroendekarta
- test av secret rotation
- separata rutiner för produktionscredentials

Secrets bör aldrig vara den enda odokumenterade delen av återställningsplanen.

## Plattformskonfiguration

OpenShift-konfiguration bör så långt möjligt vara deklarativ och versionsstyrd. Det gör återställning enklare.

Exempel:

- namespaces
- RBAC
- network policies
- resource quotas
- deploymentkonfiguration
- routes
- service accounts
- policyer
- operators
- monitoringkonfiguration

Om konfigurationen bara finns i klustret blir klustret en unik snöflinga. Då blir DR svårare.

## Kontinuitetsnivåer

Alla system behöver inte samma nivå av kontinuitet. En praktisk modell är att klassificera tjänster.

| Klass | Exempel | Typiska krav |
|---|---|---|
| Kritisk | Order, betalning, kundpåverkande flöden | Lågt RPO/RTO, testad DR |
| Viktig | Interna arbetsflöden, kundtjänststöd | Tydlig restore, viss nedtid acceptabel |
| Stödjande | Rapporter, interna verktyg | Längre RTO kan accepteras |
| Återskapbar | Cache, sekundärt index | Rebuild snarare än restore |

Klassificeringen hjälper organisationen att lägga rätt nivå av investering på rätt plats.

## Verksamhetsläge vid degradering

Kontinuitet handlar inte alltid om att allt ska fungera perfekt direkt. Ibland behöver organisationen definiera degraderade lägen.

Exempel:

- ordersökning är tillfälligt begränsad men orderläggning fungerar
- kundtjänst kan läsa äldre data men inte senaste indexerade vy
- asynkrona notifieringar pausas medan kärntransaktioner fortsätter
- rapportering är fördröjd men operativ drift fungerar
- vissa interna funktioner stängs för att skydda kundpåverkande flöden

Degraderade lägen ska vara medvetna beslut, inte improvisation under incident. De bör dokumenteras tillsammans med affärsägare, så att teknisk återställning prioriterar rätt verksamhetsförmåga.

## DR-test och övningar

DR-planer som inte testas fungerar sällan som tänkt. Test behöver inte alltid vara fullskaliga, men de måste vara verklighetsnära.

Typer av test:

- restore-test av databas
- reindexeringstest för Elasticsearch
- återställning av namespace från GitOps
- secret rotation-test
- failover-test för kritisk tjänst
- tabletop-övning med incidentroller
- test av återställningsordning
- test av kommunikation och beslutsvägar

Efter varje test bör runbooks, automation och arkitekturprinciper uppdateras.

## Beslutsguide: vilken återställningsstrategi?

| Scenario | Rekommenderad strategi |
|---|---|
| Primär affärsdata | Backup, replikering och testad restore |
| Sekundärt sökindex | Reindexering, eventuellt snapshot för snabbhet |
| Cache | Återskapa automatiskt |
| Objekt och dokument | Backup, replikering och metadata-konsistens |
| Deploymentkonfiguration | GitOps eller deklarativ restore |
| Secrets | Säker restore eller kontrollerad rotation |
| MQ-flöden | Bevara eller återskapa enligt affärssemantik |
| Temporära filer | Ingen restore om de är verkligt temporära |
| Plattformspolicyer | Återskapa från versionsstyrd konfiguration |

## Beslutsguide: prioritera kontinuitetsarbete

| Fråga | Hög prioritet om |
|---|---|
| Påverkar tjänsten kund eller intäkt? | Ja |
| Är data oersättlig? | Ja |
| Finns regulatoriska krav? | Ja |
| Är återställning oprövad? | Ja |
| Finns många beroenden? | Ja |
| Är felet svårt att upptäcka? | Ja |
| Är komponenten stateful? | Ja |
| Saknas ägare? | Ja |

## Tradeoffs

| Val | Fördel | Risk |
|---|---|---|
| Lågt RPO/RTO för allt | Hög ambitionsnivå | Mycket dyrt och komplext |
| Klassificera tjänster | Rätt investering per behov | Kräver verksamhetsdialog |
| Reindexera i stället för snapshot | Mindre backupberoende | Längre återställningstid |
| Snapshot av index | Snabbare restore | Kan dölja beroende till sekundär data |
| GitOps för konfiguration | Reproducerbarhet | Kräver disciplin |
| Manuell återställning | Flexibilitet | Personberoende och långsamt |
| Secret rotation efter incident | Bättre säkerhet | Kan ge längre återställning |
| Återställa gamla secrets | Snabbare | Risk om credentials är komprometterade |

## Anti-patterns

### Backup utan restore-test

Backup som aldrig återställts är inte verifierad återställningsförmåga.

### Allt behandlas som lika kritiskt

Om alla system får samma kontinuitetskrav blir planen antingen för dyr eller för ytlig.

### Sekundära index skyddas som primärdata

Om Elasticsearch är sekundärt bör reindexering övervägas innan dyr backupmodell införs.

### Klustret som enda källa

Om deploymentkonfiguration bara finns i OpenShift och inte i Git blir återställning svår.

### DR-plan utan verksamhetskoppling

Teknisk återställning räcker inte om den inte stödjer verksamhetens prioriterade processer.

## Vanliga fallgropar

- Att tro att persistent volume är backup.
- Att inte sätta RPO och RTO per tjänst.
- Att sakna återställningsordning.
- Att glömma secrets och certifikat.
- Att inte testa restore av nya databaser.
- Att inte veta om kömeddelanden ska bevaras.
- Att inte kunna återskapa Elasticsearch-index.
- Att sakna metadata-konsistens för objektlagring.
- Att inte dokumentera vem som beslutar vid DR.
- Att inte öva kommunikation under incident.

## Arkitektens checklista

Innan kontinuitetsmodellen godkänns bör arkitekten kunna svara på:

- Vilka tjänster är kritiska?
- Vilket RPO gäller per tjänst?
- Vilket RTO gäller per tjänst?
- Vilka data är primär sanning?
- Vilka data kan återskapas?
- Finns backup?
- Har restore testats?
- Hur återställs secrets?
- Hur återställs deploymentkonfiguration?
- Hur hanteras MQ och eventflöden?
- Hur återskapas Elasticsearch-index?
- Hur skyddas objektlagring?
- Vilken är återställningsordningen?
- Vem leder DR-arbetet?
- Hur dokumenteras och övas planen?

## Koppling till moderniseringscaset

Nordisk Handel AB klassificerar sina första moderniserade flöden. Orderkärnan får låg tolerans för dataförlust och kort återställningstid. Ordersökningen som bygger på Elasticsearch får ett annat krav: indexet får vara otillgängligt kortare tid och kan återskapas från Oracle och händelseflöden.

Organisationen dokumenterar därför olika strategier:

- Oracle skyddas via etablerad backup och restore.
- PostgreSQL för nya tjänster kräver testad restore innan produktion.
- Elasticsearch-index ska kunna reindexeras.
- MQ-flöden analyseras utifrån om meddelanden ska bevaras eller kan återskapas.
- OpenShift-konfiguration ska finnas deklarativt.
- Secrets ska kunna roteras kontrollerat.
- DR-test läggs in i roadmapen.

Detta gör kontinuitet till en del av arkitekturen, inte en separat driftbilaga.

## Snabb sammanfattning

- Backup är inte samma sak som restore eller disaster recovery.
- RPO och RTO måste sättas per tjänst och dataroll.
- Persistent volume är inte backup.
- Vissa komponenter ska återställas, andra ska återskapas.
- Återställningsordning behöver dokumenteras och testas.
- Databaser, messaging, index, objektlagring och secrets kräver olika strategier.
- DR-planer måste övas för att bli verklig förmåga.

## Nästa steg

Nästa kapitel behandlar strangler pattern och stegvis migrering. Där går vi från kontinuitet och återställning till hur organisationen praktiskt kan ersätta legacy-funktionalitet utan big bang.
