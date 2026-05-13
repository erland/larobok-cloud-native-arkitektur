# Terminologi

| Begrepp | Rekommenderad svensk användning | Kort definition |
|---|---|---|
| Cloud native | cloud native | Arkitektur- och driftmodell som utnyttjar automatisering, skalbarhet, resiliens och plattformsförmågor. |
| Container image | container image / containeravbild | Paketerad applikation med beroenden och metadata. |
| Container runtime | container runtime | Programvara som kör containrar. |
| OpenShift | OpenShift | Red Hats Kubernetes-baserade enterprise-plattform. |
| Podman | Podman | Verktyg för att bygga och köra containrar, ofta utan daemon och med stöd för rootless containers. |
| Operator | operator | Kubernetes-mönster för att automatisera drift av komplexa applikationer. |
| Stateful workload | stateful workload | Arbetslast som behöver bevara tillstånd över tid. |
| Event streaming | event streaming | Kontinuerlig publicering och konsumtion av händelser. |
| Message broker | message broker | System som förmedlar meddelanden mellan producenter och konsumenter. |
| Observability | observability | Förmåga att förstå systemets beteende via loggar, metrics och tracing. |
| ADR | Architectural Decision Record | Kort dokument som beskriver ett arkitekturbeslut, alternativ och konsekvenser. |

| Containerized | containerized / containeriserad | Applikation som paketerats och körs i container utan att nödvändigtvis ha anpassats till cloud native-principer. |
| Guardrails | guardrails / styrande ramar | Standarder, policyer och plattformsregler som möjliggör självservice utan att tappa kontroll. |
| Moderniseringsdrivare | moderniseringsdrivare | Tekniska eller organisatoriska skäl som motiverar förändring av arkitektur, plattform eller arbetssätt. |
| Lift and shift | lift and shift | Migrering där system flyttas till ny miljö med minimal arkitekturell förändring. |

| Sekundärt sökindex | sekundärt sökindex | Index optimerat för sök och läsning utan att äga primär affärssanning. |
| Primär sanning | primär sanning | Datakälla som äger korrekt affärstillstånd. |
| Reindexering | reindexering | Process för att återskapa ett index från primär källa eller händelseflöde. |
| Retention | retention | Regler för hur länge data behålls innan den arkiveras eller raderas. |
| CDC | Change Data Capture / CDC | Mönster för att fånga förändringar från databasens förändringsflöde. |
| Eventual consistency | eventual consistency | Konsistensmodell där sekundära vyer blir uppdaterade över tid snarare än omedelbart. |


| Blocklagring | block storage / blocklagring | Lagring som exponeras som blockenhet, ofta för databaser och stateful workloads. |
| Fillagring | file storage / fillagring | Lagring som exponeras som filsystem, ibland delat mellan workloads. |
| Persistent volume | persistent volume | Plattformsresurs för lagring som överlever poddens livscykel. |
| Storage class | storage class | Klassificering av lagring med definierade egenskaper som prestanda, kostnad och skyddsnivå. |
| RPO | Recovery Point Objective / RPO | Mål för hur mycket data som maximalt får gå förlorad. |
| RTO | Recovery Time Objective / RTO | Mål för hur snabbt en tjänst eller data ska kunna återställas. |
| Ephemeral storage | ephemeral storage / tillfällig lagring | Lagring som kan försvinna när workloaden ersätts. |


| Strangler pattern | strangler pattern | Moderniseringsmönster där ny funktionalitet gradvis ersätter delar av ett befintligt system. |
| Inkapsling | inkapsling | Minskning av direkta beroenden genom API, adapter, fasad eller kontrollerat gränssnitt. |
| Samexistens | samexistens | Period där gammal och ny lösning körs parallellt under kontrollerade former. |
| Trafikstyrning | trafikstyrning | Mekanism för att styra trafik mellan gammal och ny lösning. |
| Shadow traffic | shadow traffic | Trafik skickas till ny lösning för jämförelse utan att påverka faktiskt resultat. |
| Dubbel skrivning | dubbel skrivning | Data skrivs till två modeller eller system parallellt, ofta riskfyllt. |


