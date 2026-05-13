# Inledning

## Om boken

Den här boken handlar om modernisering av enterprise-applikationer mot en mer cloud native-arkitektur med fokus på Red Hat-stacken. Den utgår från en miljö där Java EE, Oracle och IBM MQ har varit centrala delar av applikations- och integrationslandskapet, och där organisationen vill röra sig mot containers, OpenShift, tydligare plattformsstyrning och mer ändringsbar arkitektur.

Boken är inte en installationsmanual för OpenShift och inte heller en komplett referens för varje teknik som nämns. Den är en arkitekturhandbok. Syftet är att hjälpa läsaren förstå vilka beslut som behöver fattas, vilka tradeoffs som finns och hur en modernisering kan genomföras stegvis utan att tappa kontroll över säkerhet, data, drift och kontinuitet.

Tyngdpunkten ligger på Podman, OpenShift, containerteknik, Java EE-modernisering, databasstrategier, kö- och eventmönster, Elasticsearch, Ceph, observability, governance och praktiska beslutsmodeller.

## Målgrupp

Boken riktar sig främst till lösningsarkitekter och erfarna Java EE-utvecklare som arbetar i eller nära enterprise-miljöer. Läsaren antas förstå grundläggande applikationsarkitektur, Java-baserade system, databaser och integrationer, men behöver inte vara expert på OpenShift eller cloud native-plattformar.

Boken är särskilt relevant för dig som:

- arbetar med modernisering av äldre Java EE-applikationer
- behöver förstå hur OpenShift påverkar arkitektur- och driftbeslut
- vill minska beroenden till central Oracle- och IBM MQ-baserad arkitektur
- behöver avgöra när PostgreSQL, Elasticsearch, Ceph eller eventdrivna mönster passar
- ansvarar för målarkitektur, teknikval, governance eller plattformsstrategi
- vill kunna resonera om modernisering utan att fastna i produktjämförelser

## Hur boken bör läsas

Boken är skriven som en sammanhängande arkitekturresa. De första kapitlen etablerar varför modernisering behövs och hur containerplattformen fungerar. Därefter behandlas data, integration, lagring, drift, säkerhet och kontinuitet. De senare kapitlen fokuserar på genomförande, Java-modernisering, transaktioner, observability, teknikbeslut och roadmap.

Boken kan läsas från början till slut, men den fungerar också som uppslagsverk. Om du arbetar med ett specifikt beslut kan du gå direkt till relevant kapitel, till exempel:

- Kapitel 5 för Oracle, PostgreSQL och dataarkitektur
- Kapitel 6 för IBM MQ, köer och eventdrivna mönster
- Kapitel 7 för Elasticsearch som sekundärt index
- Kapitel 8 för Ceph och stateful workloads
- Kapitel 12 för Java EE till cloud native Java
- Kapitel 15 för ADR:er och teknikval
- Kapitel 16 för målarkitektur och roadmap

## Moderniseringscaset

För att hålla resonemangen konkreta återkommer boken till ett fiktivt moderniseringscase: Nordisk Handel AB. Organisationen har ett klassiskt enterprise-landskap med Java EE-applikationer, Oracle som central dataplattform, IBM MQ för viktiga integrationsflöden och ett växande behov av snabbare förändringstakt.

Caset används för att visa hur beslut kan se ut i praktiken. Det är inte tänkt som en exakt referensarkitektur för alla organisationer, utan som ett pedagogiskt scenario där olika vägval blir tydliga.

## Vad boken inte försöker göra

Boken försöker inte ge en komplett produktjämförelse mellan alla databaser, brokers, sökmotorer eller lagringsplattformar. Den ger inte heller fullständiga installationsguider eller detaljerade kommandoreferenser.

Den fokuserar i stället på frågor som:

- Vad är rätt roll för en teknik i målarkitekturen?
- När bör en befintlig komponent behållas, kapslas in, kompletteras eller ersättas?
- Vilka beslut bör dokumenteras med ADR?
- Vilka risker uppstår när legacy-mönster flyttas in i containerplattformen?
- Hur kan modernisering ske stegvis och kontrollerat?

## Terminologi och språk

Boken är skriven på svenska men använder vedertagna engelska begrepp där svenska motsvarigheter saknas eller riskerar att bli otydliga. Exempel är cloud native, workload, registry, runtime, broker, event streaming, observability, tracing, golden path och fitness function.

När ett begrepp är centralt för boken introduceras det i sitt sammanhang och återanvänds konsekvent. Terminologi och definitioner samlas även i projektets terminologifil.

## Grundantaganden

Boken bygger på några återkommande antaganden:

- Modernisering bör ske stegvis snarare än som big bang.
- OpenShift bör behandlas som en intern plattformsprodukt, inte bara som ett kluster.
- Containers löser inte automatiskt arkitektur-, data- eller driftproblem.
- Oracle och IBM MQ kan fortfarande vara rätt val i vissa roller.
- Nya datalager och integrationsmönster kräver tydligt ägarskap.
- Stateful workloads kräver backup, restore, observability och driftmodell.
- Säkerhet och governance ska byggas in i golden paths.
- Observability är en förutsättning för distribuerade flöden.
- Teknikval ska dokumenteras, följas upp och kunna omprövas.

## Rekommenderat resultat efter läsning

Efter boken bör läsaren kunna beskriva en rimlig målarkitektur för modernisering mot Red Hat-stack, identifiera lämpliga första migreringskandidater, formulera teknikbeslut med tydliga tradeoffs och bygga en roadmap som balanserar affärsvärde, risk, driftbarhet och avveckling av gammal komplexitet.
