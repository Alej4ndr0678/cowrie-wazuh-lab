# Laboratorio Cowrie + Wazuh

Laboratorio de detección y análisis de interacciones SSH con Cowrie, integrado con Wazuh mediante un agente y reglas personalizadas. Implementado por Alejandro Venegas en Hyper-V para un proyecto técnico de titulación y portafolio.

## Estado comprobado

El 10 de octubre de 2026 se verificó el recorrido completo: Windows → Cowrie → archivo JSON → agente → Wazuh Manager → dashboard. El panel mostró tres alertas de nivel 5, regla 100104, correspondientes a `whoami`, `uname -a` y `exit`.

| Componente | Configuración |
|---|---|
| Host | Windows, Intel i5-12450H, 32 GB RAM |
| Wazuh | Ubuntu Server 24.04, Wazuh 4.14.8, 4 vCPU, 8 GB RAM fija, disco 60 GB |
| Cowrie | Ubuntu Server 24.04.5, Cowrie 3.1.1, 2 vCPU, 2 GB RAM fija, disco 25 GB |
| Red | Switch interno Hyper-V Lab-Honeypot, 192.168.50.0/24 |
| Windows | 192.168.50.1 |
| Wazuh | 192.168.50.10 |
| Cowrie | 192.168.50.20; administración SSH 22, honeypot SSH 2222 |

## Arquitectura

```mermaid
flowchart TD
  H["Windows: pruebas SSH"] -->|"2222/TCP"| C["Cowrie: shell simulado"]
  C --> J["cowrie.json"]
  J --> A["Agente Wazuh"]
  A --> M["Wazuh Manager: reglas JSON"]
  M --> I["Indexer"]
  I --> D["Dashboard: alertas"]
```

## Archivos

- [Implementación](docs/implementacion.md)
- [Validación y resultados](docs/validacion.md)
- [Evaluación experimental](docs/evaluacion.md)
- [Pendientes](docs/pendientes.md)
- `configs/`: ejemplos mínimos, no copias completas de los servidores.
- `rules/cowrie_rules.xml`: reglas personalizadas.
- `examples/`: eventos sintéticos sin contraseñas ni identificadores de sesiones reales.

## Límites actuales

La integración está verificada. El arranque automático de Cowrie y la retirada de su tarjeta Default Switch todavía están pendientes. Escuchar solo en la IP del laboratorio no bloquea la salida a Internet. El switch interno incluye acceso al host Windows; no implica aislamiento total del host. Las pruebas realizadas son controladas y no representan ataques recibidos desde Internet.

## Fuentes

- https://docs.cowrie.org/en/stable/INSTALL.html
- https://documentation.wazuh.com/current/quickstart.html
- https://documentation.wazuh.com/current/user-manual/reference/ossec-conf/localfile.html
- https://documentation.wazuh.com/current/user-manual/ruleset/rules/custom.html
- https://documentation.wazuh.com/current/user-manual/reference/tools/wazuh-logtest.html

Versiones documentadas corresponden al laboratorio; consultar compatibilidad antes de reproducir con versiones nuevas.
