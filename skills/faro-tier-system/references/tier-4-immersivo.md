# Tier 4 — IMMERSIVO

Include tutto il Tier 1-2-3 più:

## Quando usare
Luxury brand, architettura/immobiliare premium, configuratori prodotto,
showroom virtuali, esperienze artistiche/museali.

## Stack aggiuntivo
- Three.js + React Three Fiber (@react-three/fiber)
- @react-three/drei (helper per R3F)
- Modelli 3D: formato .glb (compresso)

## Dipendenze npm
```json
{
  "three": "^0.160",
  "@react-three/fiber": "^8",
  "@react-three/drei": "^9",
  "gsap": "^3.12",
  "@studio-freight/lenis": "^1.0"
}
```

## Componenti esclusivi Tier 4

### Scene3D
```tsx
// Canvas R3F con:
// - Environment (lighting preset o HDRI custom)
// - OrbitControls o scroll-driven camera
// - Responsive: dpr={[1, 2]} per performance
// - Fallback 2D obbligatorio per dispositivi non supportati
```

### Model3D
```tsx
// Carica e mostra modello .glb
// useGLTF con preload
// Animazione: rotazione lenta, o scroll-driven
// Placeholder skeleton durante caricamento
```

### LoadingScreen
```tsx
// Schermata di caricamento con progress bar
// useProgress da @react-three/drei
// Transizione smooth da loading → contenuto
// Deve sparire in max 5 secondi (anche se non completo)
```

### ProductConfigurator
```tsx
// Per configuratori: colore, materiale, varianti
// Stato React per opzioni → aggiorna materiali Three.js
// Pannello laterale con opzioni
// Screenshot/export opzionale
```

### ShowroomVirtual
```tsx
// Ambiente 3D navigabile
// Camera path predefinito o libero
// Hotspot cliccabili per info
// Transition tra stanze
```

## Fallback 2D
OBBLIGATORIO su ogni componente 3D:
```tsx
<Suspense fallback={<Fallback2D />}>
  <Scene3D />
</Suspense>
```
Il fallback deve essere un'immagine statica o un'illustrazione, MAI uno schermo vuoto.

## Performance Tier 4
- Modelli: max 5MB per modello .glb (usare gltf-transform per comprimere)
- Texture: max 2048x2048, formato WebP o KTX2
- dpr={[1, 2]} su Canvas → limita pixel ratio
- Lazy load della scena 3D (dynamic import del Canvas)
- Target Lighthouse: ≥60 performance (3D è pesante), ≥90 a11y

## CSP (Content Security Policy)
Se il progetto usa CSP headers, aggiungere:
- `worker-src 'self' blob:` per Three.js worker
- `script-src 'unsafe-eval'` se serve (Three.js shader compilation)

## Responsive Tier 4
- Mobile: scena 3D semplificata (meno poligoni, no SSAO, no shadow)
- Tablet: qualità media
- Desktop: qualità piena
- `prefers-reduced-motion`: camera statica, no animazione rotazione

## Stile riferimento
- petralithe.com — architettura 3D minimale
- oryzo.ai — interfaccia con elementi 3D integrati
- Apple Vision Pro site — 3D con scroll-driven

## Regole assolute
- Fallback 2D su OGNI componente 3D
- LoadingScreen con progress su OGNI scena
- Max 5s per il caricamento iniziale
- Test su dispositivo mobile REALE (non solo responsive DevTools)
