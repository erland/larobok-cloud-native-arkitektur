# Kapitel 16: Målarkitektur och roadmap

## Varför detta kapitel finns

Boken har byggt upp en arkitekturell verktygslåda för modernisering från traditionell Java EE-, Oracle- och IBM MQ-baserad enterprise-miljö mot en mer cloud native Red Hat-orienterad målbild. Vi har behandlat containerteknik, OpenShift, säkerhet, data, messaging, Elasticsearch, Ceph, stateful workloads, driftsmodell, kontinuitet, strangler pattern, Java-modernisering, transaktioner, observability och teknikbeslut.

Det här avslutande kapitlet knyter ihop delarna till en praktisk målarkitektur och roadmap. Målet är inte att ge en enda universell slutbild. Målet är att visa hur organisationen kan beskriva sin riktning, prioritera nästa steg och undvika att moderniseringsarbetet blir en samling separata teknikinitiativ.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- beskriva en sammanhängande målarkitektur för Red Hat-baserad cloud native-modernisering
- formulera principer för hur Oracle, IBM MQ, Elasticsearch, Ceph, Podman och OpenShift samspelar
- skapa en stegvis roadmap från pilot till bredare införande
- prioritera moderniseringsinitiativ utifrån värde, risk och beroenden
- identifiera vilka beslut som behöver tas tidigt och vilka som kan vänta
- formulera nästa steg efter bokens slut

## Målarkitektur som riktning, inte slutritning

En målarkitektur ska ge riktning, men den får inte bli en låst ritning som hindrar lärande. I modernisering är det ofta bättre att beskriva principer, standardmönster och gränser än att försöka rita alla framtida komponenter i detalj.

En bra målarkitektur beskriver:

- vilka plattformsförmågor organisationen ska ha
- vilka teknikval som är standard eller rekommendation
- vilka legacy-komponenter som behålls initialt
- vilka beroenden som ska kapslas in
- hur nya tjänster ska byggas
- hur dataägarskap ska utvecklas
- hur integrationer ska klassificeras
- hur säkerhet och governance fungerar
- hur driftbarhet och kontinuitet ska säkerställas
- hur beslut dokumenteras och omprövas

Målarkitekturen ska hjälpa team fatta bättre beslut i vardagen.

## Övergripande målbild

I bokens scenario rör sig Nordisk Handel AB från en centraliserad Java EE-, Oracle- och IBM MQ-miljö till en målbild där OpenShift är intern applikationsplattform och där nya tjänster byggs mer cloud native.

Den övergripande målbilden kan beskrivas så här:

- OpenShift är målplattform för nya och moderniserade containeriserade applikationer.
- Podman används för lokal containerförståelse och verifiering.
- Oracle behålls där den äger affärskritisk primär sanning.
- Direkt schemaåtkomst från nya tjänster begränsas.
- PostgreSQL kan användas för nya avgränsade relationsbehov.
- IBM MQ behålls för kritiska legacy-flöden men kapslas in där nya tjänster annars skulle kopplas hårt.
- Eventdrivna mönster införs där domänhändelser, flera konsumenter eller återspel ger värde.
- Elasticsearch används som sekundärt index för sök och operativa läsmodeller.
- Ceph och annan plattformsnära lagring används där driftmodell, backup och restore är tydliga.
- Observability, security och governance byggs in i golden paths.
- Modernisering sker stegvis genom strangler pattern och tydliga avvecklingsplaner.

## Plattformslagret

Plattformslagret består av OpenShift, container registry, image-policy, secrets-hantering, logging, metrics, tracing, nätverkspolicyer, storage classes och plattformsautomatisering.

Målet är inte att varje team ska förstå alla interna detaljer i plattformen. Målet är att team ska kunna använda plattformen genom tydliga golden paths.

Plattformslagret bör erbjuda:

- namespace-modell
- godkända bas-images
- CI/CD- eller GitOps-mönster
- secrets-standard
- standard för ConfigMaps
- resource requests och limits
- ingress och route-modell
- observability-standard
- security guardrails
- storage classes med dokumenterade egenskaper
- onboarding och supportmodell

OpenShift blir då en intern produkt, inte bara ett kluster.

## Applikationslagret

Applikationslagret består av befintliga Java EE-applikationer, anpassade Java-applikationer och nya cloud native Java-tjänster.

