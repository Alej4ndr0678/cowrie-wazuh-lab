# Plan de evaluación experimental (pendiente)

| Escenario | Evento esperado | Regla |
|---|---|---|
| Abrir sesión SSH | cowrie.session.connect | 100101 |
| Contraseña rechazada por la política configurada | cowrie.login.failed | 100102 |
| Acceso aceptado al shell simulado | cowrie.login.success | 100103 |
| Comando en shell | cowrie.command.input | 100104 |

Para cada escenario realizar 10 repeticiones controladas desde el host o una VM de pruebas autorizada. Guardar un identificador por repetición, hora de generación, llegada al manager y aparición en panel. Diferenciar demora de generación de alerta y demora de indexación. Sincronizar relojes antes de medir.

Métricas: eventos esperados frente a alertas recibidas; porcentaje detectado; latencia mediana y percentil 95; CPU/RAM durante las pruebas. No extrapolar tres comandos a una medición de efectividad general ni llamar fuerza bruta a un intento individual. No hay métricas experimentales completas todavía.