| Korrelations-id | correlation id / korrelations-id | Gemensamt id som följer ett flöde genom flera system. |
| SLI | Service Level Indicator / SLI | Mätetal för tjänstens beteende. |
| SLO | Service Level Objective / SLO | Målnivå för ett SLI. |
| Runbook | runbook | Operativ instruktion för felsökning och åtgärd. |
| Tracing | tracing | Spårning av ett flöde genom flera komponenter. |
| Larmtrötthet | alert fatigue / larmtrötthet | Tillstånd där för många eller irrelevanta larm gör att viktiga larm missas. |


| Fitness function | fitness function | Mätbart kriterium som visar om arkitekturen uppfyller önskade egenskaper. |
| Teknikradar | teknikradar | Modell för att visa vilka tekniker som ska användas, prövas, utvärderas eller undvikas. |
| Beslutstyp | beslutstyp | Klassificering av beslut som standard, rekommendation, undantag, experiment eller förbud. |
| Governance | governance / styrning | Regler och processer som gör beslut konsekventa, spårbara och förvaltningsbara. |
| Undantagsprocess | undantagsprocess | Process för att hantera motiverade avvikelser från standard. |


| Målarkitektur | target architecture / målarkitektur | Styrande riktning för framtida arkitektur. |
| Roadmap | roadmap | Tidsatt plan för stegvis införande och modernisering. |
| Golden path | golden path | Rekommenderad, stödd standardväg för team att leverera lösningar. |
| Moderniseringsförmåga | moderniseringsförmåga | Organisationens förmåga att förändra arkitektur och plattform kontrollerat över tid. |


| Begrepp | Rekommenderad svensk användning | Kort definition |
|---|---|---|
| Cloud native-modernisering | cloud native-modernisering | Förändring av arkitektur, plattform och arbetssätt för att öka automation, driftbarhet, skalbarhet och förändringsförmåga. |
| Förändringsförmåga | förändringsförmåga | Organisationens förmåga att leverera ändringar snabbt, säkert och kontrollerat. |
| Lift and shift | lift and shift | Flytt av system till ny plattform utan väsentlig arkitekturell förändring. |
| Plattform som produkt | plattform som produkt | Intern plattform som har målgrupp, onboarding, support, golden paths och tydligt värdeerbjudande. |
| Moderniseringsobjekt | moderniseringsobjekt | Befintlig teknik eller arkitekturdel som analyseras för inkapsling, komplettering, ersättning eller avveckling. |

| OCI | OCI | Open Container Initiative, standarder för container image-format och runtime-beteende. |
| Container image | container image / containeravbild | Paketerad applikation med runtime, beroenden och metadata. |
| Container runtime | container runtime | Komponent som kör containers. |
| Registry | registry | Plats där container images lagras, versioneras och hämtas. |
| Containerfile | Containerfile | Fil som beskriver hur en container image byggs. |
| Bas-image | bas-image | Grundimage som applikationens image byggs ovanpå. |
| Rootless containers | rootless containers | Containers som körs utan root-behörighet på värdsystemet. |

| Project | project | OpenShift-begrepp för användarvänlig representation av namespace. |
| Namespace | namespace | Logisk avgränsning för resurser, åtkomst, policyer och ansvar. |
| Deployment | deployment | Resurs som beskriver önskat körningstillstånd för en applikation. |
| Service | service | Stabil intern åtkomstpunkt till en eller flera poddar. |
| Route | route | OpenShift-resurs för att exponera HTTP-baserade tjänster. |
| ConfigMap | ConfigMap | Resurs för icke-hemlig konfiguration. |
| Secret | Secret | Resurs för känslig konfiguration som lösenord, tokens och nycklar. |
| Operator | operator | Mönster för att automatisera livscykelhantering av komplexa komponenter. |
| Guardrails | guardrails | Styrande ramar som möjliggör självservice utan att tappa kontroll. |
| Golden path | golden path | Rekommenderad och stödd standardväg för vanligt leveransscenario. |

