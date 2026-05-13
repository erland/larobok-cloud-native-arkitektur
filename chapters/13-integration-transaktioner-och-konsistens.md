# Kapitel 13: Integration, transaktioner och konsistens

## Varför detta kapitel finns

När en traditionell Java EE-plattform moderniseras förändras transaktionsmodellen. I den gamla miljön kunde mycket hanteras med en gemensam applikationsserver, en central Oracle-databas, serverhanterade transaktioner och IBM MQ-flöden som låg nära samma tekniska kontrollmodell.

När funktionalitet flyttas till OpenShift och delas upp i nya tjänster blir gränserna tydligare men också mer distribuerade. En operation som tidigare var en lokal transaktion kan bli ett flöde över flera tjänster, databaser, köer och event. Då räcker det inte att fråga om tekniken fungerar. Arkitekten måste förstå konsistens, felhantering, idempotens, retry, kompensation och användarupplevelse.

Det här kapitlet ger en praktisk beslutsmodell för integration, transaktioner och konsistens i en stegvis moderniserad enterprise-arkitektur.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- skilja mellan lokal transaktion, distribuerad transaktion och eventual consistency
- förstå varför gamla transaktionsgränser ofta bryts vid modernisering
- avgöra när saga-mönster är lämpligt
- förklara varför idempotens är grundläggande i distribuerade flöden
- resonera om retry, dead-letter, ordering och kompensation
- formulera en konsistensmodell för moderniserade Java- och OpenShift-tjänster

## Från lokal transaktion till distribuerat flöde

I en traditionell Java EE-applikation kan flera steg ofta ske inom samma transaktion. Applikationen uppdaterar Oracle, gör affärslogik och skickar kanske meddelanden via en serverhanterad modell. Även om lösningen är komplex upplevs den ofta som en sammanhållen runtime.

När samma affärsprocess delas upp i flera tjänster förändras förutsättningarna. Varje tjänst kan ha egen databas, egen deployment, egen releasecykel och egen felmodell.

Arkitekten behöver då svara på:

- var går transaktionsgränsen?
- vem äger data?
- vad händer om steg två lyckas men steg tre misslyckas?
- hur görs retry?
- kan samma meddelande behandlas flera gånger?
- hur vet användaren om processen är klar?
- hur kompenseras ett tidigare steg?
- hur felsöks flödet end-to-end?

Detta är en arkitekturfråga, inte bara en kodfråga.

## Konsistensmodeller

| Modell | Beskrivning | Passar när |
|---|---|---|
| Lokal transaktion | En tjänst uppdaterar sitt eget datalager atomärt | Tydligt dataägarskap och avgränsat ansvar |
| Distribuerad transaktion | Flera resurser koordineras i samma transaktion | Få kontrollerade resurser med starka krav |
| Eventual consistency | System blir konsistenta över tid | Asynkrona flöden, läsmodeller och events |
| Kompenserande transaktion | Ett senare steg korrigerar tidigare steg | Affärsprocesser med flera lokala steg |
| Manuell avvikelsehantering | Undantag hanteras operativt | Sällsynta men komplexa fel |

Stark konsistens är enklare att förstå men kan skapa hård koppling. Eventual consistency kan ge bättre autonomi och robusthet men kräver bättre design kring status, fel och förväntningar.

## Distribuerade transaktioner

Distribuerade transaktioner försöker koordinera flera resurser så att de lyckas eller misslyckas tillsammans. I vissa traditionella enterprise-miljöer har detta varit en viktig egenskap.

I cloud native-arkitektur bör distribuerade transaktioner användas försiktigt.

Risker:

- stark koppling mellan system
- känslighet för nätverksfel
- svårare skalning
- svår felsökning
- låsning till specifika tekniker
- komplex rollback
- sämre autonomi mellan team

Det betyder inte att distribuerade transaktioner alltid är fel. Men de bör vara medvetna undantag, inte standardmönster för nya tjänster.

## Lokal transaktion per tjänst

Ett vanligt cloud native-mönster är att varje tjänst äger sin egen data och använder lokala transaktioner. Tjänsten ansvarar för att dess eget tillstånd är korrekt. Kommunikation med andra tjänster sker via API, kommando, event eller processflöde.

Fördelar:

- tydligt dataägarskap
- enklare deployment
- mindre teknisk koppling
- lättare att skala enskilda tjänster
- tydligare ansvar

Utmaningen är att affärsprocessen ofta sträcker sig över flera tjänster. Då behövs en processmodell, inte en stor teknisk transaktion.

## Saga-mönster

En saga är ett mönster för att hantera en affärsprocess som består av flera lokala transaktioner. Varje steg gör sin del och processen går vidare. Om ett senare steg misslyckas kan ett tidigare steg behöva kompenseras.

Exempel på orderflöde:

1. Order skapas.
2. Betalning reserveras.
3. Lager reserveras.
4. Leverans bokas.
5. Order bekräftas.

Om lagerreservation misslyckas kan betalningsreservationen behöva släppas. Detta är inte en teknisk rollback. Det är en affärsmässig kompensation.

