# Kapitel 2: Containerteknik med Podman och OCI

## Varför detta kapitel finns

Förra kapitlet etablerade att cloud native-modernisering inte är samma sak som att bara flytta en applikation till en container. Det här kapitlet gör containertekniken konkret. Innan OpenShift, operators, nätverkspolicyer och plattformsstyrning diskuteras behöver arkitekten förstå vad en container faktiskt är, vad en container image innehåller och varför OCI-standarder spelar roll.

Podman används i den här boken som ett praktiskt och arkitekturellt verktyg. Det är inte målplattformen för produktion, men det är ett bra sätt att synliggöra applikationens antaganden: hur den startar, vilka beroenden den har, vilken användare processen kör som, hur konfiguration matas in och vad som händer när processen stoppas.

För lösningsarkitekter och erfarna Java EE-utvecklare är syftet inte att lära sig alla kommandon. Syftet är att förstå containerpaketering som en del av arkitekturen.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- förklara skillnaden mellan container image, container, registry och runtime
- beskriva varför OCI-standarder är viktiga för portabilitet och styrning
- förstå Podmans roll i en Red Hat-orienterad moderniseringsresa
- identifiera vanliga problem när Java EE-applikationer paketeras som containers
- bedöma om en container image är rimlig att gå vidare med mot OpenShift
- formulera grundläggande krav för lokal containerverifiering

## Container image, container och runtime

En container image är ett paketerat och versionerat underlag för att skapa containers. Den innehåller applikationen, runtime-beroenden och metadata som behövs för att starta processen.

En container är en körande instans av en image. Det är viktigt att skilja på dessa två. Imagen är paketet. Containern är processen som körs från paketet.

En container runtime är komponenten som kör containern. Podman kan användas lokalt för att bygga och köra containers. I OpenShift hanteras körningen av plattformens runtime och orkestrering.

| Begrepp | Kort förklaring | Arkitektonisk betydelse |
|---|---|---|
| Container image | Paketerad applikation och runtime | Påverkar säkerhet, patchning och reproducerbarhet |
| Container | Körande instans av en image | Ska normalt vara ersättningsbar |
| Runtime | Komponent som kör containern | Ska inte bli applikationens dolda beroende |
| Registry | Plats där images lagras och hämtas | Central för spårbarhet och policy |
| Tag | Namn eller etikett för image-version | Måste hanteras kontrollerat |
| Digest | Innehållsbaserad identifierare | Ger starkare spårbarhet än taggar |

För en arkitekt är en image inte bara en teknisk artefakt. Den är en del av leveranskedjan och därmed också en del av säkerhets-, drift- och livscykelmodellen.

## OCI-standarder

OCI står för Open Container Initiative. OCI-standarder definierar bland annat format för container images och runtime-beteende. Poängen är att en image inte ska vara bunden till ett enskilt verktyg eller en enskild leverantör.

För en enterprise-organisation är detta viktigt av flera skäl:

- images kan byggas och köras med flera kompatibla verktyg
- plattformen kan införa policyer runt image-format och metadata
- säkerhetsskanning och signering kan standardiseras
- registry-flöden kan göras mer förutsägbara
- team kan undvika onödig inlåsning i lokala verktygsval

OCI gör inte applikationen automatiskt portabel i praktiken. En image kan följa standarden men ändå innehålla antaganden som gör den svår att köra i OpenShift. Men standarden ger en gemensam teknisk grund.

## Podmans roll

Podman är särskilt relevant i Red Hat-orienterade miljöer. Det kan användas för att bygga, köra och inspektera containers utan att kräva samma daemon-modell som vissa andra containerverktyg. Podman stödjer också rootless containers, vilket gör det möjligt att köra containers utan root-behörighet på värdsystemet.

I den här boken används Podman för tre syften:

1. **Förståelse.** Teamen ser vad en image faktiskt innehåller och hur applikationen startar.
2. **Verifiering.** Grundläggande containerantaganden kan testas innan OpenShift blir inblandat.
3. **Standardisering.** Organisationen kan skapa ett gemensamt språk kring Containerfile, image, registry och runtime.

Podman är däremot inte ett bevis på att applikationen är redo för produktion. En container som fungerar lokalt kan fortfarande misslyckas i OpenShift på grund av security context, nätverk, secrets, persistent storage, resursgränser eller health checks.

## Containerfile som arkitekturdokument

En Containerfile beskriver hur en image byggs. Den kan verka som en teknisk detalj, men den uttrycker flera viktiga arkitekturbeslut:

