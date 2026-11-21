# Servicio 02: Servidor Web (`httpd` / Apache)

---

## 01 Arquitectura y Requisitos

### Rol del Servicio
El servidor web implementado mediante **Apache (`httpd`)** aloja y sirve contenido HTML estático a través del protocolo HTTP para los clientes de la red local. En cumplimiento con la modalidad en parejas, sirve una página de demostración personalizada en entorno aislado de laboratorio accesible mediante el nombre de dominio `web.jerves-espinoza.lab` resuelto por el servidor DNS local.

### Especificaciones Técnicas
| Parámetro | Detalle / Valor |
| :--- | :--- |
| **Hostname del Servidor** | `srv-jerves-espinoza` |
| **Dirección IP del Servidor** | `192.168.0.102` |
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
cp -p /etc/httpd/conf.d/welcome.conf /root/backups_inter
* **Causa raíz:** La tarjeta de red de la máquina anfitriona (cliente) aún estaba utilizando el servidor DNS automático del router doméstico (DHCP), el cual desconoce la zona interna `.lab`.
* **Solución aplicada:** Se configuró manualmente la dirección IP `192.168.0.110` como servidor DNS preferido en las propiedades IPv4 del adaptador de red de la máquina cliente y se limpió la caché DNS local (`ipconfig /flushdns`), logrando la resolución inmediata y la carga correcta de la página web.
