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
