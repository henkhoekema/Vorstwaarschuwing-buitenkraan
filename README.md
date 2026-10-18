# 🥶 Vorstwaarschuwing voor de buitenkraan

Met deze Home Assistant-automatiseringen krijg je een melding wanneer het buiten koud genoeg wordt om rekening te houden met vorst.
De melding vraagt je om de buitenkraan af te sluiten. Je kunt op je telefoon bevestigen dat dit is gebeurd, aangeven dat de kraan nog openstaat of vragen om later een herinnering te krijgen.

Op 1 april krijg je een herinnering om de buitenkraan weer open te zetten. Zo hoef je er niet zelf aan te denken om de winterstand weer ongedaan te maken.

## Hoe werkt het?

De template-sensor kijkt naar de temperatuur. Zodra de temperatuur vijf minuten lang lager is dan 8 °C, wordt `binary_sensor.vorst_risico` ingeschakeld. De sensor schakelt pas weer uit als de temperatuur dertig minuten lang boven de grens komt. Dat voorkomt dat de status steeds heen en weer springt bij kleine temperatuurschommelingen.

Bij vorstrisico stuurt Home Assistant overdag een melding naar je telefoon. De ernst van de melding hangt af van de temperatuur:

- **Boven 5 °C:** een gewone waarschuwing.
- **5 °C of lager:** een serieuze waarschuwing.
- **3 °C of lager:** een dringend alarm.

Zolang de kraan nog niet als afgesloten is bevestigd en er aan de voorwaarden wordt voldaan, volgt er opnieuw een herinnering. De meldingen gebruiken dezelfde tag, zodat een nieuwe melding de vorige vervangt in plaats van een lange rij meldingen te maken.

Op 1 april volgt een melding om de kraan weer open te zetten. Ook die melding kan worden herhaald totdat je bevestigt dat de kraan weer openstaat.

> **Let op:** deze automatisering meet niet of de kraan fysiek open of dicht staat. Je bevestigt dat zelf via de knoppen in de melding.

## Wat heb je nodig?

- Home Assistant.
- Een temperatuursensor die de buitentemperatuur meet.
- De Home Assistant Companion-app op je telefoon, met meldingen ingeschakeld.
- Eén helper van het type **Invoerboolean** (`input_boolean`).

## Helper aanmaken

Maak via **Instellingen → Apparaten & diensten → Helpers → Helper aanmaken** een helper van het type **Schakelaar (invoerboolean)** aan.

Gebruik bij voorkeur deze naam en entiteits-ID:

- Naam: `Water afgesloten`
- Entiteits-ID: `input_boolean.water_afgesloten`

Deze helper houdt bij wat je zelf hebt bevestigd:

- `on`: je hebt bevestigd dat de buitenkraan is afgesloten.
- `off`: de kraan is niet als afgesloten bevestigd.

De helper is dus een registratie van je bevestiging, geen meting van de werkelijke kraanstand.

## Pas de configuratie aan jouw installatie aan

### 1. Vervang de temperatuursensor

In de template en in de berichtteksten staat deze voorbeeldentiteit:

```yaml
sensor.temperatuur
```

Vervang **alle** voorkomens door de entiteits-ID van jouw eigen temperatuursensor, bijvoorbeeld `sensor.buitentemperatuur`. Gebruik de entiteits-ID zoals die in Home Assistant staat.

De template gebruikt `float(99)` als terugvalwaarde. Als de sensor geen geldige temperatuur heeft of `unavailable` is, wordt de waarde 99 gebruikt. Daardoor ontstaat er niet zomaar een vorstwaarschuwing door een defecte of tijdelijk onbeschikbare sensor.

### 2. Vervang de mobiele notificatieservice

Vervang **alle** voorkomens van:

```yaml
notify.mobile_app_<your_id_here>
```

door de notificatieservice van jouw telefoon. Je vindt de juiste naam doorgaans onder **Ontwikkelaarstools → Acties**, door te zoeken op `notify.mobile_app`.

Bijvoorbeeld:

```yaml
notify.mobile_app_henks_telefoon
```

Gebruik de service die bij jouw eigen telefoon hoort. Als je meldingen naar meerdere telefoons wilt sturen, moet je de acties daarvoor aanpassen.

### 3. Pas de sensor- en automationbeschrijvingen aan

