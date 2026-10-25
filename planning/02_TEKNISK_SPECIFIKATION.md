# Teknisk specifikation

## 1. Arkitekturbeslut

| Del | Val | Skäl och konsekvens |
|---|---|---|
| Kandidatapp | React Native, Expo, TypeScript, Expo Router | Gemensam kod för iOS/Android, stöd för nativebyggen och rimlig webbkompetensöverföring |
| Arbetsgivar- och adminwebb | Next.js, TypeScript | Desktopanpassade listor, formulär, arbetsflöden och delade datatyper |
| Databas/backend | Supabase PostgreSQL, Auth, Realtime, serverfunktioner | Relationer, transaktioner och radbehörigheter passar ansökningsdata |
| Geografi | Ort/område i v1 | Ingen exakt position eller extern karttjänst krävs |
| State/data | TanStack Query, lokal UI-state, schema-validering med Zod | Avgränsad cache och gemensamma kontrakt; servern förblir sanningskälla |
| Formulär | React Hook Form där kompatibelt | Validering, felmeddelanden och tillgänglighet |
| Bygg/CI | GitHub Actions och Expo EAS | Reproducerbara tester och signerade distributionsbyggen |
| Webbhosting | Hanterad Next.js-hosting med lämpligt avtal och EU-konfiguration | Validera personuppgiftsflöden och pris före val av leverantör |
| E-post | Transaktionell SMTP/API-leverantör med dokumenterat biträdesavtal | Engångskoder och begränsade händelsenotiser |
| Felövervakning | Sentry eller motsvarande med granskad datakonfiguration | Inga meddelandetexter eller tokenvärden i felrapporter |

Använd kompatibla stabila versioner vid byggstart och lås dem i lockfil. Expo SDK styr React Native-versionen; installera inte oberoende "senaste" versioner. Dokumentera Node, pakethanterare, SDK och build image. Undvik extra separat API-server, mikrotjänster och egen sökmotor innan behov finns.

Flutter är ett fullgott alternativ som stöder flera plattformar, men väljs inte här eftersom TypeScript kan delas med arbetsgivarwebben och teamet slipper både Dart och en separat webbstack. Det är ett projektspecifikt val, inte ett påstående att Expo alltid är bättre. Separata Swift/Kotlin-appar skulle ge två mobilkodbaser för en produkt som främst består av formulär, listor och chatt.

Supabase-projekt läggs i en specifik EU-region, helst Stockholm om aktuell plan stöder det, annars Frankfurt. En EU-databas innebär inte att alla underleverantörer eller supportflöden är inom EU. Kartlägg även loggar, e-post, Expo/push, felövervakning och hosting.

## 2. Repository

```text
apps/
  mobile/                  Expo-kandidatapp
  web/                     Next.js arbetsgivare, publika sidor, admin
packages/
  domain/                  Scheman, tillståndsregler, typer, felkoder
  api-client/              Typade klientanrop utan hemligheter
  design-tokens/           Färg, typografi, mellanrum
  test-fixtures/           Enbart syntetiska personer och företag
supabase/
  migrations/              Versionsstyrd SQL, constraints, grants och RLS
  functions/               Privilegierade funktioner och integrationsadaptrar
  tests/                   Databas-/behörighetstester
  seed.sql                 Syntetisk lokal/stagingdata
docs/                      Arkitektur, hotmodell, drift, release, Mac-guide
planning/                  Detta beställningsunderlag
.github/workflows/         CI utan hemligheter i källkod
```

Dela affärsregler och typer; tvinga inte identiska UI-komponenter mellan webben och mobilen. Ha separata lokal-, staging- och produktionsmiljöer, inklusive pushidentifiering och e-postinställningar. CI får aldrig automatiskt köra seed/reset mot produktion.

## 3. Datamodell

Alla tidsstämplar lagras i UTC; UI visar användarens tidszon. Använd UUID, `created_at`, `updated_at` och lämpliga index. PII-tabeller och auditlogg behandlas som personuppgifter. Tabellen nedan är ett logiskt kontrakt; Devin levererar faktisk SQL och migrationsprov.

