# Jobbmatchning med swipning – komplett byggunderlag

Framtaget 25 oktober 2026. Arbetsnamn: **Jobbmatch**. Namn, domän och varumärke är inte kontrollerade eller registrerade.

## Beslutet

Bygg en svensk rekryteringsplattform med React Native + Expo för iOS/Android, Next.js för arbetsgivarnas webbportal och Supabase/PostgreSQL för backend. Börja med restaurang och café i Stockholm som testmarknad. Detta är ett valt startantagande, inte en genomförd marknadsundersökning.

Kandidaten skapar en kort återanvändbar profil, hittar jobb med lön och arbetstider och skickar sin ansökan genom en tydligt förklarad högerswipe. Arbetsgivaren granskar ansökningar, visar intresse och öppnar därmed en matchningschatt. Inga personliga brev krävs. Ingen automatisk AI-bedömning av kandidater ingår i första versionen.

## Läsordning

1. [Produkt, affär och lansering](01_PRODUKT_OCH_AFFAR.md) – vad vi bygger och hur det ska få användare.
2. [Teknisk specifikation](02_TEKNISK_SPECIFIKATION.md) – datamodell, säkerhet, regler och integrationer.
3. [Byggplan och acceptanstester](03_BYGGPLAN_OCH_ACCEPTANS.md) – arbetsordning och vad som krävs för godkänt resultat.
4. [Huvudprompt till Devin](04_DEVIN_MASTER_PROMPT.md) – kopiera hela innehållet till Devin och bifoga övriga dokument.
5. [Mac, Xcode och publicering](05_MAC_XCODE_OCH_PUBLICERING.md) – konton, överlämning och release.
6. [Källor och beslut](06_KALLOR_OCH_BESLUT.md) – verifierade länkar, osäkerheter och ansvar.

## Så använder du paketet

Skapa ett privat GitHub-repository som du själv äger. Lägg dessa dokument i en katalog `planning/`. Anslut repositoryt till Devin och klistra in huvudprompten. Be Devin genomföra milstolparna i ordning med testresultat och körbara leveranser. En stor instruktion är en gemensam specifikation; den ersätter inte granskning mellan leveranserna.

Du behöver senare egna konton hos GitHub, Expo, Supabase, webbhosting, Apple Developer och Google Play Console. Hemliga nycklar läggs i respektive tjänsts hemlighetshantering, aldrig i prompten eller dokumenten. Konton, företagsnamn, domän, supportadress och utgivaridentitet måste tillhöra dig/ditt bolag.

## Vad som faktiskt är klart nu

Det här är ett planerings- och beställningspaket. Ingen app är byggd, ingen Devin-session har startats och inget har skickats till en Mac. Paketet anger exakt vad Devin ska leverera, hur det ska verifieras och hur du tar över koden. Saknad åtkomst till din Mac är inget hinder för att skriva planen, men faktisk överföring och signering kräver tillgång senare.

## Fem viktigaste principerna

- Lansera i en tät lokal marknad med riktiga lediga jobb innan bred annonsering.
- Kandidater betalar inte för att söka eller få bättre placering.
- Arbetsgivare ser bara kandidater som sökt deras jobb i första versionen.
- Ett matchningsbesked ska leda till verklig kontakt, inte bara fler swipes.
- Koden, kontona, databasen och publiceringsrättigheterna ska ägas av dig.
