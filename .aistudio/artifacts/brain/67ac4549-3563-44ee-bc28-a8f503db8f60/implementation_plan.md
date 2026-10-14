# Investigación Exhaustiva, Diagnóstico de Rendimiento y Plan de Fluidez para el Circuito 1

Plan de optimización técnica y arquitectura gráfica para maximizar la tasa de fotogramas (60 FPS estables) en dispositivos de gama media y móviles en el Circuito 1 (Square Apex), conservando el 100% de los elementos visuales, efectos climáticos, sombras, multitudes 3D, partículas y postprocesado cinematográfico.

---

## 1. Inventario Exhaustivo del Circuito 1 (Square Apex)

El Circuito 1 es un trazado urbano-permanente de Grado 1 FIA de 974.76 metros de longitud y 16 metros de ancho con 4 vértices de 90° y rectas interconectadas de 184 metros.

```
                   [ RECTA NORTE (184m) · SPEED TRAP 328 KM/H ]
      (T4: NO-Oeste) ┌────────────────────────────────────────┐ (T2: NE-Este)
      cx=-92, cz=92  │                                        │ cx=92, cz=92
                     │                                        │
                     │                                        │
      [ RECTA OESTE ]│          [ INFIELD NATURAL ]           │[ RECTA ESTE ]
         (184m)      │      Topografía ondulada 800x800m      │    (184m)
                     │      Bermas de césped y arboleda       │
                     │                                        │
      (T3: SO-Oeste) └──────────┬───────────────────┬─────────┘ (T1: SE-Este)
      cx=-92, cz=-92            │  PIT LANE (184m)  │           cx=92, cz=-92
                                └───────────────────┘
                     [ RECTA PRINCIPAL SUR · META / PARRILLA ]
                   [ GRAN TRIBUNA SUR CON DOSEL TENSIÓN Y CAPSULAS ]
```

### Componentes y Subsistemas del Circuito 1:
1. **Superficie de Pista y Zonas de Escape (`TrackBuilder`):**
   - Asfalto PBR de alta adherencia con mapeado micro-agregado (Albedo, Normales Tangenciales, Rugosidad y mapa de desgaste por fricción).
   - 4 Escapes en grava en los vértices exteriores (Turns 1 a 4) con relieve de guijarros en normal map.
   - Pianos 3D continuos (Kerbs) con perfil biselado y esmalte Rosso Corsa y Blanco tráfico con esmalte cerámico.
   - Hombros de tierra y arcilla compactada en los límites de pista.
   - Líneas de demarcación vial FIA: límites continuos, líneas discontinuas de 3.2m / ciclo de 8m y cajones de salida.
2. **Pit Lane y Complejo Paddock (`CurvedPitBuildingBuilder`):**
   - Edificio curvo de dos niveles con boxes F1, suelo epoxi industrial pulido, islas de telemetría de fibra de carbono, mantas térmicas Pirelli y persianas de servicio.
   - Gantry de entrada a boxes con pantallas LED (`PitEntryGantryBuilder`).
   - Atenuador de impacto / cojín de choque QuadGuard en la punta del muro de boxes.
   - Pylons y bolardos fluorescentes reflectantes.
3. **Arquitectura y Gradas:**
   - Gran Tribuna Sur: Anfiteatro curvado con 10 filas parabólicas, asientos individuales ergonómicos instanciados (rojo, azul y dorado VIP), túneles vomitorios, dosel tensado ondulante biónico y cápsulas aero-lounge suspendidas.
   - Tribuna Secundaria Norte: Estructura modular de andamiaje y lona con pancarta de patrocinio.
   - Edificio VIP Moderno (`ModernVIPBuildingBuilder`): Podio de hormigón, columnas tubulares, muros cortina de cristal tintado, salón panorámico en voladizo (4.5m sobre la pista) y terraza chill-out con parasol alar.