| Tabell | Nyckelfält/constraints | Åtkomst |
|---|---|---|
| profiles | user_id PK/FK auth, display_name, locale, account_state | Egen användare; inga publika kandidatprofiler |
| candidate_profiles | user_id PK, city_id, availability, start_date, employment_types | Kandidat själv; arbetsgivare får separat ansökningsprojektion |
| candidate_experience | id, candidate_id, role, dates, short_description | Samma som profil |
| skills / candidate_skills | kontrollerad taxonomi, unique(candidate_id,skill_id) | Taxonomi läsbar, koppling privat |
| organizations | id, legal_name, registration_number, verification_state, suspended_at | Medlemmar; publik minimal företagsvy för publicerade jobb |
| organization_members | organization_id, user_id, role owner/recruiter, unique pair | Serverkontrollerad medlemskapshantering |
| organization_invitations | hashed_token, normalized_email, expires_at, accepted_at | Privilegierad skapa/acceptera, engångsanvändning |
| jobs | id, org_id, status, current_revision_id, closes_at | Publicerade aktiva annonser läsbara; orgmedlemmar ser egna utkast |
| job_revisions | id, job_id, revision, title, location, salary_min/max, currency, salary_unit, hours, requirements | Publicerad version publikt; historik enbart behöriga |
| saved_jobs | candidate_id, job_id, unique pair | Bara kandidaten |
| job_passes | candidate_id, job_id, passed_at, unique pair | Bara kandidaten; användaren kan återställa passeringar |
| applications | id, candidate_id, job_id, submitted_revision_id, profile_snapshot, status, version, retention_until, unique(candidate_id,job_id) | Kandidat och aktuella jobbets organisation enligt livscykel |
| application_events | application_id, type, actor_id, timestamp, safe_metadata | Relevant kandidat/org; interna säkerhetsfält separeras |
| matches | id, application_id unique, state, opened_at | Skapas bara via auktoriserad transaktion |
| conversations | id, match_id unique | Deltagare enligt aktuella medlemskap och matchstatus |
| messages | id, conversation_id, sender_id, body, client_message_id, created_at | Deltagare; unique(sender_id,client_message_id) |
| conversation_reads | conversation_id, user_id, last_read_message_id, unique pair | Egen deltagare |
| interview_proposals | id, match_id, proposer_id, starts_at, timezone, duration, mode, location_text, state, version | Matchens parter; accepterande transaktion |
| blocks | candidate_id, organization_id, created_at, unique pair | Kandidat och server; rapporterade parter får inte se rapportdetaljer |
| reports | reporter_id, target_type/id, reason, details, case_state | Rapportörens kvittens och administratör |
| moderation_cases | target, decision, reason, moderator_id | Administratör, revisionsspår |
| push_tokens | user_id, token, platform, environment, last_seen | Egen registrering, serverleverans; aldrig publik |
| notification_preferences | user_id, category, channel, enabled | Egen användare |
| notification_outbox | event_id, recipient_id, channel, dedupe_key unique, state, attempts, next_attempt_at | Enbart server |
| privacy_requests | user_id, type, state, verified_at, completed_at | Egen kvittens, behörig handläggare |
| audit_events | actor, action, target, timestamp, minimal metadata | Server skriver, begränsad admin läser |

Profilsnapshot innehåller endast de fält kandidaten såg i förhandsvisningen; aldrig e-post, authmetadata eller dolt födelsedatum. Den är en kopia med samma raderings- och retentionkrav som övriga kandidatdata, inte permanent historik. Ingen kandidat kan skriva `verification_state`, `role`, `candidate_id` för någon annan eller administrativa fält. Servern härleder aktör från autentisering.

## 4. Tillstånd och exakta affärsregler

### Jobb

`draft → pending_review → published → paused/closed/expired`. Avvisad granskning återgår till draft med motivering. Bara verifierad, aktiv organisation kan publicera. Serverklockan avgör deadline. Väsentliga ändringar skapar en ny revision och granskas innan de blir offentliga; den redan godkända revisionen kan ligga kvar om den inte är missvisande. Avstängning döljer alla organisationens annonser direkt.

