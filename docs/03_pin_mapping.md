# 03 – Pin mapping (resumen)
- 4093: entradas Schmitt; usar 1 puerta para SÍ y 1 para NO.
- 4011/4043: latch RS, S=SI_clean, R=RESET ∨ FIN20 (∨ Alarma).
- 555 astable: 1 Hz. 555 mono: ~3 s (buzzer).
- 4026×2: unidades/decenas; FIN20 = (Dec=2) ∧ (Uni=0).
- 4017: Q3 dispara mono y resetea 4017.
