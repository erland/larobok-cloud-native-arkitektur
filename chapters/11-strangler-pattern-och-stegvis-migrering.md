# Kapitel 11: Strangler pattern och stegvis migrering

## Varför detta kapitel finns

Nu har boken etablerat flera grundförmågor: containerpaketering, OpenShift som plattform, säkerhet, dataarkitektur, messaging, sökindex, stateful workloads, driftsmodell och kontinuitet. Nästa fråga är hur organisationen faktiskt tar sig från nuläge till målbild utan att skapa ett för stort transformationsprogram.

Många enterprise-moderniseringar misslyckas inte för att målarkitekturen är dålig, utan för att genomförandet blir för stort. Ett försök att ersätta en Java EE-plattform, Oracle-beroenden och IBM MQ-flöden i ett enda samlat steg leder ofta till lång ledtid, hög risk och svår rollback.

Strangler pattern är ett sätt att modernisera stegvis. Nya komponenter byggs runt det gamla systemet, trafik och funktionalitet flyttas gradvis, och legacy-delar avvecklas när de inte längre behövs. Det är särskilt relevant när systemet är affärskritiskt och inte kan stoppas under en lång ombyggnadsperiod.

## Lärandemål

Efter kapitlet ska läsaren kunna:

- förklara strangler pattern som moderniseringsmönster
- identifiera bra första kandidater för stegvis migrering
- skilja mellan inkapsling, komplettering, parallellkörning och avveckling
- resonera om data, trafikstyrning och integration under migrering
- förstå riskerna med big bang-modernisering
- formulera en praktisk roadmap för att bryta ut funktionalitet ur legacy

## Big bang som risk

Big bang-modernisering innebär att ett stort system eller en stor plattform ersätts i ett samlat steg. Det kan låta attraktivt eftersom slutmålet blir tydligt: gammalt bort, nytt in. Men i affärskritiska enterprise-miljöer är det ofta riskfyllt.

Vanliga problem:

- beroenden upptäcks sent
- datamigrering blir större än väntat
- testomfånget blir mycket stort
- verksamheten får vänta länge på värde
- rollback blir svår eller omöjlig
- legacy-systemet fortsätter förändras under projektet
- teamen bygger ny komplexitet innan något används
- kostnad och risk samlas i slutet

Big bang kan ibland vara rätt för små eller avgränsade system. Men för en Java EE-plattform med Oracle och IBM MQ som bär centrala affärsflöden är stegvis migrering oftast säkrare.

## Strangler pattern

Strangler pattern innebär att nya lösningar byggs runt ett befintligt system och gradvis tar över delar av funktionaliteten. Namnet kommer från idén att en ny struktur växer runt den gamla tills den gamla delen kan tas bort.

En typisk sekvens:

1. Identifiera en avgränsad funktion.
2. Skapa ett kontrollerat gränssnitt framför legacy.
3. Bygg ny funktionalitet bredvid den gamla.
4. Styr en liten del av trafiken eller användarna till den nya lösningen.
5. Jämför resultat, prestanda och driftbeteende.
6. Flytta mer trafik eller fler funktioner.
7. Avveckla motsvarande legacy-del när beroenden är borta.

Strangler pattern är inte bara ett tekniskt mönster. Det är ett riskhanteringsmönster.

## Inkapsling före ersättning

Många vill börja med att bygga nytt. Ofta är ett bättre första steg att kapsla in det gamla.

Inkapsling innebär att direkta beroenden till legacy minskas genom ett kontrollerat gränssnitt. Det kan vara ett API, en adapter, en fasad, en eventbrygga eller en kontrollerad läsmodell.

Exempel:

- nya tjänster får inte läsa direkt från Oracle-schema
- IBM MQ-flöden exponeras via adapter där det är rimligt
- filbaserade integrationer kapslas bakom tjänst
- gammal funktionalitet exponeras via API i stället för databasåtkomst
- autentisering, logging och audit standardiseras runt legacy-komponenten

Inkapsling löser inte alla problem. Men den skapar en tydligare gräns som gör nästa steg möjligt.

## Första migreringskandidaten

Valet av första kandidat är avgörande. Den ska ge lärande och synligt värde, men inte vara så kritisk att varje fel hotar hela programmet.

Bra kandidater har ofta:

- tydligt affärsvärde
- begränsad domän
- kända användare
- hanterbar datamängd
- få integrationer
- möjlighet till parallellkörning
- tydliga mätetal
- ägande team
- rimlig rollbackmöjlighet

Dåliga första kandidater är ofta:

- mest affärskritiska kärntransaktionen
- funktioner med många dolda databeroenden
- flöden med okända MQ-konsumenter
- delar som kräver total datamigrering direkt
- tekniskt intressanta men verksamhetsmässigt oviktiga delar
- komponenter där ingen äger processen

En pilot ska inte bara bevisa att teknik fungerar. Den ska bevisa att organisationen kan modernisera kontrollerat.

## Migreringsdimensioner

Migrering kan ske längs flera dimensioner. Det är viktigt att veta vilken dimension som förändras i varje steg.

