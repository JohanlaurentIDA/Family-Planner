# Family OS – Version 1.0

Eine responsive React-/TypeScript-Web-App für vier Familienprofile, eigene Familientermine und eine gemeinsame Einkaufsliste. Mit Firebase Firestore werden Daten zwischen Geräten live synchronisiert. Ohne Firebase läuft ein lokaler Demo-Modus im Browser.

## Voraussetzungen

- Node.js 22.12+ (oder 20.19+) und npm
- Moderner Browser
- Für gemeinsame Nutzung: kostenloses Firebase-Projekt mit Firebase Authentication und Cloud Firestore

## 1. Auf Windows starten

ZIP entpacken. Im Ordner `family-os` PowerShell / Terminal öffnen:

```powershell
npm install
npm run dev
```

Die URL aus dem Terminal öffnen (typischerweise `http://localhost:5173`). Ohne `.env` wird automatisch der Demo-Modus angezeigt. Im Demo-Modus bleiben Daten nur im Browser (localStorage) und werden nicht zwischen Geräten synchronisiert.

## 2. Firebase einrichten (für geteilte Listen und Kalender)

1. https://console.firebase.google.com öffnen und ein neues Projekt `family-os` erstellen.
2. Projektübersicht -> **Web-App hinzufügen** (`</>`). Web-App registrieren. Die vier Konfigurationswerte `apiKey`, `authDomain`, `projectId`, `appId` kopieren.
3. **Authentication -> Sign-in method -> Email/Password** aktivieren.
4. **Firestore Database -> Datenbank erstellen**. Produktionsmodus wählen; Region nach Wunsch (z.B. EU-Region), Regeln im nächsten Schritt veröffentlichen.
5. In **Firestore Database -> Rules** den gesamten Inhalt von `firestore.rules` einfügen und **Publish** klicken. Die Regeln beschränken jeden Zugriff auf `/households/{uid}/...` auf den jeweils angemeldeten Firebase-Nutzer.
6. `.env.example` als `.env` kopieren und die vier Werte eintragen:

```env
VITE_FIREBASE_API_KEY=DEIN_WERT
VITE_FIREBASE_AUTH_DOMAIN=DEIN_WERT
VITE_FIREBASE_PROJECT_ID=DEIN_WERT
VITE_FIREBASE_APP_ID=DEIN_WERT
```

7. Entwicklungsserver neu starten (`Ctrl+C`, dann `npm run dev`). Auf der Anmeldeseite **Neues Familienkonto erstellen** wählen und eine Familien-E-Mail sowie ein sicheres, einzigartiges Passwort eintragen. Beim ersten Registrieren werden die vier Profile initialisiert.
8. Auf anderen Geräten **mit denselben Zugangsdaten** anmelden. Termine und Einkaufsliste sind dann synchronisiert (solange die Geräte Internet haben).

**Sicherheit:** In Version 1 teilen alle Geräte ein einziges Familienkonto. Die vier Profile sind Kennzeichnungen, keine getrennten Logins oder Berechtigungen. Jeder, der das Konto kennt, kann alle Daten bearbeiten/löschen. Für den Familienbetrieb geeignete Passwörter und einen Passwortmanager verwenden. Bei mehreren echten Benutzerkonten müssen Datenmodell, Familienmitgliedschaften, Einladungen und Security Rules erweitert werden. Firebase-Konfigurationswerte für die Web-App sind öffentliche Client-Konfiguration, **keine** Admin-Schlüssel. Niemals Service-Account-Keys in `.env` mit `VITE_`-Präfix schreiben oder in den Browser einbauen.

## 3. Bedienung

- **Kalender:** Eine Woche vor/zurück, auf heute springen, nach Person filtern, Termin hinzufügen/löschen.
- **Einkaufen:** Artikel hinzufügen, als erledigt markieren, löschen; optional angeben, wer ihn eingetragen hat.
- **Einstellungen:** Vier Familienprofile umbenennen.
- **Demo:** Daten bleiben auf diesem einen Gerät in localStorage. Eine spätere Firebase-Anmeldung übernimmt die Demo-Daten **nicht automatisch**.

**Noch nicht enthalten:** Google Calendar-Synchronisation, einzelne Nutzerkonten, Push-Nachrichten, KI-Assistent, Offline-Synchronisierung oder automatisierte Tests. Eigene Termine sind derzeit unabhängige Firestore-Einträge.

## 4. Auf Tablet / im Heim-WLAN testen

Laptop und Tablet mit demselben WLAN verbinden. Im Terminal zeigt Vite nach `npm run dev` normalerweise eine Network-Adresse wie `http://192.168.x.x:5173`. Diese Adresse im Tablet-Browser öffnen. Ggf. Windows-Firewall für das private Netzwerk freigeben. Der Laptop muss für diesen lokalen Zugriff eingeschaltet bleiben. Für den **dauerhaften** Einsatz eine Hosting-Lösung verwenden.

## 5. Produktiv bereitstellen (Firebase Hosting)

Auf dem Entwicklungscomputer:

```bash
npm install
npm run build
npm install -g firebase-tools
firebase login
firebase init hosting
```

Bei den Fragen: **Use an existing project** -> das erstellte Projekt, **public directory** = `dist`, **single-page app** = `Yes`, **overwrite index.html** = `No`. Danach:

```bash
firebase deploy --only hosting
```

Dann die angezeigte `https://...web.app`-Adresse auf Tablets und Smartphones öffnen. Das Tablet kann diese URL im Browser auf den Startbildschirm legen. Für einen Kiosk-Modus eignet sich beispielsweise Fully Kiosk Browser. **Hinweis:** Firebase Hosting benötigt in der Regel keine eigene Serverhardware; aktuelle Hosting-Tarife, Kontingente und Projektanforderungen in der Firebase-Konsole prüfen.

## 6. Fehlerbehebung

- `Missing or insufficient permissions`: Firestore Rules prüfen, Regeln publizieren, mit Familienkonto anmelden.
- `auth/operation-not-allowed`: Email/Password unter Firebase Authentication aktivieren.
- Anmeldung klappt, keine Synchronisierung: `.env` prüfen, Dev-Server neu starten; beide Geräte müssen dasselbe Firebase-Projekt und **denselben Login** verwenden.
- `npm` unbekannt: Node.js installieren und Terminal neu öffnen.
- Port belegt: Vite zeigt dann einen anderen Port an.

## Erweiterungsfahrplan

1. Individuelle Firebase-Accounts + sichere Haushalts-Einladungen und Membership-Rules
2. Bearbeiten wiederkehrender Termine, Tagesansicht und Erinnerungen
3. Google Calendar per OAuth mit expliziten Berechtigungen
4. KI-Assistent über Cloud Functions / Backend (niemals geheimen API-Schlüssel im Frontend)
5. PWA-Manifest, Service Worker und Kiosk-Automatik
