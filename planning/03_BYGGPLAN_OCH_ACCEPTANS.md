# Byggplan och acceptans

## Leveransmodell

Varje milstolpe slutar med en körbar leverans, tester, skärmbilder där UI ändrats, ett tydligt ändringsunderlag och lista över verkliga hinder. Devin ska skilja mellan implementerat, testat lokalt, testat på enhet och publicerat. En webbpreview bevisar inte att iOS fungerar.

## M0 – Grund och beslut, ungefär vecka 1

- Läs hela specifikationen och skapa krav→implementation→test-matris.
- Bekräfta kompatibel teknik, versionslåsning och projektstruktur.
- Skapa lokal utvecklingsmiljö, syntetisk seed, CI och `.env.example` utan hemligheter.
- Dokumentera dataflöde, roller, hotmodell och öppna ägarfrågor.
- Skapa enkel klickbar navigation i app/webb, som del av den riktiga kodbasen.

Godkänt: en ny utvecklare kan installera och köra lokal app, webb och backend från dokumentationen; lint, typer och grundtest är gröna. Demo är tydligt märkt och kan aldrig aktiveras i produktion av misstag.

## M1 – Fullständig kärnresa, ungefär vecka 2–3

Implementera autentisering, kandidatprofil, företag, manuell verifiering, annons, jobblista, ansökan, arbetsgivarintresse och match med verklig databas. Lägg säkerhetsregler samtidigt som tabellerna skapas.

Godkänt: två separata konton kan genomföra hela flödet; data finns kvar efter omstart. En tredje användare och ett annat företag kan inte läsa eller ändra data. Dubbeltryck ger en ansökan och en match. Stängd annons kan inte ta emot nya ansökningar.

## M2 – Färdig rekryteringsresa, ungefär vecka 4–5

Implementera swipe/knappar, filter, sparat, status, revisionshantering, chatt, intervjuförslag, push/e-post, teaminbjudningar och konflikthantering.

Godkänt: kandidat på telefon och arbetsgivare i webbläsare kan hitta, söka, matcha, chatta och boka intervju. Återanslutning tappar inga meddelanden. Notisfel förstör inte affärshändelsen.

## M3 – Säkerhet och kvalitet, ungefär vecka 6–7

Implementera hela modereringskedjan, blockering, rapportering, export/radering, retention, admin-MFA, tillgänglighet, säkerhetsgranskning och driftlarm. Färdigställ support och juridiska utkast för ägarens granskning.

Godkänt: testerna nedan passerar; dataradering verifieras i alla berörda tabeller och integrationsköer. Backup har återlästs till isolerad miljö. Ingen kritisk säkerhetsbrist kvarstår.

## M4 – Enheter, pilot och överlämning, ungefär vecka 8–10

Skapa iOS/Android-byggen, testa på riktiga enheter, distribuera till testspår när konton finns, kör pilot, åtgärda fel och leverera Mac-guiden. Reserv för omtag och extern granskning: vecka 11–12 och vid behov längre.

Godkänt: testprotokoll med enhet/OS/commit, releaseunderlag, källkod, migrationsfiler, build-ID:n och dokumentation. Saknade signeringskonton ska anges som blockerare, aldrig döljas bakom en skärmbild av webbappen.

## Obligatoriska tester

