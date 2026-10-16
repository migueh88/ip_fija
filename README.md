# ip_fija

## 🛠️ Instructivo – Fijar IP de salida en 184.107.6.28

### 1. Problema detectado

* El servidor tenía varias interfaces de red (`eth0`..`eth4`), cada una con una IP pública.
* OpenNebula (`one-context`) generaba un `default gateway` en **todas las NICs**.
* Eso hacía que el tráfico saliente a Internet usara a veces la **184.107.6.5**, otras la **.28**, etc.
* Resultado: **la IP pública saliente iba cambiando sola** (`curl ifconfig.me` mostraba diferentes IPs).

---

### 2. Solución aplicada

Como Sunstone/OpenNebula no permitía quitar los `GATEWAY` extra fácilmente, se creó un servicio en **systemd** que corrige la ruta por defecto en cada arranque.

#### Pasos:

1. Crear archivo del servicio:

```bash
nano /etc/systemd/system/fix-default-route.service
```

2. Pegar este contenido:

```ini
[Unit]
Description=Fix default route to eth0
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/bin/ip route replace default via 184.107.6.62 dev eth0

[Install]
WantedBy=multi-user.target
```

👉 Nota: `184.107.6.62` es el **gateway** de tu red (ajústalo si tu proveedor usa otro).

3. Guardar y activar el servicio:

```bash
systemctl daemon-reload
systemctl enable fix-default-route.service
```

4. Reiniciar el servidor y verificar:

```bash
curl ifconfig.me
```

Debe devolver siempre **184.107.6.28**. 🚀

---

### 3. Resultado

* **Entrada:** cada dominio responde en su propia IP (ej: .28, .5, .18, etc.).
* **Salida global:** todo el servidor (curl, apt, etc.) sale ahora fijo por la **184.107.6.28**.
* **Correo (Exim/WHM):** si quieres que cada dominio envíe desde su propia IP, se configura en `/etc/mailips` y funciona porque ya existen tablas de *policy routing* en tu `60-policy.yaml`.

---

✅ Con esto el problema quedó solucionado sin tener que recrear la VM en OpenNebula.

---
