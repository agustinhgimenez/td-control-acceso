# Control de Acceso (CMOS, sin MCU)

> **Proyecto TDI** — Sistema con **RS** (memoria), **temporizador ~20 s** visible (4026×2) y **alarma por 3× NO** (4017 + 555 mono). Sin microcontrolador.

---

## 🔗 Enlaces rápidos
- **Trello (tablero):** https://trello.com/b/4g7cJfvU/
- **Google Drive (carpeta pública):** https://drive.google.com/drive/folders/1jVRE6xx5ScEsgHIhIT_Imk9TTcd5e1bN?usp=sharing
- **Video demo (final):** _(agregar enlace cuando lo graben)_

> Si el repo es público, verifiquen permisos de Trello/Docs (o compartan solo lectura).

---

## 📁 Estructura del repositorio
```
tdi-control-acceso-cmos/
├─ proteus/
│  ├─ acceso_rs/                 # RS + SÍ + RESET + LEDs
│  │  ├─ acceso_rs.pdsprj
│  │  └─ acceso_rs.dsn (según versión)
│  ├─ temporizador_20s/          # 1 Hz + 4026×2 + FIN20
│  │  ├─ temporizador_20s.pdsprj
│  │  └─ temporizador_20s.dsn
│  ├─ alarma_3no/                # 4017 + 555 mono + buzzer
│  │  ├─ alarma_3no.pdsprj
│  │  └─ alarma_3no.dsn
│  └─ integrado/
│     └─ integrado.pdsprj        # proyecto final listo para correr
├─ docs/
│  ├─ 01_diseno_bloques.md       # diagramas, señales, tablas RS/4017/FIN20
│  ├─ 02_calculos_RC.md          # 555 astable (1 Hz) y mono (3 s)
│  ├─ 03_pin_mapping.md          # pines usados (4093/4011/4043/4026/4017/555)
│  ├─ 04_pruebas_plan.md         # matriz de pruebas y criterios de aceptación
│  ├─ 05_bom_lista_materiales.md # proveedores/ETAs
│  └─ 07_informe_final.md        # resumen/export del legajo
├─ media/
│  ├─ capturas/                  # PNG/JPG desde simulador
│  └─ videos/                    # MP4/MOV demo
├─ hardware/                     # (opcional) fotos protoboard / PCB / Gerbers
├─ datasheets/                   # solo extractos útiles (pines/tablas)
├─ .gitattributes                # Git LFS tracking
├─ .gitignore
└─ README.md
```

---

## ▶️ Cómo correr la simulación (Proteus)
1. **Versión:** Proteus 8.x (ISIS).  
2. Abrir `proteus/integrado/integrado.pdsprj`.  
3. Verificar instrumentos virtuales:
   - **Osciloscopio:** `CLK_1Hz`, `CLK_gate`, `SI_clean`, `NO_clean`, `Q`.
   - **Logic Analyzer:** `Q0..Q3` del 4017 y BCD hacia 4026 (opcional).
4. **Run ▶** y probar:
   - Pulsar **SÍ** → LED verde ON; **00→20** en display.
   - Al llegar a **20** → **FIN20** resetea a 00 y **LED rojo** ON.
   - Pulsar **NO** ×3 → **buzzer ~3 s**.

> **Puntos de test**: `SI_clean`, `NO_clean`, `Q/Q̄`, `CLK_1Hz`, `CLK_gate`, `Q3_4017`, `FIN20`.

---

## 🧩 Arquitectura (bloques)
- **Antirrebote (CD4093):** monoestable 10–20 ms → `SI_clean`, `NO_clean`.
- **RS (4011/4043):** `S=SI_clean`, `R=RESET ∨ FIN20 (∨ Alarma)`. `Q` gatea 1 Hz.
- **1 Hz (555 astable):** `f ≈ 1.44 / ((RA + 2·RB) · C)` → reloj de conteo.
- **Contador 00→20 (4026×2):** `FIN20 = (Dec=2) ∧ (Uni=0)`.
- **Alarma 3× NO (4017 + 555 mono):** `t ≈ 1.1 · R · C` → buzzer con **2N2222**.

Diagramas de **bloques** y **flujo**: ver `docs/01_diseno_bloques.md`.

---

