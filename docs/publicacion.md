# Publicar en GitHub

Crear un repositorio vacío llamado cowrie-wazuh-lab en la cuenta elegida. No inicializar README ni licencia si se va a subir este proyecto como primer commit.

Extraer el ZIP y ejecutar desde la carpeta cowrie-wazuh-lab con Git instalado:

```bash
git init -b main
git add .
git diff --cached --stat
git commit -m "Documentar laboratorio Cowrie y Wazuh validado"
git remote add origin https://github.com/USUARIO/cowrie-wazuh-lab.git
git push -u origin main
```

Sustituir USUARIO por la cuenta correcta. Autenticarse mediante el gestor de credenciales; nunca pegar un token en archivos o comandos compartidos. Este proyecto no incluye licencia: decidirla antes de autorizar reutilización pública del código propio. Cowrie y Wazuh conservan sus respectivas licencias.
