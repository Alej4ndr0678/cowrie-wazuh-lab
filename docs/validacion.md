# Validación

## Evidencia observada

- Ping Cowrie → Wazuh: 4 enviados, 4 recibidos, 0 % pérdida.
- Cowrie escuchando en 192.168.50.20:2222.
- Agente 001 cowrie-honeypot activo, versión 4.14.8.
- Logcollector analizando `/home/cowrie/honeypot/var/log/cowrie/cowrie.json`.
- analysisd -t sin mensajes de error.
- wazuh-logtest: JSON reconocido, regla 100104, nivel 5.
- Tres alertas reales en alerts.json y en Threat Hunting.

## Repetir la prueba

Desde Windows:

```powershell
ssh -p 2222 root@192.168.50.20
```

Usar una contraseña ficticia exclusiva para la prueba. Cowrie registra contraseñas de los intentos; no incluirlas en capturas ni subir los registros completos.

Si el honeypot acepta el acceso, ejecutar whoami, uname -a y exit. En el manager:

```bash
sudo grep '"100104"' /var/ossec/logs/alerts/alerts.json | tail -n 3
```

En Threat Hunting → Events seleccionar Last 15 minutes y DQL:

```text
agent.name:"cowrie-honeypot" AND rule.id:"100104"
```

Un rango con inicio y fin idénticos produjo cero resultados; corregir el selector antes de diagnosticar la integración.

## Resultados del 10/10/2026 (America/Guayaquil)

| Hora en panel | Comando | Regla | Nivel |
|---|---|---|---|
| 23:04:40.848 | whoami | 100104 | 5 |
| 23:04:40.848 | uname -a | 100104 | 5 |
| 23:04:41.798 | exit | 100104 | 5 |

No se adjuntan capturas: las imágenes enviadas no estuvieron accesibles en el entorno de preparación. Los datos anteriores se basan en salidas de terminal y texto del panel compartidos por el autor.

Las reglas de conexión, acceso fallido y acceso aceptado se verificaron posteriormente; ver la validación ampliada. Un nivel 5 es una clasificación local, no evidencia de compromiso del Ubuntu real.

## Arranque automático comprobado (10/10/2026)

Servicio disponible en `configs/cowrie.service`. Instalar en /etc/systemd/system/cowrie.service, detener primero la instancia manual y ejecutar daemon-reload y enable --now cowrie. El servicio usa twistd en primer plano con el usuario cowrie y el directorio de estado del honeypot.

Después de sudo reboot y reconectar por SSH se obtuvo:

```text
enabled
active
active
LISTEN 192.168.50.20:2222 (twistd)
```

Las dos líneas active corresponden a cowrie y wazuh-agent. El puerto del honeypot se verificó después del reinicio; la prueba posterior de eventos hacia Wazuh se completó, como se documenta abajo. En Cowrie 3.1.1 del laboratorio, `cowrie start -n` produjo un error de argumento; se sustituyó por la ejecución directa de twistd que se probó satisfactoriamente.

## Desconexión de Default Switch y prueba posterior (10/10/2026)

En PowerShell administrador se ejecutó:

```powershell
Disconnect-VMNetworkAdapter -VMName "cowrie-honeypot" -Name "Network Adapter"
```

La tarjeta Lab permaneció conectada a Lab-Honeypot. Ubuntu mostró eth0 DOWN, eth1 UP con 192.168.50.20/24 y una única ruta IPv4 a 192.168.50.0/24, sin ruta default. Cowrie y wazuh-agent siguieron activos. Ping a Wazuh: 4/4 respuestas, 0 % pérdida.

Después se abrió otra sesión al puerto 2222 desde Windows. Threat Hunting mostró dos alertas reales de nivel 5 y regla 100104:

| Hora local | Comando |
|---|---|
| 23:57:17.280 | echo prueba-red-laboratorio |
| 23:57:17.321 | exit |

La prueba confirma recepción de eventos después de desconectar Default Switch. No verifica por sí sola aislamiento total: las rutas IPv6 fueron inspeccionadas posteriormente; falta comprobar reenvío del host. Windows y Wazuh siguen accesibles en el segmento interno por diseño. Para volver a descargar paquetes se requiere reconectar explícitamente el adaptador y volver a desconectarlo antes de las pruebas.

## Autenticación y configuración comprobadas (11/10/2026)

| Hora local | Evento | Regla | Nivel |
|---|---|---|---|
| 00:11:07.412 | Conexión desde Windows | 100101 | 3 |
| 00:11:09.413 | Acceso fallido de root | 100102 | 5 |
| 00:21:27.509 | Nueva conexión desde Windows | 100101 | 3 |
| 00:21:35.508 | Acceso aceptado de root | 100103 | 5 |

El panel mostró ocho conexiones en la ventana consultada, una originada en la propia VM Cowrie. No se considera una serie experimental de ocho repeticiones homogéneas.

Se comprobó backend shell, hostname svr04, SSH limitado a 192.168.50.20:2222 y Telnet desactivado. Se cambió forwarding de true a false, se reinició el servicio y se verificó active (running) con el puerto escuchando. Las rutas IPv6 solo mostraron fe80::/64, sin ruta por defecto.

Se creó etc/userdb.txt con una política explícita (ver configs/userdb.txt.example). Los primeros intentos de aceptación fallaron porque la contraseña enviada difería del valor configurado, según el registro; después se usó una credencial ficticia alfanumérica y el acceso fue aceptado. Los rechazos confirman clasificación de eventos, no una tasa de falsos positivos ni efectividad general. No se publican contraseñas reales ni registros completos de autenticación.
