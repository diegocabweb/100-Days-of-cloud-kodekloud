# Ejercicio: Configuración de EC2 con Application Load Balancer (ALB)

## 📋 Descripción del Ejercicio

El Nautilus Development Team necesita configurar una nueva instancia EC2 con un servidor web que sea parte de un Application Load Balancer (ALB) para garantizar alta disponibilidad y mejor gestión del tráfico.

### Objetivos del Ejercicio

1. Crear un Security Group para la instancia EC2
2. Crear una instancia EC2 con Nginx mediante User Data
3. Configurar un Application Load Balancer (ALB)
4. Crear un Target Group
5. Enrutar tráfico del ALB a la instancia EC2
6. Verificar que el servidor web sea accesible mediante el DNS del ALB

---

## 🏗️ Diagrama de Alto Nivel - Arquitectura General
```bash
┌─────────────────────────────────────────────────────────────────────────────┐
│                            INTERNET / USUARIOS                              │
│                                                                             │
│                              http://<ALB-DNS>                               │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      APPLICATION LOAD BALANCER (ALB)                        │
│                          Nombre: devops-alb                                 │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                     LISTENER (Puerto 80)                            │    │
│  │                        Protocol: HTTP                               │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                  │                                          │
│                                  ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │              DEFAULT ACTION: Forward to devops-tg                   │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │              SECURITY GROUP: default-sg                             │    │
│  │              Regla: Puerto 80 abierto (0.0.0.0/0)                   │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           TARGET GROUP (devops-tg)                          │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                   HEALTH CHECK CONFIGURATION                        │    │
│  │  ┌─────────────────────────────────────────────────────────────┐    │    │
│  │  │  Protocol: HTTP  │  Path: /  │  Port: 80                    │    │    │
│  │  │  Interval: 30s   │  Timeout: 5s  │  Healthy Threshold: 2    │    │    │
│  │  └─────────────────────────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                  │                                          │
│                                  ▼                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                   REGISTERED TARGETS                                │    │
│  │  ┌─────────────────────────────────────────────────────────────┐    │    │
│  │  │  Instance ID: i-xxxxxxxx  │  Port: 80  │  Status: Healthy   │    │    │
│  │  └─────────────────────────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          INSTANCIA EC2 (devops-ec2)                         │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    USER DATA SCRIPT                                 │    │
│  │  ┌─────────────────────────────────────────────────────────────┐    │    │
│  │  │  1. apt-get update -y                                       │    │    │
│  │  │  2. apt-get install nginx -y                                │    │    │
│  │  │  3. systemctl start nginx                                   │    │    │
│  │  │  4. systemctl enable nginx                                  │    │    │
│  │  │  5. Crear página HTML personalizada                         │    │    │
│  │  └─────────────────────────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                    WEB SERVER (NGINX)                               │    │
│  │  ┌─────────────────────────────────────────────────────────────┐    │    │
│  │  │  Puerto: 80  │  Servicio: Running  │  Estado: Activo        │    │    │
│  │  └─────────────────────────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │              SECURITY GROUP: devops-sg                              │    │
│  │              Regla: Puerto 80 abierto (0.0.0.0/0)                   │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
```
## Diagrama de flujo de solicitudes
```bash
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│                  │     │                  │     │                  │
│  Usuario ingresa │     │  ALB (devops-alb)│     │  Target Group    │
│  URL del ALB     │────▶│  recibe solicitud│────▶│  (devops-tg)     │
│                  │     │                  │     │                  │
└──────────────────┘     └──────────────────┘     └────────┬─────────┘
                                                           │
                                                           ▼
                                                  ┌──────────────────┐
                                                  │                  │
                                                  │  EC2 (devops-    │
                                                  │  ec2) procesa    │
                                                  │  la solicitud    │
                                                  │                  │
                                                  └──────────────────┘
                                                           │
                                                           ▼
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│                  │     │                  │     │                  │
│  Usuario recibe  │◀────│  ALB envía       │◀────│  Nginx genera    │
│  respuesta       │     │  respuesta al    │     │  respuesta       │
│                  │     │  usuario         │     │                  │
└──────────────────┘     └──────────────────┘     └──────────────────┘
```
## Diagrama de Componentes
```bash
┌─────────────────────────────────────────────────────────────────────────┐
│                               AWS CLOUD                                 │
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                         VPC (Default)                           │    │
│  │                                                                 │    │
│  │  ┌──────────────────────────────────────────────────────────┐   │    │
│  │  │                    Subnet (Zona A)                       │   │    │
│  │  │                                                          │   │    │
│  │  │  ┌─────────────┐     ┌─────────────┐                     │   │    │
│  │  │  │   devops-sg │     │   default-sg│                     │   │    │
│  │  │  │   (SG-EC2)  │     │   (SG-ALB)  │                     │   │    │
│  │  │  └─────────────┘     └─────────────┘                     │   │    │
│  │  │         │                    │                           │   │    │
│  │  │         ▼                    ▼                           │   │    │
│  │  │  ┌─────────────┐     ┌─────────────┐                     │   │    │
│  │  │  │ devops-ec2  │     │ devops-alb  │                     │   │    │
│  │  │  │   (EC2)     │◀────│   (ALB)     │                     │   │    │
│  │  │  │   Port:80   │     │   Port:80   │                     │   │    │
│  │  │  └─────────────┘     └─────────────┘                     │   │    │
│  │  │         │                    │                           │   │    │
│  │  │         ▼                    ▼                           │   │    │
│  │  │  ┌─────────────┐     ┌─────────────┐                     │   │    │
│  │  │  │    Nginx    │     │ devops-tg   │                     │   │    │
│  │  │  │  (WebApp)   │     │ (Target Grp)│                     │   │    │
│  │  │  └─────────────┘     └─────────────┘                     │   │    │
│  │  │                                                          │   │    │
│  │  └──────────────────────────────────────────────────────────┘   │    │
│  │                                                                 │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```
## Diagrama de Secuencia
```bash
┌──────┐     ┌──────┐     ┌─────────┐     ┌──────┐     ┌──────┐
│      │     │      │     │         │     │      │     │      │
│ ALB  │     │  TG  │     │   EC2   │     │Nginx │     │  SG  │
│      │     │      │     │         │     │      │     │      │
└──┬───┘     └──┬───┘     └────┬────┘     └──┬───┘     └──┬───┘
   │            │              │             │            │
   │  Health Check cada 30s    │             │            │
   │───────────▶│              │             │            │
   │            │  Verificar estado          │            │
   │            │────────────▶ │             │            │
   │            │              │  GET /      │            │
   │            │              │───────────▶ │            │
   │            │              │             │            │
   │            │              │  200 OK     │            │
   │            │              │◀─────────── │            │
   │            │              │             │            │
   │            │  Estado: Healthy           │            │
   │            │◀──────────── │             │            │
   │            │              │             │            │
   │  Estado: Healthy          │             │            │
   │◀───────────│              │             │            │
   │            │              │             │            │
```
## 📋 Tabla de Componentes