Je kunt de namen en beschrijvingen aanpassen aan je eigen installatie. De namen zijn vooral bedoeld om de configuratie in Home Assistant herkenbaar te houden. Verander de `unique_id` van de template-sensor alleen als je bewust een nieuwe entiteit wilt aanmaken.

De gebruikte entiteits-ID's en helper moeten wel overeenkomen met wat in de automatiseringen staat:

```yaml
binary_sensor.vorst_risico
input_boolean.water_afgesloten
```

Als Home Assistant bij jou andere entiteits-ID's gebruikt, pas die dan op alle betreffende plekken aan.

## Knoppen in de meldingen

De wintermelding bevat deze acties:

- **JA** — bevestigt dat de buitenkraan is afgesloten.
- **NEE** — stuurt een extra waarschuwing om de kraan af te sluiten.
- **Herinner me later** — stuurt later opnieuw een herinnering.

De voorjaarsmelding bevat:

- **JA** — bevestigt dat de buitenkraan weer openstaat.
- **Herinner later** — stuurt later opnieuw een herinnering om de kraan open te zetten.

De actieknoppen worden afgehandeld door aparte automatiseringen. Laat de actienamen (`KRAAN_JA`, `KRAAN_NEE`, `KRAAN_LATER`, `KRAAN_OPEN_JA` en `KRAAN_OPEN_LATER`) daarom exact overeenkomen in de meldingen en in de bijbehorende automatiseringen.

## Belangrijke controle vóór gebruik

De helper moet consequent worden gebruikt: `on` betekent **afgesloten bevestigd** en `off` betekent **niet afgesloten bevestigd**. Controleer daarom in de vorstwaarschuwing dat de voorwaarden voor de eerste melding en de herhaalmeldingen alleen doorgaan zolang de helper `off` is. Na de actie **JA** wordt de helper `on`; daarmee moeten de winterherinneringen stoppen.

Voor de herinnering op 1 april is de omgekeerde situatie nodig: die herinnering loopt zolang de helper `on` is, totdat je bevestigt dat de kraan weer openstaat en de helper op `off` wordt gezet.

Controleer ook de actie **Herinner me later**: de herinnering moet alleen worden verstuurd als die nog relevant is. Zonder extra voorwaarden kan een uitgestelde herinnering alsnog binnenkomen nadat de kraan al is afgesloten of het vorstrisico is verdwenen.

## Wanneer wordt er een melding gestuurd?

- Alleen in de periode **oktober tot en met april**.
- Alleen tussen **07.00 en 23.00 uur** voor de winterwaarschuwing.
- De eerste waarschuwing wordt gestuurd zodra de vorstsensor actief wordt, of bij het starten van Home Assistant als er nog vorstrisico is en de kraan nog niet als afgesloten is bevestigd.
- Herinneringen volgen met het ingestelde interval van vier uur.
- Op **1 april om 08.00 uur** start de herinnering om de kraan weer open te zetten. De herhaling staat in de aangeleverde configuratie op twee uur.

Let op: een Home Assistant-herstart hervat een al lopende `repeat`-lus met `delay` niet vanaf het punt waar die was gebleven. De starttrigger van de vorstwaarschuwing kan de eerste melding na een herstart opnieuw starten als aan de voorwaarden wordt voldaan. Controleer ook de uitgestelde herinneringen afzonderlijk als je wilt voorkomen dat die na een herstart of statuswijziging nog worden verstuurd.

## Testen

Test de configuratie voordat je erop vertrouwt:

1. Controleer in **Ontwikkelaarstools → Statussen** of de temperatuursensor een geldige waarde geeft.
2. Controleer of `binary_sensor.vorst_risico` de verwachte status krijgt wanneer je de temperatuurgrens test.
3. Controleer of `input_boolean.water_afgesloten` na de knop **JA** op `on` komt.
4. Controleer of de winterherinneringen daarna stoppen.
5. Test de actie **NEE** en **Herinner me later**.
6. Test de voorjaarsmelding en controleer of **JA** de helper weer op `off` zet.

Je kunt voor tests tijdelijk de temperatuurgrens of de wachttijden aanpassen. Zet die na het testen weer terug.

---

**Tip:** begin met de template-sensor en de helper. Voeg daarna de automatiseringen toe en controleer in Home Assistant of alle entiteits-ID's en notificatieservices overeenkomen met jouw installatie.
