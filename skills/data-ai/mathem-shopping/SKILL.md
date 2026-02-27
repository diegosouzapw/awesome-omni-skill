---
name: mathem-shopping
description: Automatiserar att logga in på Mathem.se, söka och lägga till varor från en lista eller recept, hantera ersättningar enligt policy och reservera leveranstid, men lämnar varukorgen redo för manuell checkout.
license: MIT
metadata:
  author: user
  version: "1.0"
allowed-tools: browser
---

# Mathem Shopping Agent

## När ska denna skill användas

Använd denna skill när användaren vill:

- Logga in på Mathem.se med sitt konto
- Söka och lägga varor i varukorg baserat på:
  - En specifik inköpslista
  - Ingredienser från ett recept
- Hantera ersättningar enligt `SUBSTITUTION_POLICY.md`
- Välja en lämplig leveranstid
- Lämna en färdig varukorg redo för användaren att avsluta köp

## Instruktioner

### 1. Logga in på Mathem.se

1. Gå till Mathem.se:s inloggningssida.
2. Använd de angivna användaruppgifterna för att logga in.
3. Säkerställ att sessionen är etablerad för följande steg.

**Villkor**

- Inloggning måste ske säkert.
- Inga inloggningsuppgifter får sparas utanför sessionshantering.

### 2. Tolka användarens lista eller recept

Agenten ska:

- Extrahera ingredienser från användarens lista eller recepttext.
- Normalisera mängder och enheter.
- Organisera sökfrågor för varje produkt.

### 3. Söka produkter

För varje ingrediens:

1. Använd Mathem.se:s sökfunktion.
2. Om exakt produkt finns → lägg i varukorgen.
3. Om exakt produkt **inte** finns → följ `SUBSTITUTION_POLICY.md`.

Rapportera:

- Eventuella byten
- Kombinerade förpackningar
- Saknade produkter

### 4. Hantera ersättningar

Agenten ska använda `SUBSTITUTION_POLICY.md` för att välja:

- Rimliga ersättningar
- Kombinera produkter för att nå önskad mängd
- Aldrig välja alternativ som strider mot policyn

Efter att hela listan är processad, sammanställ en rapport i det format som anges i policyn.

### 5. Välj leveranstid

1. Hämta tillgängliga leveranstider för användarens postnummer.
2. Matcha mot användarens önskemål om datum/tidsintervall.
3. Reservera en passande tid.
4. Bekräfta att reservationen är giltig (t.ex. att den inte försvinner direkt).

**Viktigt**

- Slutför **inte** betalningen.
- Reservationer på Mathem.se kan vara tidsbaserade; säkra reservationen men lämna checkout till användaren.

### 6. Lämna varukorgen redo

När alla varor är lagda och leveranstiden är vald:

- Skapa en sammanfattning:
  - Tillagda produkter
  - Ersättningar
  - Kombinationsdetaljer
  - Saknade produkter
  - Vald leveranstid
- Presentera detta för användaren.
- Ge en länk eller instruktion till användaren om hur de avslutar köpet.

## Exempel på användningscase

**Inköpslista**

> Users: "Jag behöver mjölk, ägg, grädde 40% och färsk koriander"

Agenten ska:

- Söka varje produkt
- Ersätta om ett alternativ matchar policyn
- Rapportera ersättningar i slutrapporten

**Receptbaserad handling**

> Users: "Lägg till alla ingredienser från det här receptet: [URL till recept]"

Agenten ska:

- Parsat receptet
- Extrahera ingredienser
- Söka och lägga till dem enligt ovan

## Rapportformat som ska skickas till användaren

**Tillagt utan ändring**

- Produkt A – vald produkt

**Ersatt**

- Produkt B (begärt) → Ersatt med Produkt C (vald)

**Kombinerat**

- Produkt D 300g → 2 × 150g (valda)

**Saknade**

- Produkt E (inget rimligt alternativ)

**Leveranstid**

- Datum och tidsintervall reserverat

**Sammanfattning**

> 🧾 Varukorgen är klar. Jag har gjort vissa ersättningar – granska dem gärna innan du slutför köpet.

---

## Policyreferenser

Se även de inkluderadePolicies:

- `references/SUBSTITUTION_POLICY.md`

---

## Begränsningar och ansvar

- Agenten ska aldrig göra betalningar.
- Agenten ska inte spara användarens lösenord utanför sessionshantering.
- Alla ändringar ska rapporteras tydligt.
