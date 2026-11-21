# Servicio 01: Servidor DNS (`named` / BIND)

---

## 01 Arquitectura y Requisitos

### Rol del Servicio
El servicio DNS implementado mediante **BIND (`named`)** actúa como servidor de nombres autoritativo primario para la zona directa local `jerves-espinoza.lab`. Su función es traducir nombres de dominio legibles (`web.jerves-espinoza.lab`, `ssh.jerves-espinoza.lab`, `jerves-espinoza.lab`, etc.) hacia la dirección IP del servidor en la red local, permitiendo el acceso centralizado a los servicios web y de administración remota[cite: 1].

### Especificaciones Técnicas
| Parámetro | Detalle / Valor |
| :--- | :--- |
| **Hostname del Servidor** | `srv-jerves-espinoza` |
| **Dirección IP del Servidor** | `192.168.0.102` |
| **Zona Directa Configurada** | `jerves-espinoza.lab` |
| **Puerto y Protocolo** | `53/TCP` y `53/UDP` |
| **Paquetes Instalados** | `bind`, `bind-utils` |
| **Archivo de Configuración Principal** | `/etc/named.conf` |
| **Archivo de Zona Directa** | `/var/named/jerves-espinoza.lab.zone` |
| **Ruta de Respaldo Previo** | `/root/backups_interciclo/named.conf.YYYY-MM-DD_HHMM.bak` |

---

## 02 Despliegue y Configuración

### 1. Instalación de Paquetes y Respaldo Previo
Antes de intervenir la configuración del servicio, se instalaron los paquetes requeridos y se generó una copia de seguridad con marca de tiempo del archivo original[cite: 1]:

```bash
# Instalación de BIND y utilidades de diagnóstico (dig/nslookup)
dnf install -y bind bind-utils

# Respaldo previo de la configuración original
cp -p /etc/named.conf /root/backups_interciclo/named.conf.$(date +%F_%H%M).bak
ls -lh /root/backups_interciclo/
```

### 2. Configuración Principal (`/etc/named.conf`)
Se habilitó la escucha en la interfaz de red del servidor (`192.168.0.102`) y se permitió la consulta desde la subred local (`192.168.0.0/24`), declarando al final la zona maestra `jerves-espinoza.lab`[cite: 1]:

```text
options {
        listen-on port 53 { 127.0.0.1; 192.168.0.102; };
        listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
        allow-query     { localhost; 192.168.0.0/24; };
        recursion yes;
};

// Declaración de Zona Directa Local
zone "jerves-espinoza.lab" IN {
        type master;
        file "jerves-espinoza.lab.zone";
        allow-update { none; };
};
```

### 3. Creación del Archivo de Zona Directa (`/var/named/jerves-espinoza.lab.zone`)
Se definieron los registros `SOA`, `NS` y los registros `A` asociados al servidor y a los servicios web y SSH apuntando a `192.168.0.102`[cite: 1]:

```text
$TTL 86400
@   IN  SOA     ns1.jerves-espinoza.lab. admin.jerves-espinoza.lab. (
                2026112001  ; Serial
                3600        ; Refresh
                1800        ; Retry
                604800      ; Expire
                86400 )     ; Minimum TTL

; Registros de Servidores de Nombres (NS)
@       IN  NS      ns1.jerves-espinoza.lab.

; Registros de Dirección (A)
@       IN  A       192.168.0.102
ns1     IN  A       192.168.0.102
srv     IN  A       192.168.0.102
web     IN  A       192.168.0.102
www     IN  A       192.168.0.102
ssh     IN  A       192.168.0.102
admin   IN  A       192.168.0.102
```

### 4. Validación de Sintaxis y Arranque del Servicio
```bash
# Asignación de propietario y permisos de lectura para el grupo named
chown root:named /var/named/jerves-espinoza.lab.zone
chmod 640 /var/named/jerves-espinoza.lab.zone

# Verificación de sintaxis de configuración y zona
named-checkconf /etc/named.conf
named-checkzone jerves-espinoza.lab /var/named/jerves-espinoza.lab.zone

# Habilitación al inicio del sistema y arranque inmediato
systemctl enable --now named
systemctl status named
```

---

## 03 Aseguramiento Seguridad SELinux/Firewall

