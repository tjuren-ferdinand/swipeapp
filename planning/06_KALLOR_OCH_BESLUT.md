# Källor, antaganden och beslut

Kontrollerat 25 oktober 2026. Teknisk dokumentation och butikspolicyer ändras; Devin ska kontrollera dem igen inför implementation/release. Länkarna är underlag för specificerade fakta, inte certifiering av den färdiga produkten.

## Primärkällor

| Källa | Vad den stödjer i planen |
|---|---|
| [Devin: When to Use Devin](https://docs.devin.ai/essential-guidelines/when-to-use-devin) | Tydlig avgränsning och instruktion förbättrar komplexa utvecklingsuppdrag |
| [Devin: GitHub integration](https://docs.devin.ai/integrations/gh) | Repositoryintegration, åtkomst och pull requests som överlämningsflöde |
| [Expo: Submit to app stores](https://docs.expo.dev/deploy/submit-to-app-stores/) | Molnbaserad uppladdning, signerade binärer och skillnaden mellan TestFlight och publicering |
| [Expo: CLI](https://docs.expo.dev/more/expo-cli/) | Lokal iOS-körning och Xcode-felsökning |
| [Expo: CNG](https://docs.expo.dev/workflow/continuous-native-generation/) | Genererade nativeprojekt från konfiguration och plugins |
| [Flutter: platforms](https://docs.flutter.dev/platform-integration) | Flutter är ett reellt alternativ för flera plattformar |
| [Supabase: regions](https://supabase.com/docs/guides/platform/regions) | Specifika regionval och begränsningen i att likställa region med dataskyddsefterlevnad |
| [Apple: App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/) | Innehåll, rapportering/blockering, granskning och integritetskrav |
| [Apple: account deletion](https://developer.apple.com/help/app-review/guideline-reference/5-1-1-account-deletion/) | Radering för appar med konton |
| [Google: deletion requirements](https://support.google.com/googleplay/android-developer/answer/13327111?hl=en) | Raderingsvägar i app och via webbresurs |
| [Google: testing requirements](https://support.google.com/googleplay/android-developer/answer/14151465?hl=en) | Kontoavhängiga krav för testning före produktionsåtkomst |
| [IMY: rekryteringssystem](https://www.imy.se/verksamhet/dataskydd/dataskydd-pa-olika-omraden/arbetsliv/rekryteringssystem-och-kompetensdatabaser/) | Nödvändiga uppgifter, rättslig grund, gallring, ansvar och bedömning av integritetsrisker |
| [EU: AI Act, skäl 57](https://ai-act-service-desk.ec.europa.eu/en/ai-act/recital-57) | Rekryterings-AI kan innebära högriskklassificering och diskrimineringsrisk |

## Slutsatser av underlaget

Expo väljs genom en teknisk bedömning för den här produkten, inte för att en källa säger att det alltid är bäst. GitHub är källkodens överlämningsväg; byggtjänst och Xcode är olika sätt att bygga och kontrollera den. Devin behöver rätt konton och faktisk exekveringsmiljö för att verifiera slutresultatet.

Rekryteringsdata ska begränsas till det som behövs, och ansvar/rättslig grund/gallring måste fastställas. De föreslagna lagringstiderna i planen är produktförslag. Vi behandlar inte ett kryss i onboarding som generell rättslig grund för all arbetsgivarhantering. [IMY:s vägledning](https://www.imy.se/verksamhet/dataskydd/dataskydd-pa-olika-omraden/arbetsliv/rekryteringssystem-och-kompetensdatabaser/).

AI som påverkar urval av kandidater kan omfattas av högriskregler. Därför omfattar v1 tydliga användarfilter, kronologisk sortering och mänskliga beslut. Det är en avgränsning av funktionalitet, inte ett generellt juridiskt undantag för produkten. Vid framtida AI-bedömning behövs separat klassificering och kontroll av då gällande regler och tidpunkter. [EU:s beskrivning](https://ai-act-service-desk.ec.europa.eu/en/ai-act/recital-57).

## Beslut som redan är fattade för byggplanen

- iOS/Android med Expo; arbetsgivare på responsiv Next.js-webb.
- Supabase/PostgreSQL, specifik EU-region och separata miljöer.
- Svenskspråkig pilot för restaurang/café i Stockholm; engelska förbereds i struktur men översätts inte fullt i v1.
- Kompetensprofil utan foto/CV/video; kandidatens första ansökan öppnar arbetsgivarens tillgång till relevant profil.
- Ingen AI-rankning, automatisk kandidatbortsortering eller öppen kandidatdatabas.
- Gratis kandidatprodukt och kostnadsfri kontrollerad pilot; abonnemang prövas efter värdevalidering.
- Manuellt verifierade arbetsgivare och modererade annonser under pilot.

## Ägarbeslut före verklig pilot eller publicering

| Beslut | När det behövs | Kan Devin fortsätta under tiden? |
|---|---|---|
| Slutnamn, domän, varumärkeskontroll | Före publikt material och permanenta app-ID:n | Ja, med tydligt arbetsnamn |
| Bolag och utgivaridentitet | Konto-/avtalsregistrering och butik | Ja, lokal utveckling |
| Budgettak och leverantörsplaner | Före betaltjänst eller resursuppgradering | Ja, lokal/staging inom redan godkända ramar |
| Konton, behörigheter, signerande team | Riktiga byggen och distribution | Ja, oberoende kod/test |
| Ansvarsroller, rättslig grund, retention, eventuell konsekvensbedömning | Före behandling av riktiga kandidatdata | Ja, med syntetiska data |
| Support/modereringsansvarig | Före pilot | Ja |
| Faktiska pilotarbetsgivare och annonser | Före kandidatanskaffning | Ja, men ingen påhittad efterfrågan |
| Releasegodkännande | Före offentlig publicering | Ja, färdigställ och verifiera underlaget |

## Kvarvarande osäkerhet

Inga kundintervjuer, konkurrenttester, varumärkessökningar, inköpsbeslut eller juridiska godkännanden har genomförts i det här uppdraget. Budget, tid, pris och pilotmål är tydligt märkta planeringsantaganden. Inga framtida butiksgodkännanden, agentresultat eller överföringar till din Mac kan förutsättas.