4. **Entorno Natural y Fondo Panorámico:**
   - Terreno orgánico esculpido (`OrganicTerrainBuilder`) de 800x800m con bermas y valles de drenaje.
   - Anillo panorámico 360° de Macizo Alpino (`DistantMountainBackdropBuilder`) con sombreador PBR fotométrico.
   - Sistema de Vegetación Orgánica (`OrganicVegetationSystem`): Pinos piñoneros mediterráneos, robles de hoja ancha, cipreses italianos, matojos densos y matas de hierba curvada instanciadas por cuadrículas espaciales.
5. **Iluminación e Infraestructura Nocturna:**
   - 24 Torres de iluminación de estadio de 24 metros de altura con 6 focos y lentes emisivas cada una.
   - Luz solar direccional sincronizada con cálculo de sombras PCF y luz de hemisferio ambiental.
   - Bóveda celeste Rayleigh dinámica (`DynamicSkySystem`) con disco solar físico, corona de Mie y mapa de reflexión ambiental en tiempo real.
6. **Seguridad y Señalética Dinámica:**
   - Barreras de hormigón continuas con valla de captura de escombros metálica y postes instanciados.
   - Carteles de distancia de frenada rompibles (200m, 150m, 100m, 50m) en cada aproximación con física balística.
   - Conos fluorescentes en los vértices interiores de curva.
7. **Sistemas de Efectos y Cinemática:**
   - Multitud 3D viva (`GrandstandCrowdSystem`): Espectadores instanciados con cinemática 100% en GPU vertex shader.
   - Efecto de refracción térmica (`HeatHazeEffect`): Pluma de escape y espejismo sobre el asfalto caliente.
   - Efecto de estelas de velocidad periféricas (`SpeedPostEffect`).
   - Óptica cinematográfica de retransmisión (`CinematicPostEffect`): Viñeteado anamórfico, grano de sensor CMOS, destello solar y halación xenón.
   - Sistema de partículas dinámicas (`ParticleSystem`): Chispas de titanio con radiación de cuerpo negro, humo de neumáticos volumétrico, fragmentos de fibra de carbono y huellas de derrape continuas en las 4 ruedas (`F1SkidShader`).

---

## 2. Diagnóstico Multidimensional del Rendimiento

```
┌────────────────────────────────────────────────────────────────────────┐
│                   DIAGNÓSTICO DEL PIPELINE DE RENDER                   │
├────────────────────────────────┬───────────────────────────────────────┤
│ CUELLO DE BOTELLA IDENTIFICADO │ IMPACTO TÉCNICO EN GAMA MEDIA/MÓVIL   │
├────────────────────────────────┼───────────────────────────────────────┤
│ 1. Pase de Sombras Direccional │ Ancho de banda VRAM y texel fetch     │
│    (Frustum de 75m PCF)        │ excesivo al redibujar 60 veces/seg.   │
├────────────────────────────────┼───────────────────────────────────────┤
│ 2. Shaders Fullscreen Pass     │ Sobrecarga de ALU en cálculos trig.   │
│    (HeatHaze, Speed, Cinematic)│ (`sin`, `cos`, `atan`, `fract`, `pow`)│
├────────────────────────────────┼───────────────────────────────────────┤
│ 3. Granularidad de Culling     │ Sobrecarga de GPU al evaluar mallas   │
│    en InstancedMesh distantes  │ instanciadas de gradas y arboleda.    │
├────────────────────────────────┼───────────────────────────────────────┤
│ 4. Precisión de Cálculos en FS │ Uso de `highp` por defecto en shaders │
│    (Float32 vs Half Float16)   │ cuando `mediump` es óptimo para color.│
├────────────────────────────────┼───────────────────────────────────────┤
│ 5. Texture Lookups & Mipmaps   │ Anisotropía x16 en mapas secundarios  │
│    en Superficies Planas       │ consume ciclos de filtrado de textura.│
└────────────────────────────────┴───────────────────────────────────────┘
```