Målbilden bör skilja mellan tre kategorier:

| Kategori | Strategi |
|---|---|
| Legacy Java EE med hög risk | Behåll, kapsla in och modernisera stegvis |
| Java-applikationer som ska leva vidare | Containerisera och anpassa för OpenShift |
| Nya eller utbrutna funktioner | Bygg cloud native från start |

Det viktiga är att inte tvinga alla applikationer till samma moderniseringsnivå. Rätt nivå beror på livslängd, affärsvärde, risk, kopplingar och teammognad.

## Datalagret

Datalagret är ofta den svåraste delen av målarkitekturen. Oracle kan inte bara betraktas som en produkt som ska bytas. Den kan äga primär affärssanning.

Målbilden bör därför bygga på dataroller:

| Dataroll | Rekommenderad riktning |
|---|---|
| Primär kärntransaktion | Behåll Oracle initialt där risk är hög |
| Ny avgränsad relationsdata | PostgreSQL kan vara lämpligt |
| Sök och filtrering | Elasticsearch som sekundärt index |
| Dokument och filer | Objektlagring |
| Rapportering och analys | Sekundär läsmodell eller analysplattform |
| Cache | Återskapbar cache, inte primär sanning |

Den viktigaste principen är att primär sanning ska vara tydlig. Nya datalager får inte skapa oavsiktliga dubbla sanningar.

## Integrationslagret

Integrationslagret behöver gå från teknikcentrerat till mönstercentrerat. IBM MQ kan fortfarande vara rätt för kritiska legacy-flöden, men nya integrationsbeslut bör börja med frågan vilket mönster som behövs.

| Behov | Mönster |
|---|---|
| Be någon göra något | Kommando |
| Informera om något som hänt | Domänhändelse |
| Flera konsumenter behöver reagera | Publish/subscribe eller event streaming |
| Arbete ska köas internt | Kö eller broker |
| Legacy kräver befintlig MQ | Behåll eller kapsla in |
| Synkron fråga och svar behövs | API |

Målbilden ska inte vara “allt eventdrivet”. Den ska vara “rätt integrationsmönster för rätt behov”.

## Säkerhet och governance

Säkerhet ska inte vara ett separat granskningssteg i slutet. Den ska vara inbyggd i plattformens standarder.

Målbilden bör innehålla:

- least privilege
- RBAC-modell
- dedikerade servicekonton där det behövs
- standardiserad secrets-hantering
- image scanning
- godkända bas-images
- nätverkspolicyer
- regler för externa routes
- audit och spårbarhet
- undantagsprocess
- ADR för större avvikelser

Governance ska göra rätt väg enkel, inte skapa nya flaskhalsar.

## Drift och kontinuitet

Målarkitekturen måste inkludera drift. En tjänst som inte går att observera, återställa eller äga är inte produktionsklar.

Driftmålbilden bör omfatta:

- produktionsklarhetskrav
- runbooks
- incidentprocess
- supportmodell
- SLI och SLO för kritiska tjänster
- loggar, metrics och tracing
- korrelations-id
- backup och restore
- RPO och RTO
- DR-test
- release- och rollbackstrategi

Detta gäller särskilt stateful workloads, köflöden, sökindex och databaser.

## Roadmap i fyra faser

En praktisk moderniseringsroadmap kan delas in i fyra faser.

### Fas 1: Grundläggande plattformsförmåga

Målet är att göra OpenShift användbart och säkert för pilotteam.

Fokus:

- namespace-modell
- bas-images
- image-policy
- secrets-standard
- logging och metrics
- grundläggande RBAC
- Podman-baserad containerverifiering
- första golden path för Java-tjänst
- ADR-process
- första teknikradar

Resultat: organisationen kan onboarda begränsade pilotteam utan att varje team uppfinner egen plattformsmodell.

### Fas 2: Kontrollerad pilot

Målet är att testa målarkitekturen i ett verkligt men avgränsat use case.

Fokus:

- välj strangler-kandidat
- bygg ny tjänst på OpenShift
- kapsla in legacy där det behövs
- använd Elasticsearch som sekundärt index om sök är första case
- mät observability end-to-end
- dokumentera beroenden till Oracle och IBM MQ
- skapa runbook
- definiera rollback
- testa incidenthantering