| Least privilege | least privilege / minsta privilegium | Principen att bara ge nödvändig åtkomst för uppgiften. |
| RBAC | RBAC / rollbaserad åtkomstkontroll | Modell för att styra behörigheter utifrån roller. |
| Servicekonto | service account / servicekonto | Plattformidentitet som används av workload eller automation. |
| Image-policy | image-policy | Regler för vilka container images som får byggas, lagras och köras. |
| Nätverkspolicy | network policy / nätverkspolicy | Regler som styr tillåten nätverkstrafik mellan workloads och namespaces. |
| Policy-as-code | policy-as-code | Maskinläsbara policyer som kan köras automatiskt i pipeline eller plattform. |
| Audit | audit / revisionsspår | Spårbar dokumentation av åtkomst, ändringar eller känsliga operationer. |
| Secret rotation | secret rotation | Kontrollerat byte av credentials, certifikat, nycklar eller tokens. |

| Dataroll | dataroll | Den funktion data eller datalager har, exempelvis primär transaktionskälla, sökindex eller cache. |
| Primär sanning | primär sanning | Datakälla som äger korrekt affärstillstånd. |
| Delad databas | delad databas | Databas eller schema som används direkt av flera applikationer. |
| Direkt schemaåtkomst | direkt schemaåtkomst | När applikationer läser eller skriver direkt i ett annat systems databasschema. |
| Dataägarskap | dataägarskap | Tydligt ansvar för datamodell, schema, kvalitet, förändringar och livscykel. |
| Datagovernance | datagovernance | Styrning av dataägarskap, klassificering, åtkomst, retention och kvalitet. |
| Schemahantering | schemahantering | Versionsstyrd hantering av databasstruktur och migrationer. |
| Restore-test | restore-test | Verifiering av att data kan återställas från backup. |

| Message queue | meddelandekö / message queue | Kö där meddelanden lagras tills en konsument behandlar dem. |
| Command message | kommandomeddelande / command message | Meddelande som ber mottagaren utföra en operation. |
| Domain event | domänhändelse / domain event | Händelse som beskriver något som redan har hänt i en affärsdomän. |
| Publish/subscribe | publish/subscribe | Mönster där flera prenumeranter kan ta emot publicerade meddelanden. |
| Event streaming | event streaming | Händelselogg där konsumenter kan läsa och återspela händelser. |
| Integrationsmonolit | integrationsmonolit | Integrationslandskap där system är tekniskt distribuerade men semantiskt hårt kopplade. |
| Idempotens | idempotens | Förmåga att hantera samma meddelande flera gånger utan felaktigt affärsresultat. |
| Dead-letter queue | dead-letter queue | Kö eller plats för meddelanden som inte kunde behandlas korrekt. |
| Consumer lag | consumer lag | Skillnad mellan producerade händelser och vad en konsument hunnit läsa. |

| Sekundärt index | sekundärt index | Härledd sök- eller läsmodell som inte äger primär affärssanning. |
| Reindexering | reindexering | Process för att återskapa ett index från primär källa eller händelseflöde. |
| Direkt indexering | direkt indexering | Mönster där applikationen skriver direkt till både primär källa och index. |
| Eventdriven indexering | eventdriven indexering | Mönster där events används för att uppdatera ett index asynkront. |
| CDC | Change Data Capture / CDC | Mönster för att fånga förändringar från en databas och föra dem vidare. |
| Batchindexering | batchindexering | Periodisk uppbyggnad eller uppdatering av index. |
| Indexeringsfördröjning | indexeringsfördröjning | Tid mellan förändring i primär källa och synlighet i index. |
| Observability-data | observability-data | Tekniska signaler som loggar, metrics och traces. |