| ID | Situation | Förväntat utfall |
|---|---|---|
| A01 | Kandidat registrerar sig, verifierar och loggar in igen | Profil och session fungerar; kod kan inte återanvändas |
| A02 | Kandidat A begär B:s profil/snapshot med manipulerat ID | Nekas utan privat dataläckage |
| A03 | Företag A begär B:s jobbansökningar, meddelanden eller rapporter | Nekas i databas/API, inte bara UI |
| A04 | Rekryterare försöker göra sig själv till ägare/admin | Nekas; ingen klientstyrd rolländring |
| A05 | Två samtidiga ansökningar med samma eller olika idempotensnycklar | Exakt en ansökan genom unik constraint |
| A06 | Två rekryterare visar intresse samtidigt | En match, en konversation och deduplicerad outboxhändelse |
| A07 | Annons stängs samtidigt som ansökan skickas | Definierad transaktionsordning; ingen ansökan efter stängningscommit |
| A08 | Kandidaten återkallar samtidigt som arbetsgivaren matchar | Slutligt återkallad ansökan kan inte få aktiv chatt; båda ordningarna testas |
| A09 | Medlemskap tas bort med aktiv session/realtime | Nästa åtkomst nekas; inga nya events till borttagen medlem |
| A10 | Jobbet ändrar lön efter ansökan | Tidigare revision bevaras enligt retention, ändringsinformation visas |
| A11 | Internet bryts under ansökan eller meddelande | Ingen falsk kvittens; återförsök duplicerar inte data |
| A12 | Realtime kopplas bort och återansluts | Missade meddelanden hämtas i rätt ordning utan dubletter |
| A13 | Kandidaten blockerar organisationen | Nya meddelanden, notiser och ansökningar stoppas även via direkta API-anrop |
| A14 | Två intervjusvar eller gammalt tidsförslag | Versionskonflikt hanteras; en accepterad aktuell tid |
| A15 | Tidszon/dag med sommar-/vintertidsbyte | Samma verkliga tid för båda parter, tydlig lokal presentation |
| A16 | Avstängt företag försöker publicera eller kontakta | Nekas, annonser döljs och köade utskick kontrolleras |
| A17 | Rapport om bedrägeri skickas | Kvittens, synligt adminärende, loggat beslut och fungerande avstängning |
| A18 | Export begärs | Bara rätt persons avsedda data, kortlivad säker hämtning |
| A19 | Kontoradering med aktiva ansökningar/chattar | Data, snapshots, tokens och köer hanteras enligt policy; ingen vanlig åtkomst kvar |
| A20 | Återställning av backup efter tidigare radering | Raderade personer återaktiveras inte |
| A21 | Notisleverantör är nere | Ansökan/match/chatt fungerar, återförsök och larm finns |
| A22 | Gäst, kandidat, rekryterare, ägare, admin | Läs/skriv/delete-matris körs även för views, RPC och privata filer |
| A23 | Ägare vill lämna eller radera sitt konto | Ägaröverföring/stängning och personlig radering fungerar utan återvändsgränd |
| A24 | Engångsinbjudan används av fel e-post eller två gånger | Nekas; giltig mottagare får exakt avsedd roll |
| A25 | För stora payloads, HTML, missbruksvolym och länkar | Begränsas/saneras som text; ingen kodexekvering eller hemlig data i loggar |
| A26 | Återanvänd tidigare deep link efter logout/blockering | Kräver autentisering och aktuell rättighet; ingen data i URL |
| A27 | Stor text, skärmläsare, tangentbord, reducerad rörelse | Alla kärnresor kan utföras utan swipe eller färgtolkning |
| A28 | Gammal mobilversion mot ny backend | Kärnflödet fortsätter eller får tydligt kompatibilitetsbesked |
| A29 | Sökflöde utan träffar och tom ansökningsinkorg | Hjälpsamt tomt läge, inga falska profiler eller annonser |
| A30 | Fel miljö-/demo-inställning i releasebygge | Bygg/releasekontroll stoppar fel backend och testdata |

## Testnivåer och verktyg

- Enhetstester för tillståndsövergångar, validering, tidskonvertering och idempotensregler.
- Databasintegrationstester med verklig lokal PostgreSQL/Supabase, autentiserade roller och samtidiga anrop. Mocks räcker inte för RLS.
- Playwright för arbetsgivar-/adminwebb inklusive två organisationer och otillåten direktnavigering.
- React Native-komponenttester samt Maestro eller motsvarande för kärnresa på iOS och Android. Fastställ kompatibelt verktyg i M0.
- Manuella verkliga enhetstester för push, tangentbord, deep links, bakgrund/återstart och tillgänglighet.
- Lastprov mot staging med syntetiska konton; dokumentera volym, svarstider och felprocent.

## Releasegrind

Blockera release vid P0/P1-fel: dataläckage, behörighetsfel, kontoradering som inte fungerar, blockerad kontakt som går igenom, krascher i kärnresa eller felaktig målmiljö. Dokumentera mindre fel med ansvarig och plan. Granska dependency-/secret-scan-resultat, inte bara grön status.

Leveransmappen ska innehålla versionsnummer, commit SHA, databasversion, testresultat, verkliga screenshots, kända begränsningar, återställningsplan, aktuella buildlänkar, signeringsägare och driftskontakt. Ingen teststatus får hittas på. För varje ej körbart test: ange orsak, vem som kan köra det och exakt nästa steg.