| Componente | Nombre | Propósito |
|------------|--------|-----------|
| **Application Load Balancer** | `devops-alb` | Distribuir tráfico entrante entre instancias EC2 |
| **Listener** | Puerto 80 | Escuchar solicitudes HTTP en puerto 80 |
| **Target Group** | `devops-tg` | Grupo de instancias que reciben tráfico del ALB |
| **Health Check** | HTTP:80:/ | Verificar que la instancia esté funcionando correctamente |
| **Security Group (ALB)** | `default-sg` | Controlar tráfico entrante al ALB (Puerto 80) |
| **Security Group (EC2)** | `devops-sg` | Controlar tráfico entrante a la instancia (Puerto 80) |
| **EC2 Instance** | `devops-ec2` | Servidor web que ejecuta Nginx |
| **User Data** | Script Bash | Automatizar instalación y configuración de Nginx |
| **Nginx** | Web Server | Servir contenido web en puerto 80 |

---

## 🔄 Resumen del Flujo de Datos
Usuario → http://devops-alb-DNS
↓

ALB recibe solicitud en puerto 80
↓

ALB verifica Target Group (devops-tg)
↓

ALB reenvía a instancia EC2 (devops-ec2) puerto 80
↓

Security Group (devops-sg) permite tráfico
↓

Nginx procesa solicitud
↓

Nginx devuelve página HTML
↓