- vilken bas-image används?
- vilken runtime följer med?
- vilken applikationsartefakt kopieras in?
- vilka paket installeras?
- vilken användare kör processen som?
- vilka portar exponeras?
- hur startas applikationen?
- finns miljöspecifik konfiguration inbyggd?
- finns onödiga verktyg i runtime-imagen?

För Java EE-applikationer är detta särskilt viktigt eftersom mycket historiskt har legat i applikationsservern. När runtime paketeras i en image behöver teamet avgöra vad som ska finnas i imagen och vad som ska tillföras av plattformen.

## Bas-images och livscykel

En bas-image är grunden som applikationens image byggs ovanpå. Den kan innehålla operativsystemslager, Java-runtime, certifikat, bibliotek och andra komponenter.

Val av bas-image påverkar:

- säkerhet
- patchning
- support
- image-storlek
- kompatibilitet
- starttid
- driftmodell
- licens- och supportfrågor

I en enterprise-miljö bör team inte välja bas-images helt fritt. Plattformsteamet bör erbjuda godkända bas-images och en modell för hur de uppdateras. Annars riskerar organisationen att få många images med olika sårbarheter, olika runtime-versioner och otydligt ägarskap.

## Rootless containers

Rootless containers innebär att containers kan köras utan root-behörighet på värdsystemet. Ur ett arkitekturperspektiv är detta viktigt eftersom det tränar teamen i att inte bygga applikationer som kräver onödigt höga privilegier.

Vanliga problem som rootless-körning kan avslöja:

- applikationen försöker skriva till skyddade kataloger
- startscript förutsätter root
- filer i imagen har fel ägare eller rättigheter
- applikationen försöker binda till privilegierade portar
- runtime kräver operationer som inte passar i låst plattformsmiljö
- logg- eller tempkataloger är felplacerade

Om en Java EE-applikation bara fungerar som root bör arkitekten se det som en varningssignal.

## Registry och image-flöde

Ett registry är platsen där images lagras, versioneras och hämtas. I en moderniserad leveranskedja blir registryt en central kontrollpunkt.

Ett moget image-flöde bör hantera:

- byggbarhet från källkod
- versionsmärkning
- immutable tags eller digest-baserad spårbarhet
- sårbarhetsskanning
- signering
- promotion mellan miljöer
- policy för vilka images som får deployas
- koppling mellan källkod, build och image
- rensning av gamla images

Ett vanligt misstag är att bygga olika images för olika miljöer. Ett bättre mål är att bygga en image och sedan tillföra miljöspecifik konfiguration vid deployment.

## Java EE i container

En Java EE-applikation som tidigare deployades till en central applikationsserver behöver analyseras innan den paketeras i en image.

Frågor att ställa:

- Vilken applikationsserver eller runtime krävs?
- Finns datasources definierade i servern?
- Finns JMS- eller MQ-resurser i serverkonfigurationen?
- Finns gemensamma bibliotek på servern?
- Finns certifikat eller truststores i servermiljön?
- Finns miljövariabler eller system properties som inte dokumenterats?
- Skriver applikationen till lokalt filsystem?
- Hur hanteras sessionsdata?
- Hur startar och stoppar applikationen?
- Hur loggar applikationen?

Om dessa frågor inte kan besvaras är det för tidigt att säga att applikationen är containerklar.

## Lokal verifiering med Podman

Podman kan användas för en första lokal verifiering. Syftet är inte att återskapa OpenShift på utvecklarens dator, utan att hitta grundläggande problem tidigt.

En enkel verifieringsmodell:

1. Bygg imagen från ren källkod och godkänd bas-image.
2. Starta containern utan specialflaggor.
3. Tillför konfiguration externt.
4. Kontrollera att processen inte kräver root.
5. Kontrollera att loggar skrivs till standard output.
6. Kontrollera att applikationen kan stoppas kontrollerat.
7. Kontrollera att inga secrets finns i imagen.
8. Dokumentera beroenden till databas, MQ och externa system.

Om en applikation misslyckas redan här är det bättre att upptäcka det lokalt än först i OpenShift.

## Beslutsguide: när är Podman tillräckligt?