| Dimension | Exempel | Risk |
|---|---|---|
| Funktion | En viss funktion byggs om | Otydlig funktionsgräns |
| Trafik | Vissa användare styrs till ny lösning | Felaktig routing |
| Data | Data replikeras eller flyttas | Konsistensproblem |
| Integration | MQ-flöde ersätts eller kapslas | Semantik ändras |
| UI | Nytt gränssnitt ovanpå gammal backend | Dolda beroenden består |
| Plattform | Komponent flyttas till OpenShift | Driftmodell ändras |
| Organisation | Team får nytt ägarskap | Ansvar kan bli oklart |

En bra roadmap anger vilken dimension som förändras först och vilka som hålls stabila.

## Trafikstyrning

Trafikstyrning gör det möjligt att styra användare, anrop eller funktioner mellan gammal och ny lösning.

Mönster:

- pilotgrupp
- procentuell trafikstyrning
- routing per kundsegment
- routing per funktion
- feature flags
- shadow traffic
- parallellkörning
- manuell växling vid låg risk

Shadow traffic innebär att den nya lösningen får samma input som den gamla men inte påverkar resultatet. Det är användbart för att jämföra beteende utan kundpåverkan.

Trafikstyrning kräver observability. Om organisationen inte kan se skillnad mellan gammalt och nytt beteende blir stegvis migrering svår att styra.

## Data under migrering

Data är ofta den svåraste delen av modernisering. Kod kan flyttas snabbare än dataägarskap.

Vanliga datamönster:

| Mönster | Beskrivning | Passar när |
|---|---|---|
| Legacy äger data | Ny tjänst läser via API eller adapter | Tidig fas |
| Sekundär läsmodell | Data replikeras till sök eller läsmodell | Sök, rapportering, pilot |
| Ny tjänst äger ny data | Ny funktion får egen databas | Avgränsad domän |
| Dubbel skrivning | Data skrivs till gammal och ny modell | Endast kortvarigt och kontrollerat |
| Eventdriven synkronisering | Events uppdaterar andra modeller | Eventual consistency accepteras |
| Full datamigrering | Ägarskap flyttas helt | Senare fas efter stabilisering |

Dubbel skrivning är särskilt riskfyllt. Om ena skrivningen lyckas och den andra misslyckas kan dubbla sanningar uppstå. Därför bör dubbel skrivning vara tidsbegränsad, observerad och dokumenterad.

## Integration under migrering

IBM MQ och andra integrationsflöden behöver hanteras varsamt. Ett gammalt MQ-meddelande kan ha fler konsumenter än man tror, eller bära semantik som inte är dokumenterad.

Frågor:

- är meddelandet ett kommando eller en händelse?
- vilka konsumenter finns?
- finns okända konsumenter?
- kan nytt event publiceras parallellt?
- behövs adapter från MQ till eventflöde?
- vem äger kontraktet?
- hur hanteras dubbletter?
- hur verifieras samma affärsresultat?
- när kan gammalt flöde stängas?

En bra strategi är ofta att först observera och klassificera integrationsflöden innan de ändras.

## Parallellkörning

Parallellkörning innebär att gammal och ny lösning körs samtidigt under en övergångsperiod. Det kan användas för jämförelse, validering eller gradvis flytt.

Fördelar:

- lägre risk
- möjlighet att jämföra resultat
- enklare rollback
- verksamheten kan vänja sig gradvis
- fel upptäcks tidigare

Risker:

- dubbel driftkostnad
- mer komplex felsökning
- data kan divergera
- team kan skjuta upp avveckling
- gamla och nya regler kan skilja sig

Parallellkörning ska ha syfte, mätetal och slutdatum. Annars blir den permanent.

## Avveckling som del av planen

Modernisering är inte klar när den nya lösningen fungerar. Den är klar när den gamla vägen är avvecklad eller tydligt motiverad att finnas kvar.

Avveckling kräver:

- identifierade beroenden
- kommunicerat stoppdatum
- datamigrering eller arkivering
- borttagning av routes, köer och schemaåtkomst
- städning av behörigheter
- uppdaterad dokumentation
- kostnadsuppföljning
- beslut om vad som behålls för historik eller audit

Utan avveckling skapas permanent dubbel komplexitet.

## Beslutsguide: välj moderniseringsstrategi

| Scenario | Rekommenderad strategi |
|---|---|
| Funktion är avgränsad och viktig | Strangler pattern med ny tjänst |
| Många system delar databas | Inkapsla dataåtkomst först |
| Legacy MQ-flöde är kritiskt | Behåll och kapsla in initialt |
| Ny sökfunktion behövs | Bygg sekundärt index |
| UI behöver moderniseras snabbt | Nytt gränssnitt mot fasad/API |
| Kärntransaktion är hårt kopplad | Kartlägg och dela upp först |
| Ny domänfunktion byggs | Cloud native från start |
| Dataägarskap är otydligt | Etablera datakontrakt före flytt |

## Beslutsguide: första kandidat