| Cloud native-driftsmodell | cloud native-driftsmodell | Ansvars- och arbetssättsmodell för drift av dynamiska, plattformsbaserade applikationer. |
| Produktionsklarhet | produktionsklarhet | Definierad nivå av krav som en tjänst ska uppfylla innan produktion. |
| Runbook | runbook | Praktisk instruktion för felsökning och operativ hantering. |
| SRE | Site Reliability Engineering / SRE | Ingenjörsdrivet arbetssätt för tillförlitlighet och driftbarhet. |
| Toil | toil | Repetitivt manuellt arbete som inte skapar långsiktigt värde. |
| GitOps | GitOps | Modell där Git används som källa för önskat plattformstillstånd. |
| Deklarativ drift | deklarativ drift | Driftmodell där önskat tillstånd beskrivs och automation realiserar det. |
| Supportmodell | supportmodell | Dokumenterad modell för vem som hanterar olika typer av drift- och supportfrågor. |

| Disaster recovery | disaster recovery / DR | Förmåga att återställa tjänster efter större incident eller haveri. |
| Backup | backup | Kopia av data, konfiguration eller systemtillstånd. |
| Restore | restore / återställning | Praktisk återläsning eller återställning från backup eller annan källa. |
| Kontinuitetsplanering | kontinuitetsplanering | Planering för att verksamhetskritiska processer ska kunna fortsätta eller återupptas. |
| Återställningsordning | återställningsordning | Dokumenterad ordning för att återställa beroende komponenter. |
| DR-test | DR-test | Test eller övning som verifierar återställningsförmåga. |
| Rebuild | rebuild / återskapande | Att bygga upp en komponent på nytt från källor som Git, pipeline eller primärdata. |
| Degraderat läge | degraderat läge | Medvetet begränsat verksamhets- eller systemläge under incident. |

| Cloud native Java | cloud native Java | Java-applikation designad för plattformsdrift, externaliserad konfiguration och observerbarhet. |
| Serverbunden konfiguration | serverbunden konfiguration | Konfiguration som ligger i applikationsservern snarare än i explicit deploymentmodell. |
| Externaliserad konfiguration | externaliserad konfiguration | Konfiguration som tillförs vid deployment och inte byggs in i image. |
| Health check | health check | Kontroll som visar om applikationen lever eller är redo för trafik. |
| Graceful shutdown | graceful shutdown | Kontrollerad nedstängning där pågående arbete avslutas säkert. |
| Runtime footprint | runtime footprint | Applikationens resursprofil: minne, CPU, starttid, image-storlek och beroenden. |
| Sticky sessions | sticky sessions | Mönster där användare binds till en viss server- eller poddinstans. |

| Lokal transaktion | lokal transaktion | Transaktion som hanteras inom en tjänsts eget datalager. |
| Distribuerad transaktion | distributed transaction / distribuerad transaktion | Transaktion som koordinerar flera resurser eller system. |
| Eventual consistency | eventual consistency | Konsistensmodell där system blir konsistenta över tid. |
| Saga | saga | Affärsprocess uppdelad i flera lokala transaktioner med kompensation vid behov. |
| Kompenserande transaktion | kompenserande transaktion | Affärsmässig korrigering eller upphävning av tidigare steg. |
| Orkestrering | orchestration / orkestrering | Central styrning av ett processflöde. |
| Koreografi | choreography / koreografi | Eventdrivet samspel där tjänster reagerar på händelser. |
| Backpressure | backpressure | Förmåga att bromsa eller signalera överbelastning. |
| Distribuerad monolit | distributed monolith / distribuerad monolit | System med separata tjänster som fortfarande är hårt kopplade. |

| ADR | Architecture Decision Record / ADR | Kort dokument som beskriver ett arkitekturbeslut och dess konsekvenser. |
| Teknikradar | teknikradar | Visualisering eller lista över rekommenderade, prövade, utvärderade och avrådda tekniker. |
| Adopt | adopt | Teknik eller mönster som är rekommenderad standard för lämpliga fall. |
| Trial | trial | Teknik eller mönster som får användas i kontrollerade pilotfall. |
| Assess | assess | Teknik eller mönster som utvärderas men inte är normalval. |
| Hold | hold | Teknik eller mönster som ska undvikas eller kräver särskilt beslut. |
| Fitness function | fitness function | Testbart kriterium som visar om arkitekturen följer en princip. |
| Undantagsprocess | undantagsprocess | Kontrollerad process för avvikelse från standard. |