Saga passar när processen består av flera tydliga affärssteg, varje steg har egen dataägare, full atomär transaktion inte är realistisk, kompensation kan definieras, processstatus kan följas och observability finns.

## Orkestrering och koreografi

| Mönster | Beskrivning | Fördel | Risk |
|---|---|---|---|
| Orkestrering | En central process styr stegen | Tydlig status och kontroll | Risk för central processmonolit |
| Koreografi | Tjänster reagerar på events | Lösare koppling | Svårare helhetsförståelse |
| Hybrid | Kärnflöde styrs, sekundära reaktioner är eventdrivna | Praktisk balans | Kräver principer |

För enterprise-modernisering är hybrid ofta realistiskt. Ett kritiskt orderflöde kan behöva orkestrering, medan notifieringar, sökindexering och analys kan ske eventdrivet.

## Idempotens

Idempotens betyder att samma operation kan utföras flera gånger utan att skapa felaktigt resultat. Det är centralt i distribuerade system eftersom meddelanden kan levereras mer än en gång och klienter kan göra retry.

Exempel:

- Att skapa samma order två gånger är normalt inte idempotent.
- Att sätta orderstatus till “bekräftad” två gånger kan vara idempotent.
- Att behandla ett meddelande med unikt message-id kan göras idempotent genom att spara behandlade id:n.
- Att reservera lager behöver ofta idempotensnyckel kopplad till orderrad.

Idempotens ska designas från början. Den är svår att lägga till efter att fel redan uppstått.

## Retry, timeout och backpressure

Retry behövs för tillfälliga fel, men retry utan styrning kan förvärra incidenter. En bra retry-strategi definierar vilka fel som ska retried, hur många försök som görs, väntetid mellan försök, maximal total tid och när flödet ska gå till dead-letter eller avvikelsehantering.

Timeouts är lika viktiga. För långa timeouts binder resurser. För korta timeouts skapar falska fel. Timeout ska sättas utifrån flödets krav och beroendets beteende.

Backpressure innebär att systemet kan bromsa eller signalera överbelastning i stället för att okontrollerat fortsätta ta emot arbete. Detta är viktigt i kö- och eventflöden där meddelanden annars kan byggas upp snabbare än konsumenter hinner bearbeta dem.

## Dead-letter och avvikelsehantering

Dead-letter queue eller motsvarande avvikelsemekanism används när ett meddelande inte kan behandlas korrekt. Det ska inte vara en soptunna.

En dead-letter-process behöver ägare, larm, klassificering av fel, möjlighet att inspektera meddelanden, möjlighet att återköra säkert, dokumenterad hantering, skydd mot känslig information och spårbarhet.

Vissa fel kan lösas tekniskt. Andra kräver affärsbeslut. Därför behöver dead-letter och avvikelsehantering kopplas till både drift och verksamhet.

## Ordering

Vissa flöden kräver att meddelanden hanteras i ordning. Andra gör inte det. Att kräva global ordering är ofta dyrt och begränsar skalbarhet.

Frågor:

- behövs ordering globalt eller per affärsobjekt?
- räcker ordering per order, kund eller konto?
- vad händer om en senare händelse kommer först?
- finns versionsnummer eller sekvensnummer?
- kan konsumenten ignorera äldre händelser?
- hur påverkas parallell bearbetning?

Ofta räcker ordering per affärsobjekt. Global ordering bör vara ett undantag.

## Eventual consistency och användarupplevelse

Eventual consistency är tekniskt accepterbart i många flöden, men bara om användarupplevelsen och verksamhetsprocessen stödjer det.

Exempel:

- Ett sökindex får vara några sekunder efter primär källa.
- En orderstatus kan visas som “behandlas” innan alla steg är klara.
- En notifiering kan skickas senare.
- En rapport kan vara fördröjd.

Det viktiga är att inte visa en falsk bild av omedelbar konsistens. Om användaren behöver veta att processen pågår ska systemet visa det.

## Transaktioner och Oracle

När Oracle är primär källa kan vissa transaktionsgränser behöva behållas initialt. Det kan vara rätt val för kärntransaktioner med hög risk. Men nya tjänster bör inte reflexmässigt försöka sträcka Oracle-transaktioner över flera distribuerade komponenter.

Strategier:

- behåll lokal Oracle-transaktion i legacy-kärna
- kapsla in Oracle via API
- publicera events efter commit
- bygg sekundära läsmodeller
- använd saga för nya flöden
- flytta dataägarskap stegvis

Målet är inte att ta bort stark konsistens överallt. Målet är att använda den där den behövs och undvika att den blockerar all förändring.

## Beslutsguide: välj konsistensmodell

