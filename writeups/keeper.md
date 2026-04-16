# HTB: Keeper

> Escaneo de vulnerabilidades y explotación de servicios expuestos.

## Reconnaissance

Iniciamos con un escaneo de puertos básico:

```bash
nmap -sC -sV -oN nmap/initial 10.10.11.227
```

### Port Scan Results
- **22/tcp**: open ssh
- **80/tcp**: open http (BestKeeper)

## User Access

Al acceder al servidor web encontramos un panel de administración. Tras probar credenciales por defecto `admin:admin`, logramos acceso al panel de control de tickets.

![Panel de administrador de Keeper](https://raw.githubusercontent.com/KyyroxxX/i4mk1r0x/main/assets/machines/keeper_panel.png)

## Root Privilege Escalation

Encontramos un archivo de base de datos `.kdbx` (KeePass). Utilizamos `keepass2john` para obtener el hash:

```python
# Script para automatizar la extracción si fuera necesario
import sys
print("Automatizando...")
```

---
*Writeup generado para el portfolio de Kyrox.*
