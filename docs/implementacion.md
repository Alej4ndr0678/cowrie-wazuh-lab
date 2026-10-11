# Implementación del laboratorio

## Máquinas y red

Crear dos VM generación 2 con arranque seguro Microsoft UEFI Certificate Authority. Usar discos nuevos e instalar Ubuntu Server desde la misma ISO. Desactivar memoria dinámica: se observó que Ubuntu veía aproximadamente 520 MiB en Cowrie y 606 MiB en Wazuh antes de fijar la RAM.

Mantener Default Switch durante la descarga de paquetes y añadir otra tarjeta llamada Lab al switch interno Lab-Honeypot. Configurar Windows 192.0.2.1/24. Los ejemplos Netplan asumen que la tarjeta Lab es eth1: comprobar `ip -br link` antes de aplicarlos. No configurar gateway ni DNS en eth1.

Guardar el YAML como `/etc/netplan/60-lab-honeypot.yaml`, permisos 600. Ejecutar `sudo netplan generate` y `sudo netplan try` con la consola Hyper-V disponible. Confirmar si funciona; DHCP de eth0 puede cambiar su dirección durante la aplicación. Administrar después mediante las IP fijas.

## Wazuh

Instalar los componentes centrales con el asistente oficial compatible con la versión elegida. En este laboratorio quedó Wazuh 4.14.8. Verificar:

```bash
sudo systemctl is-active wazuh-manager wazuh-indexer wazuh-dashboard filebeat
```

Los cuatro devolvieron active. Acceder al panel HTTPS en 192.0.2.10. Guardar las credenciales generadas fuera del repositorio.

## Cowrie

En Ubuntu de Cowrie, preparar dependencias:

```bash
sudo apt update
sudo apt install -y openssh-server curl git python3-venv python3-dev build-essential libssl-dev libffi-dev
sudo systemctl enable --now ssh
sudo adduser --disabled-password --gecos "" cowrie
sudo -iu cowrie
mkdir -p ~/honeypot
cd ~/honeypot
python3 -m venv cowrie-env
source cowrie-env/bin/activate
python -m pip install --upgrade pip
python -m pip install 'cowrie==3.1.1'
cowrie init
```

Fusionar `configs/cowrie.cfg.example` con el archivo generado. No ejecutar Cowrie como root. Iniciar con `cowrie start`, verificar con `cowrie status` y `ss -lnt`. Se comprobó escucha en 192.0.2.20:2222. El hostname simulado observado fue svr04; el ejemplo propone servidor-lab, pendiente de verificar en el archivo real.

## Agente y recolección

Desde Deploy new agent, seleccionar DEB amd64, manager 192.0.2.10 y nombre cowrie-honeypot. Ejecutar el comando generado como administrador de Ubuntu. La versión comprobada fue 4.14.8. Habilitar e iniciar wazuh-agent.

Añadir `configs/wazuh-agent-localfile.xml` dentro de ossec_config en `/var/ossec/etc/ossec.conf` de Cowrie. Respaldar antes y reiniciar el agente. Confirmar que ossec.log muestra Analyzing file para cowrie.json.

## Reglas

En el manager, respaldar `/var/ossec/etc/rules/local_rules.xml`. Añadir el grupo de `rules/cowrie_rules.xml` después del grupo existente. Comprobar que los IDs no estén ocupados por otras reglas locales. Ejecutar `sudo /var/ossec/bin/wazuh-analysisd -t` y probar mediante wazuh-logtest antes de reiniciar wazuh-manager.
