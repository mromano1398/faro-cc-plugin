# Expo Patterns — Faro Mobile

Pattern completi per app mobile iOS/Android con Expo + React Native.

## SETUP — Nuovo progetto

```bash
npx create-expo-app@latest faro-mobile --template
cd faro-mobile
npm run ios   # simulator iOS
npm run android # emulator Android
```

`app.json`:
```json
{
  "expo": {
    "name": "Faro",
    "slug": "faro-mobile",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "scheme": "faromobile",
    "userInterfaceStyle": "automatic",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.example.faro",
      "buildNumber": "1",
      "infoPlist": {
        "NSCameraUsageDescription": "Richiesta per caricare foto",
        "NSPhotoLibraryUsageDescription": "Richiesta per scegliere foto",
        "ITSAppUsesNonExemptEncryption": false
      }
    },
    "android": {
      "package": "com.example.faro",
      "versionCode": 1,
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "permissions": ["CAMERA", "READ_EXTERNAL_STORAGE"]
    },
    "plugins": [
      "expo-router",
      "expo-secure-store",
      ["expo-notifications", { "icon": "./assets/notification-icon.png" }]
    ]
  }
}
```

## NAVIGATION — Expo Router

Struttura file:
```
app/
  _layout.tsx           # Root layout
  index.tsx             # /
  (auth)/
    _layout.tsx
    login.tsx
    signup.tsx
  (tabs)/
    _layout.tsx         # Tab bar
    index.tsx           # Home
    profile.tsx
    settings.tsx
  [...not-found].tsx
```

`app/_layout.tsx`:
```tsx
import { Stack } from 'expo-router'
import { AuthProvider } from '@/lib/auth'

export default function RootLayout() {
  return (
    <AuthProvider>
      <Stack screenOptions={{ headerShown: false }}>
        <Stack.Screen name="(auth)" />
        <Stack.Screen name="(tabs)" />
      </Stack>
    </AuthProvider>
  )
}
```

`app/(tabs)/_layout.tsx`:
```tsx
import { Tabs } from 'expo-router'
import { Home, User, Settings } from 'lucide-react-native'

export default function TabsLayout() {
  return (
    <Tabs screenOptions={{ tabBarActiveTintColor: '#3B82F6' }}>
      <Tabs.Screen
        name="index"
        options={{ title: 'Home', tabBarIcon: ({ color }) => <Home color={color} /> }}
      />
      <Tabs.Screen
        name="profile"
        options={{ title: 'Profilo', tabBarIcon: ({ color }) => <User color={color} /> }}
      />
      <Tabs.Screen
        name="settings"
        options={{ title: 'Impostazioni', tabBarIcon: ({ color }) => <Settings color={color} /> }}
      />
    </Tabs>
  )
}
```

## AUTH — Supabase + SecureStore

```bash
npm install @supabase/supabase-js expo-secure-store
```

`lib/supabase.ts`:
```ts
import 'react-native-url-polyfill/auto'
import { createClient } from '@supabase/supabase-js'
import * as SecureStore from 'expo-secure-store'

const secureStoreAdapter = {
  getItem: (key: string) => SecureStore.getItemAsync(key),
  setItem: (key: string, value: string) => SecureStore.setItemAsync(key, value),
  removeItem: (key: string) => SecureStore.deleteItemAsync(key),
}

export const supabase = createClient(
  process.env.EXPO_PUBLIC_SUPABASE_URL!,
  process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!,
  {
    auth: {
      storage: secureStoreAdapter,
      autoRefreshToken: true,
      persistSession: true,
      detectSessionInUrl: false, // OBBLIGATORIO su mobile
    },
  }
)
```

`lib/auth.tsx`:
```tsx
import { createContext, useContext, useEffect, useState } from 'react'
import { supabase } from './supabase'
import { Session } from '@supabase/supabase-js'
import { router } from 'expo-router'

const AuthContext = createContext<{ session: Session | null; loading: boolean }>({
  session: null,
  loading: true,
})

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [session, setSession] = useState<Session | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    supabase.auth.getSession().then(({ data: { session } }) => {
      setSession(session)
      setLoading(false)
      router.replace(session ? '/(tabs)' : '/(auth)/login')
    })

    const { data: { subscription } } = supabase.auth.onAuthStateChange((_event, session) => {
      setSession(session)
      router.replace(session ? '/(tabs)' : '/(auth)/login')
    })

    return () => subscription.unsubscribe()
  }, [])

  return <AuthContext.Provider value={{ session, loading }}>{children}</AuthContext.Provider>
}

export const useAuth = () => useContext(AuthContext)
```

## PUSH NOTIFICATIONS

```bash
npm install expo-notifications expo-device
```