Resultat: organisationen lär sig hur teknik, drift och governance fungerar tillsammans.

### Fas 3: Breddning med standarder

Målet är att fler team ska kunna använda plattformen utan att komplexiteten ökar okontrollerat.

Fokus:

- fler golden paths
- tydligare supportmodell
- standardiserade ADR-mallar
- teknikradar i aktiv användning
- PostgreSQL-modell för avgränsade domäner
- tydligare messagingbeslut
- stateful workload-beslutsprocess
- SLO:er för kritiska tjänster
- återkommande DR- och restore-test

Resultat: plattformen går från pilot till återanvändbar intern produkt.

### Fas 4: Avveckling och optimering

Målet är att minska dubbel komplexitet.

Fokus:

- avveckla gamla sökvägar
- ta bort direkt schemaåtkomst där den ersatts
- stäng gamla MQ-flöden som inte längre behövs
- minska serverbunden Java EE-konfiguration
- optimera kostnad och kapacitet
- förbättra SLO:er
- minska toil
- uppdatera teknikradar och ADR:er
- ompröva målarkitekturen

Resultat: moderniseringen ger faktisk förenkling, inte bara nya lager ovanpå gamla.

## Prioriteringsmodell

Allt kan inte göras samtidigt. Prioritering bör väga affärsvärde, risk, beroenden och lärande.

| Faktor | Fråga |
|---|---|
| Affärsvärde | Skapar detta synlig nytta? |
| Riskreduktion | Minskar detta teknisk eller operativ risk? |
| Lärande | Lär vi oss något som kan återanvändas? |
| Beroenden | Blockerar detta andra initiativ? |
| Genomförbarhet | Kan teamet lyckas inom rimligt scope? |
| Avveckling | Minskar detta gammal komplexitet? |
| Driftbarhet | Blir tjänsten lättare att äga? |

En bra första roadmap balanserar snabba vinster med fundament som krävs för hållbar modernisering.

## Beslut som måste tas tidigt

Vissa beslut bör tas tidigt eftersom de påverkar många andra delar:

- namespace-modell
- secrets-standard
- image-policy
- bas-images
- loggingstandard
- health check-krav
- ADR-process
- extern exponeringsmodell
- minimikrav för produktionsklarhet
- direkt schemaåtkomst-policy
- klassificering av IBM MQ-flöden
- stateful workload-process

Andra beslut kan vänta tills användningsfall kräver dem. Exempel är full event streaming-plattform, avancerad service mesh eller generell dataplattform för analys.

## Mät framgång

Modernisering bör mätas i förmågor, inte bara antal migrerade applikationer.

Exempel på mätetal:

- lead time från ändring till produktion
- antal tjänster med runbook
- antal tjänster med SLO
- andel deployments via standardiserad pipeline
- tid att felsöka incident
- antal direkta schemaåtkomster som avvecklats
- antal gamla MQ-flöden som klassificerats
- andel images från godkända bas-images
- antal restore-testade stateful workloads
- antal aktiva undantag med slutdatum
- antal avvecklade legacy-vägar

Målet är inte att maximera aktivitetsmätetal. Målet är att visa förbättrad förändringsförmåga, driftbarhet och riskkontroll.

## Beslutsguide: nästa bästa steg

| Nuläge | Nästa bästa steg |
|---|---|
| OpenShift finns men används olika | Definiera golden path och plattformsprodukt |
| Många Java EE-appar är okartlagda | Kartlägg serverberoenden och livslängd |
| Oracle är gemensam integrationsyta | Begränsa ny direktåtkomst och skapa inkapsling |
| IBM MQ används för allt | Klassificera flöden i kommando, event och arbetskö |
| Sök belastar Oracle | Inför sekundärt index med tydlig primär källa |
| Stateful workloads efterfrågas | Definiera backup, restore och storage class-process |
| Incidenter tar lång tid | Inför korrelations-id och end-to-end-observability |
| Teknikval är spretiga | Inför ADR och teknikradar |
| Modernisering saknar effekt | Mät ledtid, driftbarhet och avveckling |

## Beslutsguide: när är moderniseringen mogen?