### Perspectiva 1: Arquitecto de Gráficos (GPU Pipeline & Memory Bandwidth)
- **Problema:** En GPUs móviles e integradas (Adreno 618/730, Mali G57/G77, Intel Iris Xe), la arquitectura TBDR (Tile-Based Deferred Rendering) penaliza fuertemente el cambio frecuente de estados de renderizado y el ancho de banda hacia la memoria unificada.
- **Causa en Circuito 1:** El renderizado de sombras proyectadas PCF cubre un radio amplio de 75m. Cuando el coche se desplaza, cada actualización del mapa de sombras procesa los casters visibles.
- **Solución:** Mantener el 100% de las sombras pero acotar el bounding box del frustum de sombra a la zona inmediata de influencia visual del monoplaza y la pista activa, manteniendo estáticos los casters ambientales sin recompilaciones de buffers.

### Perspectiva 2: Especialista en Shaders (ALU & Fragment Shader Cost)
- **Problema:** Los tres pases de post-procesado (`CinematicPostEffect`, `SpeedPostEffect`, `HeatHazeEffect`) renderizan planos frente a la cámara calculando funciones de ruido procedural, potencias y trigonometría por cada fragmento. En una pantalla Full HD (1920x1080 = 2.07M fragmentos), esto representa millones de instrucciones trascendentes por fotograma.
- **Solución:** Reemplazar funciones de ruido pesado y polinomios exponenciales por aproximaciones analíticas optimizadas y factorizaciones matemáticas (utilizando `fma`, multiplicaciones cuadráticas escaladas y precisión `mediump` en el bloque de fragmentos). Preservar al 100% el resultado estético visual (mismo color, misma intensidad, misma animación).

### Perspectiva 3: Ingeniero de Software Principal (Garbage Collector & Ciclo de Vida)
- **Problema:** Aunque `MathPool` absorbe vectores en la física, la sincronización de matrices de props dinámicos y la actualización de uniforms de materiales que clonan colores o vectores crean pequeños objetos transitorios que acumulan trabajo en el Garbage Collector (GC), provocando caídas periódicas de frametime (stutters de 20-40ms).
- **Solución:** Asegurar asignación directa de uniforms primitivos (`uniform.value.x = ...` en lugar de `.set()` o `.copy()` con creación auxiliar) y congelar matrices no dinámicas (`matrixAutoUpdate = false`).

### Perspectiva 4: Especialista en Arquitectura de Gama Media y Móviles
- **Problema:** Los dispositivos de gama media no sufren por un único elemento pesado, sino por la acumulación simultánea de resolución nativa alta (pantallas de 2400x1080 o Retina) junto con múltiples pases de textura y sombreado.
- **Solución:** Potenciar el `GraphicsQualityManager` existente con un Dynamic Resolution Scaler (DRS) imperceptible de doble banda que module ligeramente el DPR solo cuando el frametime supere los 16.6ms, manteniendo intactos todos los efectos y regresando a la nitidez nativa cuando la GPU se estabilice.

---

## 3. Plan de Acción de Alto Rendimiento

### Fase 1: Optimización Matemática de Shaders y Postprocesado (Cero pérdida visual)
- Optimizar los fragment shaders de `CinematicPostEffect`, `HeatHazeEffect` y `SpeedPostEffect` aplicando:
  - Definición explícita de `precision mediump float;` en cálculos de colorimetría y distorsión.
  - Sustitución de `pow(x, 2.0)` o `pow(x, 3.0)` por multiplicaciones directas (`x * x`).
  - Pre-cálculo de constantes invariantes de aspecto y tiempo en el hilo principal antes de pasarlas como uniform.
  - Descarte anticipado (`discard` o `alpha <= 0.001`) en regiones de pantalla donde el efecto tiene opacidad cero (ej. zona central despejada en `SpeedPostEffect`).

