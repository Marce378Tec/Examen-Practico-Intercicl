# Servicio 02: Servidor Web (`httpd` / Apache)

---

## 01 Arquitectura y Requisitos

### Rol del Servicio
El servidor web implementado mediante **Apache (`httpd`)** aloja y sirve contenido HTML estático a través del protocolo HTTP para los clientes de la red local. En cumplimiento con la modalidad en parejas, sirve una página de demostración personalizada en entorno aislado de laboratorio accesible mediante el nombre de dominio `web.jerves-espinoza.lab` resuelto por el servidor DNS local.

### Especificaciones Técnicas
| Parámetro | Detalle / Valor |
| :--- | :--- |
| **Hostname del Servidor** | `srv-jerves-espinoza` |
| **Dirección IP del Servidor** | `192.168.0.110` |
| **Dominio Web Asignado** | `http://web.jerves-espinoza.lab` |
| **Puerto y Protocolo** | `80/TCP` (HTTP) |
| **Paquete Instalado** | `httpd` |
| **Archivo de Configuración Principal** | `/etc/httpd/conf/httpd.conf` |
| **Directorio Raíz Web (DocumentRoot)** | `/var/www/html/` |
| **Archivo Web Personalizado** | `/var/www/html/index.html` |
| **Ruta de Respaldo Previo** | `/root/backups_interciclo/httpd.conf.YYYY-MM-DD_HHMM.bak` |

---

## 02 Despliegue y Configuración

### 1. Instalación de Paquete y Respaldo Previo
Antes de modificar la configuración o el contenido web por defecto, se instaló el servicio Apache y se respaldaron los archivos originales con marca de tiempo:

```bash
# Instalación del servidor web Apache
dnf install -y httpd

# Respaldo previo de la configuración y página de bienvenida
FECHA=$(date +%F_%H%M)
cp -p /etc/httpd/conf/httpd.conf /root/backups_interciclo/httpd.conf.${FECHA}.bak
cp -p /etc/httpd/conf.d/welcome.conf /root/backups_interciclo/welcome.conf.${FECHA}.bak
ls -lh /root/backups_interciclo/
```

### 2. Creación de Página Web Personalizada (`/var/www/html/index.html`)
Se implementó el documento HTML de demostración identificando visiblemente a ambos integrantes en formato `nombre_apellido`, junto con los parámetros del servidor de laboratorio:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Infraestructura de Servicios de Red - Jerves & Espinoza</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; text-align: center; margin-top: 60px; background-color: #eef2f5; }
        .card { background: #ffffff; padding: 35px; border-radius: 12px; display: inline-block; box-shadow: 0 4px 12px rgba(0,0,0,0.1); max-width: 550px; }
        h1 { color: #1e3a8a; margin-bottom: 10px; }
        .sub { color: #475569; font-size: 1.05em; }
        .integrantes { background: #f0fdf4; border: 1px solid #bbf7d0; padding: 15px; border-radius: 8px; font-size: 1.2em; color: #15803d; font-weight: bold; margin: 20px 0; }
        .footer { font-size: 0.9em; color: #64748b; }
    </style>
</head>
<body>
    <div class="card">
        <h1>Examen Práctico de Interciclo</h1>
        <p class="sub">Servidor Web Apache (HTTPD) - Entorno de Laboratorio</p>
        <div class="integrantes">
            Integrante 1: nombre_jerves <br>
            Integrante 2: nombre_espinoza
        </div>
        <p class="footer">Host: srv-jerves-espinoza | IP: 192.168.0.110 | Dominio: web.jerves-espinoza.lab</p>
    </div>
</body>
</html>
```

### 3. Habilitación y Arranque del Servicio
```bash
systemctl enable --now httpd
systemctl status httpd
```

---

## 03 Aseguramiento Seguridad SELinux/Firewall

### Gestión de Firewall (`firewalld`)
Se habilitó únicamente el tráfico entrante para el servicio `http` (`80/TCP`) de manera permanente para permitir el acceso desde el navegador de la máquina cliente:

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
firewall-cmd --list-all
```

### Gestión de Contextos en SELinux
Manteniendo SELinux en modo `Enforcing`, se verificó y restauró el contexto de seguridad `httpd_sys_content_t` en `/var/www/html/index.html` para garantizar que el proceso de Apache tenga autorización de lectura sobre el archivo creado:

```bash
getenforce
restorecon -Rv /var/www/html/
ls -Z /var/www/html/index.html
```

---

## 04 Pruebas de Aceptación y Troubleshooting

### 1. Validación Funcional desde el Servidor y Cliente
Se comprobó la respuesta HTTP local desde consola mediante `curl` y desde el navegador web de la máquina cliente apuntando al dominio `http://web.jerves-espinoza.lab`:

```bash
# Verificación de cabeceras HTTP desde consola
curl -I [http://web.jerves-espinoza.lab](http://web.jerves-espinoza.lab)
```

**Evidencia en consola y navegador:**
> *(Pegar aquí las capturas de pantalla donde se observe el estado `active (running)` de `httpd` con el prompt `root@srv-jerves-espinoza`, y el navegador web mostrando la página cargada mediante `http://web.jerves-espinoza.lab` con ambos nombres).*
> `![Evidencia HTTPD Servidor](ruta/a/tu/captura_httpd_srv.png)`
> `![Evidencia HTTPD Navegador](ruta/a/tu/captura_httpd_web.png)`

### 2. Incidente Encontrado y Solución Aplicada (Troubleshooting)
* **Problema detectado:** Al ingresar `http://web.jerves-espinoza.lab` en el navegador de la máquina anfitriona por primera vez, el navegador arrojó un error de sitio no encontrado (`DNS_PROBE_FINISHED_NXDOMAIN`).
* **Causa raíz:** La tarjeta de red de la máquina anfitriona (cliente) aún estaba utilizando el servidor DNS automático del router doméstico (DHCP), el cual desconoce la zona interna `.lab`.
* **Solución aplicada:** Se configuró manualmente la dirección IP `192.168.0.110` como servidor DNS preferido en las propiedades IPv4 del adaptador de red de la máquina cliente y se limpió la caché DNS local (`ipconfig /flushdns`), logrando la resolución inmediata y la carga correcta de la página web.