### Gestión de Firewall (`firewalld`)
Se habilitó únicamente el servicio `dns` (puertos `53/TCP` y `53/UDP`) de manera permanente para permitir peticiones desde los clientes de la red sin desactivar el cortafuegos[cite: 2]:

```bash
firewall-cmd --permanent --add-service=dns
firewall-cmd --reload
firewall-cmd --list-all
```

### Gestión de Contextos en SELinux
Manteniendo SELinux activo en modo `Enforcing`, se restauró el contexto de seguridad (`named_zone_t`) sobre el nuevo archivo de zona creado manualmente en `/var/named/` para evitar bloqueos de lectura por parte del demonio `named`[cite: 2]:

```bash
getenforce
restorecon -Rv /var/named/
ls -Z /var/named/jerves-espinoza.lab.zone
```

---

## 04 Pruebas de Aceptación y Troubleshooting

### 1. Configuración y Validación Integral desde el Cliente (Windows)
Desde la máquina cliente (anfitriona), se configuró la interfaz de red `Wi-Fi 3` apuntando al servidor DNS `192.168.0.102`, se limpió la caché DNS y se validó la resolución de nombres y conectividad de todos los servicios[cite: 1]:

```powershell
# 1. Configurar DNS 192.168.0.102 en Wi-Fi 3 y limpiar caché
netsh interface ip set dns name="Wi-Fi 3" static 192.168.0.102
ipconfig /flushdns

# 2. Verificar resolución DNS
nslookup web.jerves-espinoza.lab 192.168.0.102
nslookup ssh.jerves-espinoza.lab 192.168.0.102

# 3. Abrir en el navegador:
# [http://jerves-espinoza.lab](http://jerves-espinoza.lab)

# 4. Probar conexión SSH en puerto 2222 (luego escribe 'exit' para volver a Windows)
ssh -p 2222 jerves_espinoza@ssh.jerves-espinoza.lab

# 5. Probar transferencia SCP (Subida y Descarga desde tu PC Windows)
echo Archivo de prueba Jerves y Espinoza > prueba_subida.txt
scp -P 2222 prueba_subida.txt jerves_espinoza@ssh.jerves-espinoza.lab:/home/jerves_espinoza/
scp -P 2222 jerves_espinoza@ssh.jerves-espinoza.lab:/home/jerves_espinoza/prueba_subida.txt ./prueba_descargada.txt
dir prueba*

# 6. (SOLO AL TERMINAR EL EXAMEN) Restaurar DNS automático en Windows:
# netsh interface ip set dns name="Wi-Fi 3" dhcp
```

**Evidencia en consola (Servidor y Cliente):**
> *(Pegar aquí las capturas de pantalla donde se observe claramente el prompt `root@srv-jerves-espinoza`, el estado `active (running)` de `named` y la respuesta exitosa de `nslookup` desde el cliente)[cite: 1, 2].*
> `![Evidencia DNS Servidor](ruta/a/tu/captura_dns_srv.png)`
> `![Evidencia DNS Cliente](ruta/a/tu/captura_dns_cliente.png)`

### 2. Incidente Encontrado y Solución Aplicada (Troubleshooting)
* **Problema detectado:** Al intentar iniciar el servicio con `systemctl start named` tras crear el archivo `/var/named/jerves-espinoza.lab.zone`, el servicio falló al cargar la zona reportando un error de permisos (`permission denied`) en los registros del sistema (`journalctl -xeu named`)[cite: 2].
* **Causa raíz:** Al crear el archivo de zona como usuario `root`, el propietario y grupo quedaron asignados a `root:root` con permisos restrictivos, impidiendo que el usuario del servicio (`named`) pudiera leer el archivo de zona[cite: 2].
* **Solución aplicada:** Se cambió la propiedad del archivo al grupo `named`, se ajustaron los permisos a `640` y se restableció la etiqueta de contexto SELinux antes de reiniciar el servicio[cite: 2]:
  ```bash
  chown root:named /var/named/jerves-espinoza.lab.zone
  chmod 640 /var/named/jerves-espinoza.lab.zone
  restorecon -v /var/named/jerves-espinoza.lab.zone
  systemctl restart named
  ```