`lib/notifications.ts`:
```ts
import * as Notifications from 'expo-notifications'
import * as Device from 'expo-device'
import { Platform } from 'react-native'
import { supabase } from './supabase'

Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: false,
  }),
})

export async function registerForPushNotifications() {
  if (!Device.isDevice) return null // non funziona su simulator

  const { status: existing } = await Notifications.getPermissionsAsync()
  let finalStatus = existing
  if (existing !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync()
    finalStatus = status
  }
  if (finalStatus !== 'granted') return null

  const token = (await Notifications.getExpoPushTokenAsync({
    projectId: process.env.EXPO_PUBLIC_EAS_PROJECT_ID,
  })).data

  if (Platform.OS === 'android') {
    await Notifications.setNotificationChannelAsync('default', {
      name: 'default',
      importance: Notifications.AndroidImportance.MAX,
      vibrationPattern: [0, 250, 250, 250],
    })
  }

  // Salva token nel backend
  const { data: { user } } = await supabase.auth.getUser()
  if (user) {
    await supabase.from('push_tokens').upsert({
      user_id: user.id,
      token,
      platform: Platform.OS,
    })
  }

  return token
}
```

## EAS BUILD — Config

```bash
npm install -g eas-cli
eas login
eas build:configure
```

`eas.json`:
```json
{
  "cli": { "version": ">= 5.0.0" },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "ios": { "simulator": true }
    },
    "preview": {
      "distribution": "internal",
      "channel": "preview",
      "ios": { "resourceClass": "m-medium" }
    },
    "production": {
      "channel": "production",
      "autoIncrement": true,
      "ios": { "resourceClass": "m-medium" },
      "android": { "buildType": "app-bundle" }
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "your@email.com",
        "ascAppId": "1234567890",
        "appleTeamId": "XXXXXXXXXX"
      },
      "android": {
        "serviceAccountKeyPath": "./secrets/play-key.json",
        "track": "production"
      }
    }
  }
}
```

Build:
```bash
eas build --profile production --platform all
```

Submit:
```bash
eas submit --profile production --platform ios
eas submit --profile production --platform android
```

## OTA UPDATES

```bash
# Pubblica update JS/assets senza nuova submission
eas update --channel production --message "fix bug login"
```

**Quando usare OTA:**
- Fix JS/TS
- Aggiornamenti asset (immagini, testi)
- Cambi config client-side

**Quando NON usare OTA (serve nuova build+submission):**
- Aggiunta libreria con codice nativo
- Cambio permessi in `app.json`
- Upgrade Expo SDK
- Cambio icona/splash

## iOS SUBMISSION — Asset e metadati

### Asset obbligatori

- Icona: 1024×1024 PNG (no trasparenza, no angoli arrotondati)
- Screenshot: almeno 3 per ciascuno di: 6.7", 6.5", 5.5" iPhone
- (Opzionale) iPad screenshot per 12.9"

### App Store Connect metadati

- Nome: max 30 char
- Sottotitolo: max 30 char
- Descrizione: max 4000 char
- Keywords: max 100 char, separate da virgola
- Privacy policy URL **obbligatorio**
- Support URL
- Categorie primaria + secondaria
- Age rating
- App Review Info: account demo + istruzioni

### Motivi rejection comuni

1. **Guideline 4.2** — App troppo minimalista o "web wrapper"
2. **Guideline 5.1.1** — Chiedi permessi senza spiegarne uso
3. **Guideline 2.3** — Screenshot non corrispondono all'app
4. **Guideline 3.1.1** — Uso in-app purchase vs link esterno
5. **Account demo non funzionante** per review team
6. **Privacy policy mancante** o non accessibile

## ANDROID SUBMISSION — Play Store

### Asset obbligatori

- Icona: 512×512 PNG
- Feature graphic: 1024×500 PNG
- Screenshot: min 2, max 8 per factor (telefono, tablet)

### Play Console requisiti

- Privacy policy URL
- **Data Safety** form completato (obbligatorio dal 2022)
- Target API level aggiornato
- App Bundle `.aab` (no più `.apk`)
- Rating IARC

### Data Safety checklist

- Quali dati raccogli? (email, nome, location, ecc.)
- Come sono usati? (funzionalità app, analytics, ads)
- Sono condivisi con terze parti?
- Sono cifrati in transito?
- L'utente può chiederne cancellazione?

## CRASH MONITORING — Sentry

```bash
npm install @sentry/react-native
npx sentry-wizard -i reactNative
```

```ts
// App.tsx
import * as Sentry from '@sentry/react-native'

Sentry.init({
  dsn: process.env.EXPO_PUBLIC_SENTRY_DSN,
  enableAutoSessionTracking: true,
  tracesSampleRate: 0.2,
})

export default Sentry.wrap(App)
```

## CHECKLIST PRE-SUBMISSION

- [ ] Testato su dispositivo fisico iOS (2 modelli almeno)
- [ ] Testato su dispositivo fisico Android (2 modelli almeno)
- [ ] Tutte le schermate accessibili funzionano
- [ ] Login/logout funziona
- [ ] Deep link funzionano (se configurati)
- [ ] Push notification ricevute
- [ ] Orientamento schermo corretto
- [ ] Tastiera non copre input critici
- [ ] Dark mode supportato (o disabilitato esplicitamente)
- [ ] Nessun log di debug in produzione
- [ ] Privacy policy online e accessibile
- [ ] Account demo testato dal team
- [ ] Asset store pronti e caricati
- [ ] Version + build number incrementati
- [ ] Changelog chiaro per reviewer
