# ==============================================================================
# SERVICIO 02: SERVIDOR WEB (HTTPD / Apache) - IP: 192.168.0.110
# ==============================================================================

# 1. Instalación de Apache y respaldo previo con marca de tiempo
dnf install -y httpd
FECHA=$(date +%F_%H%M)
cp -p /etc/httpd/conf/httpd.conf /root/backups_interciclo/httpd.conf.${FECHA}.bak
[ -f /etc/httpd/conf.d/welcome.conf ] && cp -p /etc/httpd/conf.d/welcome.conf /root/backups_interciclo/welcome.conf.${FECHA}.bak
[ -f /var/www/html/index.html ] && cp -p /var/www/html/index.html /root/backups_interciclo/index.html.${FECHA}.bak

# 2. Creación de página personalizada (Modalidad Parejas: Jerves y Espinoza)
cat << 'EOF' > /var/www/html/index.html
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
EOF

# 3. Restaurar contexto SELinux, arrancar servicio y abrir Firewall
restorecon -Rv /var/www/html/
systemctl enable --now httpd
firewall-cmd --permanent --add-service=http
firewall-cmd --reload

# 4. Verificación
systemctl status httpd --no-pager
firewall-cmd --list-all
ls -Z /var/www/html/index.html
