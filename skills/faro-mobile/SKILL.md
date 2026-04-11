---
name: faro-mobile
description: |
  Pattern Expo/React Native: navigazione, gesture, notifiche push, app store.
  Trigger: "expo", "react native", "app mobile", "push notification", "app store"
---

# faro-mobile — iOS · Android · App Store · Play Store

**Lingua:** Sempre italiano. Riferimenti ufficiali:
- Expo: https://docs.expo.dev
- React Native: https://reactnative.dev/docs/getting-started
- Expo Router: https://docs.expo.dev/router/introduction
- EAS Build: https://docs.expo.dev/build/introduction

## QUANDO USARE

- Nuova app mobile iOS/Android
- Setup Expo Router, auth mobile, push notifications
- Build produzione con EAS
- Submission App Store (iOS) o Google Play (Android)
- OTA updates senza passare per i store

## FLUSSO DECISIONALE

```
Nuovo progetto → Expo managed workflow (sempre, poi ejetti solo se necessario)
Auth → Supabase Auth nativo (consigliato) o Clerk (social login semplice)
Token sensibili → sempre in SecureStore, mai in AsyncStorage
Build → EAS Build (development → preview → production)
Deploy → OTA update per JS/assets, nuova submission per nativo
```

## REGOLE CHIAVE

1. **Inizia con Expo managed** — ejetti solo se serve libreria nativa non supportata
2. **Token in SecureStore** — mai in AsyncStorage per dati sensibili
3. **`detectSessionInUrl: false`** per Supabase su mobile — obbligatorio
4. **`EXPO_PUBLIC_` solo per variabili pubbliche** — senza prefisso = solo server-side
5. **OTA update** per fix JS/assets, **nuova submission** per cambio nativo
6. **Privacy policy URL obbligatorio** su entrambi i store — senza di essa → rejection
7. **Account demo per revisori Apple** — senza di esso → rejection quasi certa
8. **App Bundle `.aab`** per Android in produzione, non `.apk`
9. **Testa su dispositivo fisico** prima della submission — non solo simulator
10. **`autoIncrement: true`** in `eas.json` production per incremento automatico versione

## FLUSSO OPERATIVO

1. **Init**: `npx create-expo-app` con template TypeScript
2. **Navigation**: Expo Router con layout groups
3. **Auth**: Supabase + SecureStore adapter
4. **Push**: `expo-notifications` + registrazione token backend
5. **Build**: `eas.json` con 3 profili, `eas build --profile production`
6. **Submit**: `eas submit` per iOS/Android
7. **OTA**: `eas update` per fix rapidi

## CHECKLIST

- [ ] `app.json`: bundleIdentifier iOS e package Android corretti
- [ ] Icona 1024×1024 px (no trasparenza iOS, no angoli arrotondati)
- [ ] Splash screen configurato
- [ ] Auth con SecureStore per token
- [ ] Push notifications registrate e token salvato nel backend
- [ ] `eas.json` con profili development / preview / production
- [ ] Privacy policy URL accessibile online
- [ ] Screenshot per ogni schermata principale
- [ ] Testata su dispositivo fisico (min 2 per iOS)
- [ ] Account demo funzionante per revisori store
- [ ] Monitoraggio crash configurato (Sentry / Expo Crashlytics)

## TABELLA — EAS Build profiles

| Profilo | Uso | Firmato? | Distribuzione |
|---------|-----|----------|---------------|
| development | Dev con hot reload nativo | Dev cert | Expo Dev Client |
| preview | QA/tester | Distribution cert | Internal link |
| production | Store release | Distribution cert | App Store / Play Store |

## REFERENCES

Per dettagli tecnici completi, leggi:
- [references/expo-patterns.md] — setup Expo, navigation, auth Supabase mobile, push notifications, EAS build config, submission iOS/Android, OTA updates, motivi rejection
