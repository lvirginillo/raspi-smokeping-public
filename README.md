# 📡 raspi-smokeping — Monitoreo de latencia en red doméstica

Configuración de SmokePing sobre Raspberry Pi para monitoreo continuo de latencia y pérdida de paquetes en dispositivos de red local, con acceso remoto vía Tailscale VPN.

> Autor: [Lautaro Virginillo](https://www.linkedin.com/in/lautaro-virginillo/)

---

## ¿Qué hace este proyecto?

Levanta SmokePing como servicio systemd en una Raspberry Pi y lo expone vía Apache. Monitorea en tiempo real la latencia hacia distintos puntos de la red: salida a internet, dispositivos IoT, cámaras y la propia infraestructura local.

Los gráficos históricos quedan accesibles desde cualquier dispositivo de la red, o de forma remota a través de Tailscale sin necesidad de exponer puertos al exterior.

---

## 🗺️ Arquitectura

```
[Dispositivos LAN / Internet] ──ICMP──► [SmokePing] ──► [Apache :80] ──► [Navegador]
                                         Raspberry Pi 4B (Tailscale VPN)
```

---

## 📡 Categorías de targets monitoreados

El archivo `Targets` define los grupos de dispositivos a monitorear. La estructura típica incluye:

| Grupo | Ejemplos de targets |
|---|---|
| Red local | Router, cable modem, salida ISP |
| Infraestructura | Raspberry Pi (ETH y WiFi) |
| Dispositivos IoT | Cámaras IP, dispositivos Smartlife |
| Internet | Hosts externos de referencia |

> El archivo `Targets` con las IPs reales no está incluido en este repositorio por privacidad.  
> Se incluye un `Targets.example` con la estructura completa para referencia.

---

## 📁 Estructura del proyecto

```
raspi-smokeping/
├── Targets.example    # Estructura de configuración (sin IPs reales)
└── README.md

/etc/smokeping/config.d/   # Configuración activa en el sistema (fuera del repo)
└── Targets
```

> El archivo activo vive en `/etc/smokeping/config.d/`. El repo funciona como referencia y documentación.

---

## 🛠️ Servicios involucrados

| Servicio | Manejado por | Arranca automáticamente |
|---|---|---|
| smokeping | systemd | ✅ sí |
| apache2 | systemd | ✅ sí |

---

## 🚀 Instalación

```bash
# Instalar dependencias
sudo apt update
sudo apt install smokeping fping apache2

# Habilitar e iniciar el servicio
sudo systemctl enable smokeping
sudo systemctl start smokeping

# Verificar que Apache sirve la interfaz
# http://<IP_LOCAL>/smokeping/
```

---

## ⚙️ Configuración de targets

Copiá el archivo de ejemplo y editalo con tus propios hosts:

```bash
sudo cp Targets.example /etc/smokeping/config.d/Targets
sudo nano /etc/smokeping/config.d/Targets

# Verificar sintaxis antes de reiniciar
sudo smokeping --check

# Aplicar cambios
sudo systemctl restart smokeping
```

---

## 🔧 Comandos útiles

```bash
# Estado del servicio
sudo systemctl status smokeping

# Ver logs
journalctl -u smokeping -n 50 --no-pager

# Reiniciar tras cambios
sudo systemctl restart smokeping

# Datos históricos RRD
ls /var/lib/smokeping/
```

---

## 🌐 Acceso remoto

El acceso fuera de la red local se gestiona a través de Tailscale VPN, sin exponer puertos al exterior:

```
http://<TAILSCALE_IP>/smokeping/
```

---

## 📦 Dependencias

- `smokeping` — motor de monitoreo y generación de gráficas RRD
- `apache2` — servidor web para la interfaz
- `fping` — backend de ping utilizado por SmokePing

---

## Notas

- Probado en Raspberry Pi 4B, Raspbian Bookworm (ARM64)
- Apache en puerto 80, compartido con otros proyectos del mismo servidor
- Los datos históricos RRD se almacenan en `/var/lib/smokeping/` y persisten entre reinicios