| Scenario | Rekommenderad modell |
|---|---|
| En tjänst uppdaterar egen data | Lokal transaktion |
| Flera autonoma tjänster deltar i process | Saga |
| Sökindex uppdateras efter orderändring | Eventual consistency |
| Betalning reserveras och kan släppas | Saga med kompensation |
| Finansiell bokföring kräver strikt korrekthet | Stark konsistens eller särskild kontrollmodell |
| Legacy kräver MQ och databas i samma flöde | Behåll initialt och kapsla in |
| Flera system behöver informeras | Domain events |
| Användaren behöver omedelbart svar | Synkront API eller tydlig async-status |

## Beslutsguide: orkestrering eller koreografi

| Fråga | Orkestrering passar bättre | Koreografi passar bättre |
|---|---|---|
| Behövs tydlig processstatus? | Ja | Inte alltid |
| Är flödet affärskritiskt? | Ofta | Ibland |
| Finns många oberoende reaktioner? | Mindre lämpligt | Ja |
| Behövs central kompensation? | Ja | Svårare |
| Ska nya konsumenter kunna tillkomma enkelt? | Mindre flexibelt | Ja |
| Är organisationen ovan vid events? | Ja | Kräver mognad |

## Tradeoffs

| Val | Fördel | Risk |
|---|---|---|
| Lokal transaktion per tjänst | Tydligt dataägarskap | Kräver processhantering över tjänster |
| Distribuerad transaktion | Stark konsistens | Hård koppling och operativ komplexitet |
| Eventual consistency | Skalbarhet och lösare koppling | Fördröjning och mer felhantering |
| Saga med orkestrering | Tydlig processkontroll | Risk för processmonolit |
| Saga med koreografi | Autonomi och flexibilitet | Svårare helhetsförståelse |
| Aggressiv retry | Klarar tillfälliga fel | Kan förstärka incidenter |
| Manuell avvikelsehantering | Praktiskt för komplexa undantag | Kräver operativ disciplin |

## Anti-patterns

### Distribuerad monolit

Tjänster är separata i deployment men så hårt beroende av varandra att de måste ändras, testas och deployas tillsammans.

### Eventual consistency utan användarförväntan

Systemet är asynkront men användaren tror att allt är omedelbart uppdaterat.

### Retry utan gräns

Oändliga retries kan överbelasta beroenden och förvärra incidenter.

### Saga utan kompensation

En saga utan definierade kompensationssteg är bara en kedja av osäkra operationer.

### Events utan ägarskap

Om ingen äger eventets semantik, versionering och livscykel blir flödet svårt att ändra.

## Vanliga fallgropar

- Att försöka bevara gamla transaktionsgränser över nya tjänster.
- Att inte definiera dataägarskap.
- Att sakna idempotens.
- Att underskatta dubbletter.
- Att inte mäta consumer lag och ködjup.
- Att sakna statusmodell för långa processer.
- Att blanda kommando och event.
- Att göra kompensation teknisk när den egentligen är affärsmässig.
- Att glömma användarupplevelsen vid asynkrona flöden.
- Att sakna operativa rutiner för fastnade processer.

## Arkitektens checklista

Innan ett distribuerat flöde införs bör arkitekten kunna svara på:

- Var går transaktionsgränsen?
- Vem äger varje dataobjekt?
- Vilka steg ingår i processen?
- Vilka steg kan misslyckas?
- Finns kompensation?
- Är kompensation teknisk eller affärsmässig?
- Vilka meddelanden är kommandon?
- Vilka meddelanden är events?
- Är konsumenter idempotenta?
- Behövs ordering?
- Hur hanteras retry?
- Hur hanteras dead letters?
- Hur ser användaren processstatus?
- Vilka larm behövs?
- Hur felsöks ett fastnat flöde?
- Finns ADR?

## Koppling till moderniseringscaset

Nordisk Handel AB analyserar sitt orderflöde. I nuläget sker mycket inom Java EE, Oracle och IBM MQ. Vid modernisering delas flödet inte upp enbart efter tekniska komponenter, utan efter dataägarskap, affärssteg och kompensationsmöjlighet.

Organisationen väljer orkestrering för kärnflödet eftersom orderstatus är affärskritisk. Domänhändelser används för sekundära reaktioner som sökindexering, notifieringar och analys. IBM MQ behålls initialt för vissa legacy-flöden, men nya event får tydliga kontrakt.

Det viktiga beslutet är att inte försöka sprida den gamla transaktionsmodellen över många nya tjänster. I stället designas varje flöde med tydlig konsistensmodell.

## Snabb sammanfattning

- Modernisering bryter ofta gamla transaktionsgränser.
- Distribuerade transaktioner bör vara medvetna undantag.
- Lokal transaktion per tjänst kräver processmodell över tjänster.
- Saga-mönster kan hantera affärsprocesser med flera steg.
- Idempotens, retry, dead-letter och ordering är grundläggande designfrågor.
- Eventual consistency kräver tydlig användarförväntan.
- Dataägarskap och meddelandesemantik måste definieras före uppdelning.

## Nästa steg

Nästa kapitel behandlar observability och operativ mognad. Där går vi igenom hur loggar, metrics, tracing, SLO:er och runbooks gör distribuerade flöden möjliga att förstå och drifta.
