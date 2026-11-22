# Examen Práctico de Interciclo
## Infraestructura de Servicios de Red en Linux (DNS, HTTPD y Acceso Remoto Seguro)

---

### 📋 Información General
* **Asignatura:** Sistemas Operativos 1
* **Modalidad:** En Parejas
* **Integrante 1:** Carlos Jerves
* **Integrante 2:** Marcelino Espinoza
* **Sistema Operativo:** CentOS Stream 10 (sin GUI - Interfaz de Línea de Comandos)
* **Fecha:** 20 de Noviembre 2026

---

### 🖥️ Parámetros del Servidor y Red
| Parámetro | Valor Configurado |
| :--- | :--- |
| **Hostname del Servidor** | `srv-jerves-espinoza` |
| **Dirección IP (VM)** | `192.168.0.107/24` |
| **Dominio DNS Local** | `jerves-espinoza.lab` |
| **Directorio de Respaldos** | `/root/backups_interciclo/` |
| **Seguridad Activa** | `SELinux (Enforcing)` + `firewalld` |

---

### 📂 Índice de Documentación Técnica
Cada documento técnico sigue la estructura obligatoria de 4 secciones (*01 Arquitectura y Requisitos, 02 Despliegue y Configuración, 03 Aseguramiento Seguridad SELinux/Firewall, 04 Pruebas de Aceptación y Troubleshooting*):

1. **[01-DNS.md](./examen%20practico/01-DNS.md)** — Implementación y resolución de zona directa con BIND (`named`).
2. **[02-HTTPD.md](./examen%20practico/02-HTTPD.md)** — Despliegue de servidor web Apache y página personalizada de laboratorio.
3. **[03-SSH.md](./examen%20practico/03-SSH.md)** — Hardening de acceso remoto SSH en puerto personalizado y transferencia bidireccional SCP.

---

### 🛠️ Evidencia Rápida: Entorno Base y Respaldos
Antes de intervenir los servicios, se configuró la identidad del nodo, se verificó la conectividad externa (10 paquetes) y se creó el directorio de copias de seguridad:

```bash
# 1. Asignación de Hostname
hostnamectl set-hostname srv-jerves-espinoza

# 2. Verificación de red y salida a Internet
ip -4 addr show
ping -c 10 8.8.8.8

# 3. Creación de carpeta de respaldos
mkdir -p /root/backups_interciclo
ls -lh /root/backups_interciclo/
```

**Evidencia en consola (Hostname, IP y Ping de 10 paquetes):**
<img width="1917" height="726" alt="image" src="https://github.com/user-attachments/assets/cf54c9a0-602f-434f-86ea-ad5c53a31e1d" />
<img width="1157" height="383" alt="image" src="https://github.com/user-attachments/assets/ed3efea1-9c51-4741-b898-ddb57a3d80e8" />



---

### 🎥 Entregable Audiovisual
* **Enlace al Video Demostrativo:** `https://youtu.be/-oVp-ugUQHg`
