# Från Devin till din Mac och appbutikerna

## Rekommenderat flöde

**Devin → ditt GitHub-repository → Expo EAS-bygge → TestFlight/Google Play-testning → godkänd publicering.**

**För lokal utveckling:** GitHub → klona på din Mac → installera beroenden → generera iOS-projekt → öppna projektets `.xcworkspace` i Xcode.

Du behöver alltså inte skicka "iOS-filer" manuellt mellan datorer efter varje ändring. GitHub håller reda på exakt vilken källkod som hör till varje version. Expo kan bygga och ladda upp iOS från molnet; lokal kompilering och nativefelsökning med Xcode kräver macOS. Uppladdning till TestFlight är ett teststeg, inte automatisk App Store-publicering. Se [Expos publiceringsguide](https://docs.expo.dev/deploy/submit-to-app-stores/).

## 1. Konton du ska äga

| Konto/resurs | Användning | Vad Devin behöver |
|---|---|---|
| GitHub, privat repository | Källkod, historik, CI | Avgränsad repositoryåtkomst |
| Expo | Byggen och distribution | Projektåtkomst/token med minsta nödvändiga behörighet |
| Supabase | Staging och produktion | Miljöberoende åtkomst; privilegierade nycklar endast på server/CI |
| Webbhosting och domän | Arbetsgivarportal och support | Projektåtkomst, DNS-ändring vid rätt steg |
| Transaktionell e-post | Inloggningskoder och notiser | API/SMTP-hemlighet i säker lagring; verifierad avsändardomän |
| Apple Developer/App Store Connect | Signering, TestFlight och App Store | Delegerad roll och vid behov API-nyckel via säker kanal |
| Google Play Console | Android-testning och publicering | Avgränsad behörighet/servicekonto via säker kanal |
| Supportadress | Frågor, incidenter, datarättigheter | Riktig fungerande kontakt, inte påhittad adress |

Använd organisationskonton där det passar ditt faktiska bolag. Kontrollera aktuella kostnader, identitetskrav och handläggningstid på respektive tjänst innan du betalar. Dela inte Apple-lösenord, privata signeringsnycklar eller produktionshemligheter i Devin-prompten. Ha återställningsmetoder och MFA hos kontoägaren.

## 2. Överlämningspaketet från Devin

Kräv repositorylänk och tagg/commit, versionslista, miljömallar, databasändringar, testprotokoll, kända fel och en README som fungerar från en ny dator. Utöver detta ska du få build-ID/länkar där byggen verkligen har körts och dokumentation om vilken backend varje bygge använder.

Filerna har olika roller:

- **Källkod:** det du redigerar och äger; hela projektet krävs.
- **`.xcworkspace`:** genererat Xcode-arbetsprojekt efter beroendeinstallation, öppnas lokalt på Mac.
- **`.ipa`:** signerad iOS-app för en viss distributionsmetod; inte redigerbar projektkälla.
- **Simulator-`.app`:** avsedd för iOS-simulator, inte en vanlig fysisk iPhone.
- **`.aab`:** Android App Bundle för Google Play.
- **`.apk`:** kan användas för Android-testinstallation när korrekt byggd.

## 3. På din Mac

Devin ska anpassa och verifiera kommandona nedan för den faktiska leveransen. De är planerade exempel, inte körda instruktioner mot en byggd app.

1. Installera Xcode från Apple och de utvecklingsverktyg som den valda Expo-versionen kräver. Starta Xcode, acceptera villkoren och installera en kompatibel simulator.
2. Installera den Node-version och pakethanterare som repositoryt har låst. Följ projektets instruktioner för CocoaPods/Ruby om det behövs; gissa inte versionskombinationer.
3. Klona ditt repository och välj den testade taggen. Ska du börja ändra skapar du en ny gren från taggen.
4. Hämta tillåtna lokala miljövärden via säker kanal. Lägg inga serverhemligheter i mobilens miljöfil.
5. Installera låsta beroenden och kör projektets verifieringskommandon.
6. Starta Expo eller generera/bygg iOS-projektet och öppna rätt workspace.

Exempel, med faktiska värden ersatta innan körning:

```sh
git clone <DIN_REPOSITORY_URL>
cd <PROJEKTKATALOG>
git checkout <TESTAD_RELEASE_TAGG>
pnpm install --frozen-lockfile
pnpm verify
cd apps/mobile
pnpm exec expo prebuild --platform ios
pnpm exec expo run:ios
```

`pnpm verify` är ett repositoryskript Devin ska skapa och dokumentera. Projektet ska ha en fastslagen metod att installera rätt pnpm-version. Kör inte platshållarna ordagrant.

När du behöver Xcode: öppna den faktiska `.xcworkspace`-filen under `apps/mobile/ios/`, välj rätt scheme och simulator eller ansluten iPhone. För fysisk enhet: välj din signing team och korrekt bundle identifier. Exakt workspace/scheme ska stå i `MAC_HANDOVER.md`, inte gissas i efterhand.

Med Expo CNG är genererade nativekataloger normalt härledda från appkonfiguration och plugins. Direkta ändringar i dem kan försvinna vid regeneration. Spara därför avsiktliga ändringar i rätt källkonfiguration och undvik rutinmässig `prebuild --clean` över manuellt arbete. [Expo förklarar CNG här](https://docs.expo.dev/workflow/continuous-native-generation/).

## 4. Testdistribution

Devin ska skapa och dokumentera separata byggprofiler, exempelvis `development`, `ios-simulator`, `preview` och `production`. Varje profil har explicit miljö och ändamål. Kontrollera aktuell syntax vid genomförandet.

Exempel från mobilkatalogen när konton och projekt är konfigurerade:

```sh
eas build --platform ios --profile production
eas build --platform android --profile production
```

Välj sedan uttryckligen rätt build-ID för uppladdning. Använd inte slentrianmässigt "senaste" bygget när flera miljöer finns. Dokumentera den faktiska kommandoraden och verifiera backend/versionsnummer före distribution.

iOS: ladda upp rätt signerad distribution till App Store Connect, vänta på bearbetning och bjud in TestFlight-testare enligt gällande process. Android: använd internt testspår först, följ därefter relevanta krav för kontot. Nya personliga Google Play-konton kan omfattas av krav på minst 12 anslutna testare under 14 sammanhängande dagar innan ansökan om produktionsåtkomst; kontrollera ditt konto. [Googles testkrav](https://support.google.com/googleplay/android-developer/answer/14151465?hl=en).

## 5. Testa på riktigt

Testa minst en mindre/äldre och en nyare iPhone samt minst två olika Android-enheter, inom den OS-matris Devin fastställer utifrån kompatibel SDK. Gör kärnresan med riktiga testkonton mot staging, aldrig med privata kandidatuppgifter i demo.

Kontrollera e-postkod, profil, ansökan, match, chatt, push i bakgrund, notisens deep link efter utloggning, textstorlek, internetavbrott, återkallande, blockering och radering. Ange enhet, OS, commit och resultat. Saknas fysisk Mac/enhet ska detta kvarstå som ej testat; det får inte bockas av genom webbtest.

## 6. Före offentlig release

- Ägaren godkänner version, målmiljö, kostnader och publicering.
- Konton, signering, app-ID:n, ikoner, skärmbilder och beskrivning är korrekta.
- Integritetspolicy, användarvillkor, kontakt och publik raderingssida har verkliga adresser och ansvarig verksamhet.
- Apple privacy-uppgifter, Google Data safety och SDK-/behörighetsdeklarationer motsvarar faktisk kod och faktisk databehandling.
- Anmäla/blockera/moderera fungerar och någon bemannar ärendena.
- Granskaren får fungerande teståtkomst utan att exponera andra användares data.
- Produktion har säkerhetskopiering, övervakning, budgetlarm och utsedd ansvarig.
- Appar från testspåret har testats mot den avsedda konfigurationen och releasegrinden är uppfylld.

Apple ställer krav på bland annat hantering av användargenererat innehåll och kontoradering. [App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/) och [kontoradering](https://developer.apple.com/help/app-review/guideline-reference/5-1-1-account-deletion/). Google kräver också raderingsmöjligheter för appar med kontoskapande. [Googles raderingskrav](https://support.google.com/googleplay/android-developer/answer/13327111?hl=en).

Ingen kan garantera butikernas godkännande i förväg. Betalfunktioner, social inloggning eller nya SDK:er kräver ny policykontroll innan de läggs till. V1 använder egen e-postinloggning och inga mobilköp.

## 7. Efter lansering

Släpp stegvis, följ krascher, ansökningsfel och support, och behåll föregående kompatibla version. Leverera säkerhetsfixar och SDK-uppdateringar löpande. Ändringar i nativeberoenden kräver nytt nativebygge. Inför inte överluftsuppdateringar i första versionen utan separat rutin för kompatibilitet och rollback.

Faktisk överföring till din Mac återstår tills koden är byggd och du har klonat repositoryt eller gett auktoriserad åtkomst. Den här planen är ingen bekräftelse på att filer redan har överförts.