| Fråga | Bra signal | Varningssignal |
|---|---|---|
| Finns tydlig domängräns? | Ja | Funktionen skär genom allt |
| Finns ägande team? | Ja | Oklart ansvar |
| Är datamodellen avgränsad? | Ja | Delas av många system |
| Kan rollback göras? | Ja | Svårt eller omöjligt |
| Finns mätbar nytta? | Ja | Enbart tekniskt intresse |
| Kan gammalt och nytt samexistera? | Ja | Nej |
| Finns observability? | Ja | Nej |
| Är integrationsflöden kända? | Ja | Okända konsumenter |

## Tradeoffs

| Val | Fördel | Risk |
|---|---|---|
| Stegvis migrering | Lägre risk och tidigare lärande | Längre period av dubbel komplexitet |
| Big bang | Tydlig slutbild | Hög risk och svår rollback |
| Inkapsla legacy | Minskar direktkoppling | Kan skapa nytt lager att förvalta |
| Bygga ny tjänst parallellt | Gradvis övergång | Kräver synkronisering |
| Behålla data i legacy initialt | Lägre risk | Begränsar autonomi |
| Flytta data tidigt | Större frikoppling | Hög migreringsrisk |
| Shadow traffic | Säker validering | Kräver avancerad jämförelse |
| Feature flags | Flexibel styrning | Kan bli teknisk skuld om de inte städas |

## Anti-patterns

### Ny monolit bredvid gammal monolit

Om den nya lösningen byggs som en stor sammanhängande ersättare har organisationen inte minskat risk. Den har bara skapat en parallell monolit.

### Evig samexistens

Samexistens är ofta nödvändig, men måste ha avvecklingsplan. Annars blir den permanent.

### Fasad utan ägarskap

En fasad kan minska koppling, men om ingen äger kontrakt och livscykel blir den ett nytt legacy-lager.

### Data först utan domänförståelse

Att börja med att flytta tabeller utan att förstå affärsregler leder ofta till fel gränser.

### Pilot utan väg till produktion

En teknisk demo ger begränsat värde om den inte testar verklig drift, säkerhet, data och support.

## Vanliga fallgropar

- Att välja första kandidat enbart utifrån teknisk enkelhet.
- Att underskatta dolda databeroenden.
- Att sakna rollback-plan.
- Att inte mäta gammalt och nytt beteende.
- Att låta dubbel skrivning pågå för länge.
- Att migrera MQ-flöden utan att förstå semantik.
- Att glömma avvecklingsbudget.
- Att inte involvera verksamheten i trafikstyrning.
- Att sakna tydlig ägare för fasad eller adapter.
- Att inte dokumentera större beslut med ADR.

## Arkitektens checklista

Innan en strangler-migrering startar bör arkitekten kunna svara på:

- Vilken funktion eller domän ska migreras?
- Varför är den en bra första kandidat?
- Vilken legacy-del kapslas in?
- Vilka Oracle-beroenden finns?
- Vilka MQ- eller integrationsberoenden finns?
- Vem äger data?
- Hur styrs trafik?
- Hur mäts resultat?
- Hur görs rollback?
- Kan gammalt och nytt köras parallellt?
- Vilka delar är temporära?
- När avvecklas gammal funktion?
- Vilka risker accepteras?
- Finns observability?
- Finns ADR?

## Koppling till moderniseringscaset

Nordisk Handel AB väljer orderhistoriksökning som första tydliga strangler-kandidat. Funktionen är viktig för kundtjänst, men den är inte samma sak som orderkärnans primära transaktioner. Oracle fortsätter äga orderdata, medan Elasticsearch används som sekundärt index. En ny söktjänst byggs på OpenShift.

Mönstret blir:

1. Behåll Oracle som primär sanning.
2. Bygg ny söktjänst på OpenShift.
3. Mata index via CDC initialt och events där det är möjligt.
4. Exponera ny sök via API.
5. Låt en pilotgrupp i kundtjänst testa.
6. Jämför resultat och prestanda med befintlig sök.
7. Flytta fler användare stegvis.
8. Avveckla gammal sökväg när kvaliteten är bevisad.

Detta testar flera målarkitekturdelar utan att flytta hela orderkärnan. Organisationen lär sig om OpenShift, Elasticsearch, dataflöden, observability och supportmodell i ett kontrollerat område.

## Snabb sammanfattning

- Strangler pattern möjliggör modernisering utan big bang.
- Inkapsling är ofta ett bättre första steg än ersättning.
- Första kandidaten ska ge affärsvärde, lärande och hanterbar risk.
- Data, trafikstyrning och integration är de svåraste delarna.
- Parallellkörning kräver mätetal och slutdatum.
- Modernisering är inte klar förrän gamla vägar avvecklas eller medvetet behålls.
- ADR, observability och rollback är centrala för kontrollerad migrering.

## Nästa steg

Nästa kapitel behandlar förflyttningen från Java EE till cloud native Java. Där går vi närmare applikationens runtime, konfiguration, health checks, sessioner och de konkreta förändringar som krävs för att en Java-applikation ska fungera väl i OpenShift.