### Ansökan

`submitted → interested → hired | rejected | withdrawn`. `submitted → rejected | withdrawn` är också tillåtet. `interested` skapar match och chatt. `hired` är arbetsgivarens uppgift med separat frivillig kandidatbekräftelse. Intervjutillstånd ligger i egen tabell, inte i samma statuskolumn. Arkivering/senare är en intern inkorgsegenskap och får inte döljas som ett avslag.

V1 har högst en ansökan per kandidat och jobb. Efter återkallande visas "Återkallad" och ingen ny ansökan till samma annons; arbetsgivaren kan publicera en ny tjänst vid ny rekrytering. Det är en medveten enkel regel. Ingen betalande användare får kringgå den.

När jobbet stängs stängs nya ansökningar direkt. Redan matchade konversationer kan fortsätta i högst 30 dagar eller tills en part avslutar dem. Icke matchade ansökningar visas som "Tjänsten avslutad" genom jobbstatus utan att skriva ett fiktivt personligt avslag. Retention avgör därefter hur länge data finns kvar.

### Intresse och swipes

Kandidatens högerswipe skickar ansökan. Arbetsgivarens högerswipe på den ansökan betyder mänskligt intresse. Matchning är därför ömsesidigt intresse för ett specifikt jobb. Arbetsgivaren ser inga personer som inte sökt och kan inte skicka kalla massmeddelanden.

Vänsterswipe på jobb är bara passering. "Ångra" kan ta tillbaka senaste passeringen. Återkallande av skickad ansökan är en separat handling med bekräftelse. En oavsiktlig högerswipe ska inte tyst raderas efter att arbetsgivaren sett den.

Blockering på kandidatnivå avser organisationen i v1: dölj jobb, stoppa nya ansökningar/kontakt/notiser och stäng skrivning i befintliga chattar. Ge tydlig förklaring före blockering. Organisationen kan avsluta en konversation och anmäla kandidaten; plattformen hanterar kontoavstängning. Avblockering återöppnar inte automatiskt gamla chattar.

### Intervjuer

Förslag innehåller tid, tidszon, längd, telefon/video/plats och valfri mötesinformation. Ingen kalenderintegration behövs. Mottagaren accepterar, avböjer eller lämnar motförslag. Ett accepterande låser versionen atomiskt så att två samtidiga svar inte skapar olika bokningar. Ny tid kräver nytt accepterande. Skicka ICS-fil via säker funktion eller e-post, med stabilt UID och uppdateringssekvens vid avbokning. UI ska varna för överlappande tider som appen känner till men lovar inte att kontrollera extern kalender.

## 5. Flöde och sortering

Filtrera på aktiva publicerade jobb, användarvalda orter/områden, anställningstyp, arbetstider och löneintervall. Första versionen använder synliga filter och stabil sortering på publiceringstid + ID; ingen AI, latent kompetenspoäng eller sannolikhet att bli anställd. Visa vilka filter som påverkar resultatet och låt dem återställas.

Arbetsgivarens ansökningar sorteras på inskickningstid. Rekryteraren kan välja filter för angivna arbetsrelevanta färdigheter och tillgänglighet, men ett filter avvisar aldrig någon. Visa att ett filter är aktivt och antalet dolda poster. Inga filter för kön, ålder, etnicitet, hälsa, religion eller antagna skyddade egenskaper. Begär inte medborgarskap; eventuellt framtida kontroll av arbetstillstånd kräver separat utformning.

## 6. API-/funktionskontrakt

Implementera typade serveroperationer eller PostgreSQL RPC med motsvarande kontrakt. Privilegierade operationer måste åter kontrollera användare, organisation och tillstånd. Publicera inte generiska skrivoperationer som kringgår regler.

