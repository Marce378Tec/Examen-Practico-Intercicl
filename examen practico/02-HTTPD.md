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
cp -p /etc/httpd/conf.d/welcome.conf /root/backups_interciclo/welcome.conf.${FECHA}.bak
ls -lh /root/backups_interciclo/
```

### 2. Creación de Página Web Personalizada (`/var/www/html/index.html`)
Se implementó el documento HTML de demostración identificando visiblemente a ambos integrantes en formato `nombre_apellido` (`carlos_jerves` y `marcelino_espinoza`), junto con los parámetros del servidor de laboratorio:

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Sign In - Práctica académica</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      min-height: 100vh;
      background: #121212;
      color: #fff;
      font-family: "Segoe UI", Arial, sans-serif;
      display: flex;
      justify-content: center;
    }

    /* Barra superior (controles de ventana) */
    .window-controls {
      position: absolute;
      top: 8px;
      right: 14px;
      display: flex;
      gap: 22px;
      font-size: 16px;
      color: #fff;
      user-select: none;
    }

    /* Logo (marcador de posición) */
    .logo {
      position: absolute;
      top: 42px;
      left: 28px;
      width: 34px;
      height: 38px;
      background: #fff;
      color: #121212;
      font-size: 9px;
      font-weight: 800;
      line-height: 1;
      text-align: center;
      padding-top: 8px;
      clip-path: polygon(0 0, 100% 0, 100% 80%, 50% 100%, 0 80%);
    }

    .login {
      width: 100%;
      max-width: 498px;
      padding: 55px 16px 24px;
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    h1 {
      font-size: 14px;
      font-weight: 600;
      letter-spacing: .5px;
      margin-bottom: 22px;
    }

    /* Botones sociales */
    .social {
      display: flex;
      gap: 9px;
      width: 100%;
    }
    .social button {
      flex: 1;
      height: 48px;
      border: 2px solid transparent;
      color: #fff;
      font-size: 20px;
      font-weight: 700;
      cursor: pointer;
    }
    .ps   { background: #00a9f0; }
    .xbox { background: #107c10; }
    .nin  { background: #e60012; }
    .fb   { background: #3b5998; }
    .goog { background: #fff; color: #4285f4; }
    .social button:hover { filter: brightness(1.1); }

    /* Separador "or" */
    .divider {
      width: 100%;
      display: flex;
      align-items: center;
      gap: 18px;
      margin: 20px 0 46px;
      font-size: 13px;
    }
    .divider::before, .divider::after {
      content: "";
      flex: 1;
      height: 1px;
      background: #a0a0a0;
    }

    /* Formulario */
    form { width: 100%; }

    input[type="email"],
    input[type="password"] {
      width: 100%;
      height: 48px;
      background: #070707;
      border: 1px solid #5a5a5a;
      color: #fff;
      font-size: 15px;
      padding: 0 16px;
      margin-bottom: 40px;
      outline: none;
    }
    input::placeholder { color: #8a8a8a; text-transform: uppercase; }
    input[type="email"]:focus,
    input[type="password"]:focus { border-color: #fff; }

    .options {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-top: -4px;
      margin-bottom: 26px;
      font-size: 14px;
    }
    .remember { display: flex; align-items: center; gap: 14px; cursor: pointer; }
    .remember input {
      appearance: none;
      width: 20px;
      height: 20px;
      border: 1px solid #8fd49a;
      background: #070707;
      display: grid;
      place-content: center;
      cursor: pointer;
    }
    .remember input:checked::after { content: "✓"; color: #fff; font-size: 14px; }

    a { color: #8fd49a; text-decoration: none; }
    a:hover { text-decoration: underline; }

    .btn-signin {
      width: 100%;
      height: 46px;
      background: #88d19a;
      border: none;
      color: #fff;
      font-size: 14px;
      font-weight: 700;
      letter-spacing: .5px;
      cursor: pointer;
    }
    .btn-signin:hover { background: #99dcaa; }

    .privacy { display: block; text-align: center; margin-top: 12px; font-size: 13px; }

    .signup {
      margin-top: 20px;
      font-size: 14px;
      text-align: center;
    }
    .signup span { margin: 0 6px; }

    .settings {
      position: absolute;
      bottom: 28px;
      left: 38px;
      font-size: 18px;
      color: #fff;
    }
  </style>
</head>
<body>

  <div class="window-controls"><span>—</span><span>☐</span><span>✕</span></div>
  <div class="logo">EPIC<br>GAMES</div>

  <main class="login">
    <h1>SIGN IN</h1>

    <div class="social">
      <button class="ps"   type="button" aria-label="PlayStation">PS</button>
      <button class="xbox" type="button" aria-label="Xbox">X</button>
      <button class="nin"  type="button" aria-label="Nintendo">N</button>
      <button class="fb"   type="button" aria-label="Facebook">f</button>
      <button class="goog" type="button" aria-label="Google">G</button>
    </div>

    <div class="divider">or</div>

    <!-- Formulario de práctica: no envía datos a ningún servidor -->
    <form onsubmit="event.preventDefault(); alert('Esto es solo una práctica de diseño.');">
      <input type="email" placeholder="Email" required>
      <input type="password" placeholder="Password" required>

      <div class="options">
        <label class="remember">
          <input type="checkbox" checked> Remember Me
        </label>
        <a href="#">Forgot Your Password?</a>
      </div>

      <button class="btn-signin" type="submit">SIGN IN</button>
    </form>

    <a class="privacy" href="#">Privacy Policy</a>

    <p class="signup">
      Don't have an Epic Games account? <a href="#">Sign Up</a>
      <span>or</span> <a href="#">Sign In Later</a>
    </p>
  </main>

  <div class="settings">⚙</div>

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
* **Solución aplicada:** Se configuró manualmente la dirección IP `192.168.0.102` como servidor DNS preferido en las propiedades IPv4 del adaptador de red de la máquina cliente y se limpió la caché DNS local (`ipconfig /flushdns`), logrando la resolución inmediata y la carga correcta de la página web.
