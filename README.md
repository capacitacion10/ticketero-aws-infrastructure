# 🎫 Ticketero AWS Infrastructure

**Sistema completo de gestión de tickets bancarios con infraestructura AWS automatizada**

[![Java](https://img.shields.io/badge/Java-21-orange.svg)](https://openjdk.java.net/projects/jdk/21/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.2-green.svg)](https://spring.io/projects/spring-boot)
[![AWS CDK](https://img.shields.io/badge/AWS%20CDK-2.170.0-yellow.svg)](https://aws.amazon.com/cdk/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue.svg)](https://www.postgresql.org/)
[![RabbitMQ](https://img.shields.io/badge/RabbitMQ-3.13-orange.svg)](https://www.rabbitmq.com/)

## 🏗️ Arquitectura del Sistema

```
┌──────────────────────────────────────────────────────────────────┐
│                        AWS CLOUD                               │
├──────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐        │
│  │     ALB     │───▶│ ECS Fargate │───▶│ RDS Postgres│        │
│  │   (Port 80) │    │  (Java App) │    │   (Port 5432)│        │
│  └─────────────┘    └─────────────┘    └─────────────┘        │
│         │                   │                                  │
│         │            ┌─────────────┐    ┌─────────────┐        │
│         │            │  Amazon MQ  │    │   Secrets   │        │
│         │            │  (RabbitMQ) │    │  Manager    │        │
│         │            └─────────────┘    └─────────────┘        │
│         │                                                      │
│  ┌───────────────────────────────────────────────────────────┐   │
│  │                    VPC Network                          │   │
│  │  ┌─────────────┐              ┌─────────────┐          │   │
│  │  │   Public    │              │   Private   │          │   │
│  │  │   Subnets   │              │   Subnets   │          │   │
│  │  └─────────────┘              └─────────────┘          │   │
│  └───────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────────┐
                    │  Telegram   │
                    │    Bot      │
                    └─────────────┘
```

## 📁 Estructura del Proyecto

```
ticketero-aws-infrastructure/
├── 📱 ticketero/                    # Aplicación Java Spring Boot
│   ├── src/main/java/               # Código fuente Java
│   ├── src/main/resources/          # Configuraciones y migraciones
│   ├── docker-compose.yml           # Entorno local de desarrollo
│   ├── Dockerfile                   # Imagen Docker optimizada
│   └── pom.xml                      # Dependencias Maven
├── 🏗️ ticketero-infra/              # Infraestructura AWS CDK
│   ├── src/main/java/               # Definiciones CDK en Java
│   ├── cdk.json                     # Configuración CDK
│   └── pom.xml                      # Dependencias CDK
├── 🔄 .github/workflows/            # CI/CD Pipeline
└── 📋 docs/                         # Documentación
```

## 🚀 Despliegue Rápido

### Prerrequisitos
- Java 21+
- Maven 3.9+
- Node.js 18+
- AWS CLI v2
- Docker

### 1️⃣ Configurar AWS
```bash
aws configure
# Configurar credenciales y región us-east-1
```

### 2️⃣ Instalar CDK
```bash
npm install -g aws-cdk@2.170.0
```

### 3️⃣ Bootstrap CDK
```bash
cd ticketero-infra
export CDK_DEFAULT_ACCOUNT=$(aws sts get-caller-identity --query Account --output text)
export CDK_DEFAULT_REGION=us-east-1
cdk bootstrap aws://$CDK_DEFAULT_ACCOUNT/$CDK_DEFAULT_REGION
```

### 4️⃣ Desplegar Infraestructura
```bash
cdk deploy --all --require-approval never
```

**⏱️ Tiempo estimado**: 15-20 minutos

## 💰 Costos Estimados (Ambiente Dev)

| Servicio | Costo Mensual (USD) |
|----------|---------------------|
| RDS PostgreSQL (db.t3.micro) | ~$25-35 |
| Amazon MQ RabbitMQ (mq.t3.micro) | ~$40-50 |
| ECS Fargate + ALB | ~$25-35 |
| **Total Estimado** | **~$90-120** |

## 🔧 Componentes Principales

### 📱 Aplicación Java (ticketero/)
- **Framework**: Spring Boot 3.2 + Java 21
- **Base de Datos**: PostgreSQL con Flyway migrations
- **Mensajería**: RabbitMQ con patrón Outbox
- **Notificaciones**: Telegram Bot integrado
- **Monitoreo**: Actuator + Prometheus metrics
- **Testing**: TestContainers + JUnit 5

### 🏗️ Infraestructura CDK (ticketero-infra/)
- **Red**: VPC con subnets públicas/privadas
- **Compute**: ECS Fargate con Auto Scaling
- **Base de Datos**: RDS PostgreSQL Multi-AZ
- **Mensajería**: Amazon MQ RabbitMQ
- **Load Balancer**: Application Load Balancer
- **Seguridad**: Security Groups + Secrets Manager

## 🔗 URLs Post-Despliegue

Después del despliegue exitoso, obtendrás:

```bash
# URL principal de la aplicación
http://ticketero-dev-alb-XXXXXXXXX.us-east-1.elb.amazonaws.com

# Endpoints de la API
GET  /api/tickets/{uuid}           # Consultar ticket
POST /api/tickets                  # Crear ticket
GET  /api/admin/dashboard          # Dashboard admin

# Health checks
GET  /actuator/health              # Estado de la aplicación
GET  /actuator/prometheus          # Métricas Prometheus
```

## 🛠️ Desarrollo Local

### Ejecutar con Docker Compose
```bash
cd ticketero
cp .env.example .env
# Editar .env con credenciales de Telegram
docker compose up -d
```

### Ejecutar nativamente
```bash
cd ticketero
./mvnw spring-boot:run
```

## 📊 Monitoreo y Observabilidad

- **Health Checks**: `/actuator/health`
- **Métricas**: `/actuator/prometheus`
- **Logs**: CloudWatch Logs
- **RabbitMQ Management**: Puerto 15672 (local)

## 🔒 Seguridad

- ✅ Credenciales en AWS Secrets Manager
- ✅ Security Groups restrictivos
- ✅ Subnets privadas para base de datos
- ✅ Usuario no-root en contenedores
- ✅ HTTPS ready (certificado SSL opcional)

## 🧹 Limpieza

```bash
# Destruir toda la infraestructura
cd ticketero-infra
cdk destroy --all
```

## 📚 Documentación Adicional

- [📱 Aplicación Java](./ticketero/README.md)
- [🏗️ Infraestructura CDK](./ticketero-infra/README.md)
- [🏛️ Arquitectura](./ticketero/docs/ARCHITECTURE.md)
- [🚀 Despliegue](./ticketero/docs/DEPLOYMENT.md)

## 🤝 Contribuir

Este proyecto es parte del programa de capacitación en desarrollo Java y DevOps con AWS.

## 📄 Licencia

Proyecto educativo - Capacitación Java Developer 3.0

---

**🎯 ¿Listo para desplegar?** Sigue los pasos de [Despliegue Rápido](#-despliegue-rápido) y tendrás tu sistema funcionando en AWS en menos de 20 minutos.