| Operation | Input | Atomiskt resultat |
|---|---|---|
| list_jobs | filter, cursor, limit≤50 | Minimal publicerad projektion + nästa cursor |
| submit_application | job_id, revision_id, idempotency_key | Kontrollerar aktivt jobb/profil/blockering, sparar snapshot och en outboxhändelse |
| withdraw_application | application_id, expected_version | Återkallar, stänger matchskrivning, uppdaterar händelser/notiser |
| express_interest | application_id, expected_version | Kontrollerar medlemskap, skapar högst en match/chatt/notishändelse |
| reject_application | application_id, expected_version, reason_template | Mänskligt avslag, spärr för otillåten statusövergång |
| send_message | conversation_id, body, client_message_id | Kontrollerar deltagare, blockering, status och frekvens; skriver exakt en gång |
| propose/accept_interview | match_id/proposal_id, version, time fields | Rätt deltagare och giltig tid; versionskontroll |
| block_organization | org_id | Stoppar kontakt servermässigt och invaliderar aktiv åtkomst |
| invite/accept_member | org_id/email eller token | Ägarkontroll, tokenhash, utgångstid, ingen privilegieeskalering |
| request_export/deletion | reauthenticated user | Skapar säkert bakgrundsärende och kvittens |

Standardfel: `AUTH_REQUIRED`, `FORBIDDEN`, `NOT_FOUND`, `JOB_CLOSED`, `REVISION_CHANGED`, `ALREADY_APPLIED`, `CONFLICT`, `BLOCKED`, `RATE_LIMITED`, `VALIDATION_ERROR`. Obehöriga ska inte få veta om ett privat objekt finns. Fritextfel från SQL eller leverantörer ska inte visas för användaren.

Lås relevant jobbrad vid ansökan och vid stängning, så att deadline/stängning och inskickning har en definierad ordning. Samtidiga intresseanrop serialiseras per ansökan. Låt unika constraints och transaktioner skydda även vid flera serverinstanser. Använd förväntad version för rekryteraråtgärder; vid konflikt hämtas nytt tillstånd.

## 7. Säkerhet och kommunikation

- Row Level Security på samtliga exponerade tabeller; standard är ingen åtkomst. RLS omfattar SELECT, INSERT, UPDATE och DELETE samt eventuella lagringsobjekt.
- Lägg serverprivilegierade data i privat schema när möjligt. Skydda även views och RPC. Om `SECURITY DEFINER` behövs: fast `search_path`, uttrycklig aktörskontroll, minsta grants och ingen godtycklig SQL.
- Service role/servernycklar får aldrig förekomma i mobilpaket, `NEXT_PUBLIC_*`, `EXPO_PUBLIC_*`, webbundle, loggar eller repository. Publik klientnyckel är inte hemlig men kräver fungerande RLS.
- E-postkod får kort giltighet, återanvänds inte, och försöks-/utskicksgränser gäller både användare och IP. Felmeddelanden ska inte avslöja registrerade adresser. Testa kodåteranvändning och felaktig deep link.
- Mobil session lagras med lämplig säker lagring. Webb använder serveranpassad sessionshantering med säkra cookies, origin-/CSRF-skydd för cookieautentiserade mutationer och restriktiv säkerhetspolicy.
- Admin använder MFA, separat serverkontrollerad roll och loggade ingrepp. Organisationens ägare får inte ändra den sista ägarens behörighet utan överföring eller organisationens stängning.
- Medlem som lämnar organisation förlorar tillgång till alla organisationsdata och realtimekanaler. Kontrollera medlemskap vid varje operation; gammal JWT får inte räcka som permanent behörighet.
- Realtime är uppdateringssignal; servern lagrar sanningen. Hämta missade meddelanden med cursor efter återanslutning. Bekräfta inte meddelandet som skickat förrän servern accepterat det.
- Visa meddelanden som text, aldrig rå HTML. Max 2 000 tecken, initialt 20 meddelanden/minut/användare med konfigurerbar gräns. Ingen filöverföring i v1. Begränsa länkar från nya konton och flagga misstänkta mönster för mänsklig granskning.
- Kontroller vid publicering: otillåtna/diskriminerande annonser, bedrägerier, kontaktuppgifter som används för otillåten insamling, betalningskrav på sökande. Pilotens manuella granskning kompletterar grundläggande innehållsregler.
- Push har neutral text, exempelvis "Du har ett nytt meddelande". Inga namn, lön eller meddelandeinnehåll på låsskärmen. Deep link kontrollerar session och åtkomst på nytt.
- Outbox skrivs i samma transaktion som affärshändelsen. Arbetare återförsöker med backoff och dead-letter-kö. Intern deduplicering minskar dubletter, men extern pushleverans får inte beskrivas som garanterat exakt en gång.
- Om push eller e-post misslyckas ska match/chatt fortfarande fungera. Inaktiva tokens tas bort. Återkallande/blockering kontrolleras igen när en köad notis ska skickas.

