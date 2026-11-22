# Servicio 03: Acceso Remoto Seguro (`sshd` / SSH) y Transferencia SCP

---

## 01 Arquitectura y Requisitos

### Rol del Servicio
El servicio **OpenSSH (`sshd`)** proporciona administración remota cifrada y transferencia segura de archivos mediante **SCP (Secure Copy Protocol)**. Como medida de endurecimiento (*hardening*), se modificó el puerto de escucha estándar (`22/TCP`) hacia el puerto personalizado **`2222/TCP`**, permitiendo el acceso remoto y la transferencia bidireccional de archivos desde la máquina cliente utilizando el nombre de dominio `ssh.jerves-espinoza.lab`.

### Especificaciones Técnicas
| Parámetro | Detalle / Valor |
| :--- | :--- |
| **Hostname del Servidor** | `srv-jerves-espinoza` |
| **Dirección IP del Servidor** | `192.168.0.107` |
| **Dominio de Acceso Remoto** | `ssh.jerves-espinoza.lab` |
| **Puerto Personalizado** | `2222/TCP` |
| **Usuario Remoto Operativo** | `jerves_espinoza` |
| **Paquetes Instalados** | `openssh-server`, `policycoreutils-python-utils` |
| **Archivo de Configuración Principal** | `/etc/ssh/sshd_config` |
| **Ruta de Respaldo Previo** | `/root/backups_interciclo/sshd_config.YYYY-MM-DD_HHMM.bak` |

---

## 02 Despliegue y Configuración

### 1. Respaldo Previo de Configuración y Creación de Usuario
Antes de modificar la configuración del demonio SSH, se generó una copia de respaldo con marca de tiempo y se creó una cuenta de usuario operativa para las conexiones remotas:

```bash
# Respaldo previo de sshd_config
cp -p /etc/ssh/sshd_config /root/backups_interciclo/sshd_config.$(date +%F_%H%M).bak
ls -lh /root/backups_interciclo/

# Creación de usuario operativo para SSH y SCP
useradd jerves_espinoza
passwd jerves_espinoza
```

### 2. Modificación del Puerto de Escucha (`/etc/ssh/sshd_config`)
Se editó el archivo `/etc/ssh/sshd_config` para sustituir el puerto por defecto (`22`) por el puerto personalizado **`2222`**:

```text
# Configuración aplicada en /etc/ssh/sshd_config
Port 2222
AddressFamily any
ListenAddress 0.0.0.0
PubkeyAuthentication yes
PasswordAuthentication yes
```

### 3. Reinicio y Validación del Servicio
```bash
sshd -t
systemctl enable --now sshd
systemctl restart sshd
systemctl status sshd
```

---

## 03 Aseguramiento Seguridad SELinux/Firewall

### Gestión de Puertos en SELinux
Con SELinux activo en modo `Enforcing`, el demonio `sshd` únicamente tiene permiso para escuchar en el puerto `22/TCP`. Para evitar el bloqueo del servicio al cambiar al puerto `2222/TCP`, se registró el nuevo puerto bajo el contexto `ssh_port_t` mediante `semanage`:

```bash
# Instalación de utilidades de gestión de políticas SELinux
dnf install -y policycoreutils-python-utils

# Registro del puerto 2222/TCP en el contexto ssh_port_t
semanage port -a -t ssh_port_t -p tcp 2222

# Verificación de puertos autorizados para SSH
semanage port -l | grep ssh
getenforce
```

### Gestión de Firewall (`firewalld`)
Se habilitó explícitamente el puerto personalizado `2222/TCP` de forma permanente en `firewalld`:

```bash
firewall-cmd --permanent --add-port=2222/tcp
firewall-cmd --reload
firewall-cmd --list-all
```

---

## 04 Pruebas de Aceptación y Troubleshooting

### 1. Validación de Conexión SSH y Transferencia Bidireccional SCP
Desde la máquina cliente (anfitriona), se validó la conexión remota y tanto la subida como la descarga de archivos mediante SCP utilizando el dominio `ssh.jerves-espinoza.lab` y el puerto `2222`:

```bash
# 1. Conexión remota SSH por dominio y puerto personalizado
ssh -p 2222 jerves_espinoza@ssh.jerves-espinoza.lab

# 2. Subida de archivo desde cliente hacia servidor (SCP)
echo "Evidencia de transferencia Jerves - Espinoza" > prueba_subida.txt
scp -P 2222 prueba_subida.txt jerves_espinoza@ssh.jerves-espinoza.lab:/home/jerves_espinoza/

# 3. Descarga de archivo desde servidor hacia cliente (SCP)
scp -P 2222 jerves_espinoza@ssh.jerves-espinoza.lab:/home/jerves_espinoza/prueba_subida.txt ./prueba_descargada.txt
```

**Evidencia en consola (SSH y SCP Bidireccional):**
<img width="941" height="69" alt="image" src="https://github.com/user-attachments/assets/ff1206fe-0ad0-4b62-8cf4-f5f8e426a339" />
<img width="940" height="410" alt="image" src="https://github.com/user-attachments/assets/ccca3b93-9212-4b42-ac14-5e2a847f238f" />
<img width="940" height="498" alt="image" src="https://github.com/user-attachments/assets/2ecd87ce-bacf-4242-92c9-5d4e47713cae" />


### 2. Incidente Encontrado y Solución Aplicada (Troubleshooting)
- **Problema detectado:** Al cambiar la directiva `Port 2222` en `/etc/ssh/sshd_config` y ejecutar `systemctl restart sshd`, el servicio falló al iniciar con el mensaje `Job for sshd.service failed because the control process exited with error code` y en los registros apareció `error: Bind to port 2222 on 0.0.0.0 failed: Permission denied`.
- **Causa raíz:** La política predeterminada de SELinux en modo `Enforcing` restringe el enlace (*bind*) del demonio `sshd` exclusivamente al puerto `22/TCP`, bloqueando cualquier puerto no registrado en `ssh_port_t`.
- **Solución aplicada:** Se instaló el paquete `policycoreutils-python-utils` y se añadió el puerto `2222/TCP` a la definición de puertos permitidos para SSH en SELinux antes de reiniciar el demonio:
  ```bash
  semanage port -a -t ssh_port_t -p tcp 2222
  systemctl restart sshd
  ```