| Fråga | Mognadssignal |
|---|---|
| Kan team deploya säkert utan specialhjälp? | Golden paths fungerar |
| Kan system felsökas över tjänstegränser? | Observability fungerar |
| Kan nya datalager väljas kontrollerat? | Datagovernance och ADR finns |
| Kan legacy kapslas in stegvis? | Strangler pattern används |
| Kan stateful workloads återställas? | Restore-test finns |
| Kan säkerhet följas utan manuella köer? | Guardrails och policyer finns |
| Kan gamla lösningar avvecklas? | Avvecklingsplaner genomförs |
| Kan beslut omprövas? | ADR och teknikradar hålls levande |

## Anti-patterns

### Målarkitektur utan roadmap

En målbild utan steg, ansvar och prioritering blir svår att realisera.

### Roadmap utan avveckling

Om roadmapen bara lägger till nytt men aldrig tar bort gammalt ökar komplexiteten.

### Plattform utan produktägarskap

OpenShift som kluster utan intern produktmodell ger begränsat värde.

### Allt standardiseras för tidigt

För hård standardisering innan organisationen lärt sig kan låsa in dåliga beslut.

### Allt blir pilot

Piloter utan produktion, support och avveckling ger inte verklig modernisering.

## Arkitektens slutchecklista

Innan moderniseringsprogrammet går vidare bör arkitekten kunna svara på:

- Vilken målbild gäller?
- Vilka principer styr nya tjänster?
- Vad är standard på OpenShift?
- Vilka legacy-delar behålls initialt?
- Var finns primär sanning?
- Hur hanteras direkt schemaåtkomst?
- Vilka MQ-flöden är klassificerade?
- När används event streaming?
- När används Elasticsearch?
- När används Ceph eller annan stateful lagring?
- Vad betyder produktionsklar?
- Hur mäts observability?
- Hur dokumenteras beslut?
- Hur hanteras undantag?
- Vilka avvecklingsmål finns?
- Vad är nästa tre prioriterade initiativ?

## Koppling till moderniseringscaset

Nordisk Handel AB sammanfattar sin första roadmap:

1. Etablera OpenShift som intern plattformsprodukt.
2. Skapa golden path för Java-tjänster.
3. Kartlägga Java EE-applikationer, Oracle-beroenden och MQ-flöden.
4. Välja orderhistoriksökning som strangler-kandidat.
5. Införa Elasticsearch som sekundärt index med reindexeringsstrategi.
6. Behålla Oracle som primär sanning för orderkärnan.
7. Införa korrelations-id och observability end-to-end.
8. Dokumentera PostgreSQL, event streaming och Ceph som kontrollerade trial-områden.
9. Införa ADR och teknikradar.
10. Planera avveckling av gamla sökvägar när pilotkvalitet är bevisad.

Roadmapen är inte maximal. Den är genomförbar. Det är viktigare att skapa en hållbar moderniseringsförmåga än att lova total transformation direkt.

## Snabb sammanfattning

- Målarkitektur ska ge riktning, inte låsa varje detalj.
- OpenShift bör fungera som intern plattformsprodukt.
- Oracle, IBM MQ, Elasticsearch och Ceph ska förstås utifrån roller, inte som enkla byt-ut-objekt.
- Roadmapen bör gå från plattformsgrund till pilot, breddning och avveckling.
- Modernisering ska mätas i förändringsförmåga, driftbarhet och riskreduktion.
- Beslut, undantag och teknikval behöver vara spårbara.
- Den verkliga nyttan uppstår när gamla vägar kan avvecklas.

## Avslutning

Cloud native-modernisering är inte en rak migrering från gammalt till nytt. Det är en stegvis förflyttning av arkitektur, plattform, drift och ansvar. För organisationer med Java EE, Oracle och IBM MQ handlar framgången inte om att byta bort allt gammalt så snabbt som möjligt. Den handlar om att bygga förmågan att förändra säkert.

Podman gör containerantaganden synliga. OpenShift ger en plattform för standardiserad drift. Oracle och IBM MQ kan behållas där de fortfarande löser rätt problem. PostgreSQL, event streaming, Elasticsearch och Ceph kan införas där deras roller är tydliga. Observability, security, governance och ADR:er gör arkitekturen hanterbar över tid.

Det viktigaste resultatet är inte en perfekt målbild. Det viktigaste resultatet är en organisation som kan fatta bättre beslut, leverera oftare, återställa säkrare och modernisera utan att tappa kontroll.