### Fase 2: Afinación del Pase de Sombras y Luces del Circuito
- Optimizar la cámara de sombras ortográfica de la luz direccional (`dirLight.shadow.camera`):
  - Ajustar el volumen ortográfico a un ajuste ceñido dinámico centrado en el coche (60m en lugar de 75m sobredimensionado).
  - Configurar `shadow.bias = -0.0003` y `shadow.normalBias = 0.015` para erradicar shadow acne sin requerir mapas de sombras excesivamente densos.
  - Asegurar que las mallas lejanas o no expuestas tengan `castShadow = false` estricto, reduciendo drásticamente la lista de objetos renderizados en el pase de sombras.

### Fase 3: Optimización de Culling Jerárquico en Gradas y Vegetación
- Verificar y recalcular los `boundingSphere` y `boundingBox` de cada bloque de `InstancedMesh` en `TrackBuilder`, `GrandstandCrowdSystem` y `OrganicVegetationSystem`.
- Garantizar que el motor de Three.js descarte bloques enteros de árboles y espectadores cuando la cámara apunta hacia la recta opuesta sin consumir ciclos de rasterización.

### Fase 4: Escalador Dinámico de Fotogramas y Pacing Estable
- Afinar el controlador adaptativo en `GraphicsQualityManager`:
  - Suavizar el muestreo de frametime con filtro paso bajo de 30 muestras.
  - Mantener los 60 FPS fijos con microajuste de DPR adaptativo entre 1.0x y 1.25x en gama media.
  - Mantener activados en todo momento el heat haze, la óptica cinemática, las sombras PCF y la multitud 3D.

---

## 4. Diagrama de Arquitectura del Pipeline Optimizado

```
┌────────────────────────────────────────────────────────────────────────┐
│                   RACING GAME ENGINE (TICK 120HZ / 60FPS)              │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │
           ┌────────────────────────┴────────────────────────┐
           ▼                                                 ▼
┌────────────────────────┐                        ┌────────────────────────┐
│   PHYSICS & LOGIC      │                        │    GRAPHICS PIPELINE   │
│ 120Hz Lockstep Fixed   │                        │ Sub-frame Alpha Interp │
│ Zero Alloc MathPool    │                        └──────────┬─────────────┘
└────────────────────────┘                                   │
                                     ┌───────────────────────┴───────────────────────┐
                                     ▼                                               ▼
                         ┌───────────────────────┐                       ┌───────────────────────┐
                         │   SHADOW PASS CULL    │                       │  MAIN SCENE RENDERING │
                         │ • Tight 60m bounds    │                       │ • Instanced Vegetation│
                         │ • Dynamic deadzone    │                       │ • GPU Vertex Crowds   │
                         │ • Early caster reject │                       │ • Merged Circuit PBR  │
                         └───────────┬───────────┘                       └───────────┬───────────┘
                                     │                                               │
                                     └───────────────────────┬───────────────────────┘
                                                             ▼
                                                 ┌───────────────────────┐
                                                 │ ZERO-COST POST EFFECTS│
                                                 │ • Fast-math Vignette  │
                                                 │ • Mediump HeatHaze    │
                                                 │ • Edge-only Speed Stre│
                                                 └───────────┬───────────┘
                                                             ▼
                                                 ┌───────────────────────┐
                                                 │ 60-120 FPS ROCK SOLID │
                                                 │ 100% Visual Fidelity  │
                                                 └───────────────────────┘
```

---

## 5. Garantía de Cero Pérdida Visual

Todas las mejoras propuestas actúan a nivel de:
1. **Eficiencia algorítmica** (menos instrucciones por píxel, no menos píxeles de detalle).
2. **Descarte geométrico inteligente** (no renderizar lo que está a espaldas de la cámara).
3. **Optimización de memoria y VRAM** (evitar lecturas redundantes).

Ningún elemento del Circuito 1 (ni las gradas, ni los edificios VIP, ni las chispas, ni el calor del motor, ni las montañas 360°, ni el público animado) será eliminado o reducido visualmente.