## 🧪 Plan de pruebas (resumen)
| # | Caso | Acción | Esperado |
|---|-----|--------|----------|
| 1 | SÍ abre | Pulsar **SÍ** | `Q=1`, Verde ON, inicia conteo |
| 2 | 00→20 | Esperar a 20 | `FIN20` → reset 4026 y `R` del RS |
| 3 | NO×3 | Pulsar **NO** tres veces | 4017 en **Q3**, buzzer ~3 s |
| 4 | RESET | Pulsar **RESET** | Estado inicial |
| 5 | Rebotes | Mantener pulsador | 1 toque = 1 pulso |

Matriz completa: `docs/04_pruebas_plan.md`.

---

## 🔧 Cálculos y valores típicos
- **Astable ~1 Hz (555):** p. ej., **RA=8.2 kΩ**, **RB=8.2 kΩ**, **C=47 µF** → ≈1.0 s.  
- **Monoestable ~3 s (555):** p. ej., **R=270 kΩ**, **C=10 µF** → ≈3.0 s.  
- **Antirrebote (4093):** **R=100 kΩ**, **C=100 nF** → ≈10 ms.

> Ajustar según tolerancias y registrar en `docs/02_calculos_RC.md`.

---

## 🧾 BOM (resumen)
- **Lógicos/Timers:** CD4093 ×1–2, CD4011 **o** CD4043/4044 ×1, NE555 ×2, CD4017 ×1, CD4026 ×2, CD4081 ×1, CD4049/4069 ×1 (si hace falta).  
- **Displays/LEDs:** 7-seg **cátodo común** ×2 + resistencias 220–330 Ω/segmento; LEDs rojo/verde + resistencias 330–1 kΩ.  
- **Driver/Audio:** 2N2222 + diodo (1N4148/1N4007), buzzer 5–9 V.  
- **Pasivos:** surtido + **100 nF por CI**.

Detalle y proveedores: `docs/05_bom_lista_materiales.md`.

---

## ⚡ Compatibilidad y buenas prácticas
- **Familia:** CMOS 4000 a 5–9 V (si se mezcla con TTL, usar 74HCT a 5 V).  
- **Entradas no flotantes:** pull-up/pull-down; **salidas** pueden quedar libres.  
- **Desacople:** **100 nF por CI** cerca de Vcc/GND.  
- **Inicio/Reset:** al energizar no debe arrancar solo → usar **reset RC** (power-on).

---

## 🧭 Gestión del proyecto
- Fechas y flujo de trabajo en **Trello** (ver enlace).  
- **Legajo técnico** en Google Drive (se exporta a `docs/07_informe_final.md`).

---

## 🛠️ Setup del repo (Git LFS + ignores)
**.gitattributes** (recomendado para binarios grandes):
```
*.pdsprj filter=lfs diff=lfs merge=lfs -text
*.dsn    filter=lfs diff=lfs merge=lfs -text
*.mp4    filter=lfs diff=lfs merge=lfs -text
*.mov    filter=lfs diff=lfs merge=lfs -text
*.png    filter=lfs diff=lfs merge=lfs -text
*.jpg    filter=lfs diff=lfs merge=lfs -text
```
**.gitignore** (mínimo):
```
.DS_Store
Thumbs.db
/media/raw/
*.bak
*.tmp
```

**Comandos iniciales**
```bash
git init
git lfs install
git lfs track "*.pdsprj" "*.dsn" "*.mp4" "*.mov" "*.png" "*.jpg"
git add .gitattributes .gitignore README.md proteus docs media
git commit -m "feat: estructura inicial + README + LFS"
# crea el repo en GitHub (por ej. tdi-control-acceso-cmos) y luego:
git remote add origin https://github.com/<tu-usuario>/tdi-control-acceso-cmos.git
git push -u origin main
```

---

## 👥 Autores y roles
- **A** — RS + integración; documentación de bloques.  
- **B** — Timer/contador + NO×3; pruebas y compras.  
**Ambos:** integración final, informe y defensa (cada uno expone TODO).

---

## 📜 Licencia
Uso educativo. (Definir si el repo será público o privado).

---

## 🗒️ Changelog
- `2025-09-01` — Versión inicial (estructura + simulación por bloques)
- `2025-09-XX` — Integrado y demo