## 8. Dataskydd som produktkrav

Fastställ vilka behandlingar plattformen respektive arbetsgivaren ansvarar för. Anta inte att plattformen alltid är biträde: egna konton, marknadsplats och missbruksbekämpning kan ha egna ändamål. Dokumentera rättslig grund per behandling och bedöm behov av konsekvensbedömning före pilot med riktiga data. Allmänt samtycke är inte en universallösning för rekrytering.

Produktförslag att låta ansvarig granska: gallra avslutade ansökningar och ansökningssnapshots efter 90 dagar, chatt 90 dagar efter avslut, driftsloggar efter 30 dagar och begränsad säkerhetslogg efter 180 dagar. Dessa är konfigurationsförslag, inte lagstadgade tider. Motiverade rättsliga undantag ska ha tydligt ändamål, egen behörighet och slutdatum. Backupens lagringstid bestäms utifrån vald tjänst och dokumenteras.

Kontoradering: verifiera användaren, blockera ny användning, återkalla sessioner och tokens, stoppa utskick, radera/anonymisera profil, snapshots, meddelanden och analyskopplingar enligt fastställd policy, hantera authkonto och bekräfta slutförande. Enbart soft delete räcker inte. För organisationsägare behövs tydlig överföring eller stängning av organisationen, utan att göra personlig radering omöjlig. Hantera minimalt motiverat bevarande separat. Efter återläsning av backup ska raderingsjournal återappliceras före att användarna släpps in.

Export: strukturerad JSON och läsbar sammanställning, kortlivad åtkomst och ny verifiering. Exportera inte andra personers privata uppgifter okontrollerat. Raderingslänk ska även finnas på publika webben. Publicera riktiga support- och integritetskontakter innan release.

## 9. Prestanda, drift och kapacitet

Designmål för pilot: 10 000 kandidatkonton, 1 000 aktiva jobb och 100 samtidiga användare. Detta är testmål, inte verifierad kapacitet. Använd cursorpaginering, index på status/ort/tid, ansökan per jobb, organisationsmedlemskap och meddelanden per konversation. Undvik att hämta hela kandidatbasen till klienten.

Målvärden i dokumenterad testmiljö: p95 för jobbflödes-API under 800 ms och skrivoperationer under 1 s utan externa notisleveranser. Chatten ska normalt synas inom 2 s på stabil anslutning. Mät kallstart och första interaktion på en faktisk äldre telefon, inte bara utvecklarens dator.

Driftmål i pilot: 99,5 procent tillgänglighet, RPO högst 24 timmar och RTO högst 8 timmar, förutsatt att valda backuper och bemanning stödjer det. Aktivera och prova backup/restore före pilot; redovisa uppmätt återställning. Larm på serverfel, authfel, växande outbox, raderingskö, ovanlig användning och budget. Driftansvarig och eskaleringskontakt måste utses.

Release sker med bakåtkompatibla databasändringar eftersom äldre mobilversioner finns kvar. Använd expand–migrate–contract och separata funktionsflaggor. SQL rollback får inte innebära att användardata raderas. Vid fel: stoppa ny funktion, rulla tillbaka webb/kod när kompatibelt och gör korrigerande migration.