| Fråga | Podman kan hjälpa | OpenShift måste verifiera |
|---|---|---|
| Kan imagen byggas? | Ja | Pipeline och policy |
| Startar processen? | Ja | Ja, med plattformens regler |
| Fungerar rootless-körning? | Ja | Ja, med security context |
| Är konfiguration externaliserad? | Ja | Ja, med ConfigMaps och Secrets |
| Skrivs loggar till stdout? | Ja | Ja, med central logginsamling |
| Fungerar health endpoint? | Delvis | Ja, med readiness och liveness |
| Fungerar service discovery? | Nej | Ja |
| Fungerar horisontell skalning? | Nej | Ja |
| Fungerar persistent storage? | Begränsat | Ja |
| Fungerar nätverkspolicyer? | Nej | Ja |

Podman svarar på frågan: “är detta en rimlig container?”. OpenShift svarar på frågan: “fungerar detta som workload i vår plattform?”.

## Beslutsguide: image-strategi

| Situation | Rekommenderat val |
|---|---|
| Ny Java-tjänst | Använd godkänd lättviktsbas och bygg image reproducerbart |
| Befintlig Java EE-app med komplex runtime | Paketera befintlig runtime initialt, men dokumentera övergångsplan |
| App kräver root | Anpassa image och filrättigheter innan OpenShift |
| App kräver miljöspecifik image | Flytta konfiguration ut ur imagen |
| App kräver lokala filer | Avgör om filerna är temporära, objekt, state eller legacy-integration |
| App innehåller secrets i build | Stoppa och korrigera buildflödet |
| App har stor image | Analysera runtime, beroenden och patchmodell |

## Vanliga misstag

### Secrets i imagen

Credentials, tokens, certifikat och lösenord får inte byggas in i en image. Det skapar stora säkerhetsrisker och gör rotation svår.

### Senaste taggen som produktionsreferens

Att deploya `latest` eller motsvarande otydliga taggar gör spårbarhet och rollback svårare. Produktion bör använda tydligt versionerade images eller digest.

### Root som standard

Att köra processen som root för att slippa rättighetsproblem döljer designproblem och försämrar säkerhetsställningen.

### Miljöspecifika images

Om test, staging och produktion kräver olika images blir leveranskedjan svårare att säkra och verifiera.

### Container som liten VM

Om en container innehåller flera processer, manuell konfiguration och serverliknande beteende har man ofta bara paketerat den gamla servern i ett nytt format.

## Arkitektens checklista

Innan en image går vidare till OpenShift bör arkitekten kunna svara på:

- Vilken bas-image används?
- Vem äger bas-imagens livscykel?
- Vilken Java-runtime används?
- Hur patchas runtime?
- Byggs imagen reproducerbart?
- Finns secrets i imagen?
- Kan samma image användas i flera miljöer?
- Kör processen utan root?
- Skriver applikationen loggar till standard output?
- Hur tillförs konfiguration?
- Hur stoppas processen?
- Finns health endpoints?
- Vilka externa beroenden finns?
- Finns lokalt state?
- Är image-versionen spårbar till källkod och build?

## Koppling till moderniseringscaset

Nordisk Handel AB väljer en stödapplikation som första containerkandidat. Applikationen är Java EE-baserad men har relativt få integrationer och begränsat state. Teamet bygger en image med Podman och upptäcker tre saker:

1. Applikationen förutsätter en datasource som tidigare fanns i applikationsservern.
2. Ett startscript skriver temporära filer i en katalog som inte fungerar rootless.
3. Loggar skrivs till en lokal fil i stället för standard output.

Detta visar värdet av lokal verifiering. Problemen är inte OpenShift-problem. De är gamla serverantaganden som blir synliga när applikationen paketeras som container.

Teamet åtgärdar problemen innan applikationen går vidare till OpenShift-test. Därmed används Podman som arkitekturell kontrollpunkt, inte som slutgiltig produktionsvalidering.

## Snabb sammanfattning

- En container image är en del av arkitekturens leveranskedja, inte bara ett build-resultat.
- OCI-standarder ger en gemensam grund för image- och runtime-kompatibilitet.
- Podman är användbart för lokal förståelse och verifiering.
- Rootless containers hjälper till att upptäcka säkerhets- och rättighetsproblem tidigt.
- Containerfile bör behandlas som ett arkitekturdokument.
- En image som fungerar lokalt är inte automatiskt redo för OpenShift.
- Java EE-applikationer behöver analyseras utifrån runtime, konfiguration, state och beroenden.

## Nästa steg

Nästa kapitel går vidare till OpenShift som plattform. Där flyttas fokus från lokal containerverifiering till hur applikationer körs, styrs, exponeras och förvaltas i en gemensam enterprise-plattform.
