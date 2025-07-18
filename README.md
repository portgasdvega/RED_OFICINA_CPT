# Red de Oficina con DHCP, NAT, DNS y Acceso WiFi (Packet Tracer)

## Topología

Una red de oficina simulada que incluye:

* Segmento LAN `192.168.1.0/24`
* Servidor DHCP (asigna IPs a PCs y dispositivos WiFi)
* Router LAN con configuración de NAT (PAT) para salida a Internet
* Router ISP simulado (proveedor)
* Servidor DNS externo (`8.8.8.8`) que resuelve `www.servidor.com`
* Wireless Router funcionando como Access Point
* Laptop conectada por WiFi con salida a Internet

## Direccionamiento IP

| Dispositivo       | Interfaz   | IP            | Rol                                                  |
| ----------------- | ---------- | ------------- | ---------------------------------------------------- |
| Servidor DHCP     | Fa0        | 192.168.1.2   | DHCP / DNS relay opc.                                |
| PC0 - PC3         | Fa0        | 192.168.1.3-6 | Clientes LAN                                         |
| Wireless Router   | LAN (Eth1) | 192.168.1.7   | Punto de acceso WiFi                                 |
| Laptop WiFi       | WiFi       | 192.168.1.99  | Cliente inalámbrico                                  |
| Router LAN (2911) | G0/0 (LAN) | 192.168.1.1   | Gateway, NAT inside                                  |
|                   | G0/1 (WAN) | 200.0.0.1     | NAT outside                                          |
| Router ISP (2911) | G0/0       | 200.0.0.2     | Enlace WAN                                           |
|                   | G0/1       | 8.8.8.1       | Salida a Internet sim.                               |
| Servidor DNS      | Fa0        | 8.8.8.8       | Resuelve [www.servidor.com](http://www.servidor.com) |

## Configuraciones clave

### Router LAN (NAT y ruta por defecto)

```bash
interface gig0/0
 ip address 192.168.1.1 255.255.255.0
 ip nat inside
 no shutdown

interface gig0/1
 ip address 200.0.0.1 255.255.255.252
 ip nat outside
 no shutdown

access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface gig0/1 overload
ip route 0.0.0.0 0.0.0.0 200.0.0.2
```

### Router ISP

```bash
interface gig0/0
 ip address 200.0.0.2 255.255.255.252
 no shutdown

interface gig0/1
 ip address 8.8.8.1 255.255.255.0
 no shutdown
```

### Servidor DHCP (Services > DHCP)

* Default Gateway: `192.168.1.1`
* DNS Server: `8.8.8.8`
* Start IP: `192.168.1.100`
* Subnet Mask: `255.255.255.0`

### Servidor DNS (Services > DNS)

* Nombre: `www.servidor.com`
* IP: `8.8.8.8`
* Estado: ON

### Wireless Router (modo Access Point)

* Conectado por **puerto LAN (Eth1)** al switch
* IP: `192.168.1.7`
* DHCP: **Desactivado**
* SSID: `oficinawifi`
* Seguridad: WPA2-PSK, clave: `segura123`

## Verificaciones exitosas

Desde laptop WiFi (IP `192.168.1.99`):

```bash
ping 192.168.1.1          # Gateway
ping 200.0.0.2            # Router ISP
ping 8.8.8.8              # Servidor DNS
nslookup www.servidor.com # Resolución DNS correcta
```

Navegador accede correctamente a:

```
http://www.servidor.com
```

## Estado final

* DHCP funcionando
* NAT (PAT) activo
* Acceso a Internet simulado
* Resolución DNS activa
* WiFi operativo con acceso completo

---