ALB envía respuesta al usuario
↓

Usuario ve la página web

---

## 📝 Configuración Paso a Paso (Consola Web)

### Paso 1: Crear el Security Group (devops-sg)

1. Ingresa a la consola de AWS → Servicio **EC2**
2. En el menú lateral izquierdo, haz clic en **"Security Groups"**
3. Haz clic en **"Create security group"**

**Configuración:**
- **Security group name:** `devops-sg`
- **Description:** `Security group for devops EC2 instance`
- **VPC:** Selecciona la VPC por defecto

**Reglas de entrada (Inbound rules):**
- **Type:** HTTP | **Protocol:** TCP | **Port range:** 80 | **Source:** `0.0.0.0/0`
- **Type:** SSH | **Protocol:** TCP | **Port range:** 22 | **Source:** `Tu IP`

---

### Paso 2: Crear la instancia EC2 (devops-ec2)

1. En el servicio **EC2**, haz clic en **"Instances"** → **"Launch instance"**

**Configuración:**
- **Name:** `devops-ec2`
- **AMI:** Ubuntu 22.04 LTS
- **Instance type:** `t2.micro`
- **Key pair:** Selecciona o crea una
- **Network settings:** 
  - VPC: Default VPC
  - Subnet: Cualquiera
  - Auto-assign public IP: Enable
  - Firewall: Selecciona `devops-sg`

**User Data (Advanced details):**
```bash
#!/bin/bash
apt-get update -y
apt-get install nginx -y
systemctl start nginx
systemctl enable nginx
echo "<h1>Bienvenido a devops-ec2 - Servidor Nginx</h1>" > /var/www/html/index.html
Paso 3: Crear el Target Group (devops-tg)
En EC2 → "Target Groups" → "Create target group"
```
Configuración:

Target type: Instances

Target group name: devops-tg

Protocol: HTTP | Port: 80

VPC: Default VPC

Health check path: /

Register targets:

Selecciona la instancia devops-ec2

Puerto: 80

Paso 4: Crear el Application Load Balancer (devops-alb)
En EC2 → "Load Balancers" → "Create load balancer"

Selecciona "Application Load Balancer"

Configuración:

Load balancer name: devops-alb

Scheme: Internet-facing

Network mapping: Selecciona 2 zonas de disponibilidad

Security group: default

Listener: HTTP:80 → Forward to devops-tg

Paso 5: Ajustar el Security Group por defecto
Ve a EC2 → "Security Groups"

Selecciona el grupo "default"

Edit inbound rules → Add rule:

Type: HTTP | Port: 80 | Source: 0.0.0.0/0

✅ Verificación
Ve a EC2 → "Load Balancers"

Copia el DNS name del ALB

Abre tu navegador: http://<DNS-del-ALB>

Deberías ver: "Bienvenido a devops-ec2 - Servidor Nginx"

🎯 Beneficios de esta Arquitectura
Beneficio	Descripción
Alta Disponibilidad	El ALB distribuye tráfico entre instancias
Escalabilidad	Fácil agregar más instancias al Target Group
Balanceo de Carga	Distribuye carga entre instancias disponibles
Health Checks	Detecta y excluye instancias no saludables
Seguridad	Security Groups controlan acceso a nivel de red
Automatización	User Data automatiza la configuración inicial
🧹 Limpieza de Recursos
Para evitar costos, elimina los recursos en este orden:

Eliminar el ALB: EC2 → Load Balancers → devops-alb → Delete

Eliminar el Target Group: EC2 → Target Groups → devops-tg → Delete

Terminar la instancia: EC2 → Instances → devops-ec2 → Terminate

Eliminar Security Groups: EC2 → Security Groups → devops-sg → Delete

📚 Conclusión
Has configurado exitosamente una arquitectura en AWS que incluye:

✅ Una instancia EC2 con Nginx instalado automáticamente mediante User Data

✅ Un Application Load Balancer para distribuir el tráfico

✅ Un Target Group con health checks para monitorear la instancia

✅ Security Groups configurados correctamente

✅ Acceso al servidor web mediante el DNS del ALB

Esta configuración proporciona alta disponibilidad, escalabilidad y gestión eficiente del tráfico para aplicaciones web. 🚀
