# Blueprint — Mobile (Expo / React Native)

## Struttura
```
src/
├── app/                            # Expo Router (file-based)
│   ├── (tabs)/
│   │   ├── _layout.tsx             # Tab bar
│   │   ├── index.tsx               # Home
│   │   ├── [feature].tsx
│   │   └── profile.tsx
│   ├── (auth)/
│   │   ├── login.tsx
│   │   └── register.tsx
│   ├── (modals)/
│   │   └── [modal].tsx
│   └── _layout.tsx                 # Root layout
├── components/
│   ├── ui/                         # Componenti custom (no shadcn)
│   └── [feature]/
├── lib/
│   ├── supabase.ts
│   └── storage.ts                  # SecureStore per token
├── hooks/
├── types/
└── assets/
```

## Sicurezza specifica
- Token in SecureStore (MAI AsyncStorage per token)
- Certificate pinning (opzionale)
- Biometric auth (opzionale)
- Deep link validation

## UX specifica
- Tab bar per navigazione principale
- Stack navigation per drill-down
- Pull-to-refresh su liste
- Haptic feedback su azioni
- Skeleton su ogni lista/card
