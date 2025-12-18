# 🏗️ Ticketero Infrastructure - AWS CDK

**Infraestructura como código para el sistema Ticketero usando AWS CDK con Java**

## 📋 Descripción

Este proyecto define toda la infraestructura AWS necesaria para ejecutar el sistema Ticketero en producción, incluyendo:

- **VPC** con subnets públicas y privadas
- **RDS PostgreSQL** para la base de datos
- **Amazon MQ RabbitMQ** para mensajería
- **ECS Fargate** para la aplicación
- **Application Load Balancer** para distribución de tráfico
- **Secrets Manager** para credenciales seguras
- **Security Groups** y configuraciones de red

## 🏛️ Arquitectura de Stacks

```
┌─────────────────────────────────────────────────────────────────┐
│                    ticketero-dev-network                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │     VPC     │  │   Subnets   │  │    Security Groups      │  │
│  │ 10.0.0.0/16 │  │ Pub + Priv  │  │  ALB, App, DB, MQ       │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
┌───────▼──────┐    ┌─────────▼──────┐    ┌────────▼─────────┐
│ ticketero-   │    │ ticketero-     │    │ ticketero-       │
│ dev-database │    │ dev-messaging  │    │ dev-notification │
│              │    │                │    │                  │
│ RDS Postgres │    │ Amazon MQ      │    │ Secrets Manager  │
│ + Secrets    │    │ RabbitMQ       │    │ (Telegram)       │
└──────────────┘    └────────────────┘    └──────────────────┘
                              │
                    ┌─────────▼──────────┐
                    │ ticketero-dev-     │
                    │ application        │
                    │                    │
                    │ ECS Fargate + ALB  │
                    │ Auto Scaling       │
                    └────────────────────┘
```

## 🚀 Despliegue

### Prerrequisitos

- **Java 21+**
- **Maven 3.9+**
- **Node.js 18+**
- **AWS CLI v2** configurado
- **Docker** (para build de imágenes)

### 1. Instalar AWS CDK

```bash
npm install -g aws-cdk@2.170.0
```

### 2. Configurar Variables de Entorno

```bash
export CDK_DEFAULT_ACCOUNT=$(aws sts get-caller-identity --query Account --output text)
export CDK_DEFAULT_REGION=us-east-1
```

### 3. Bootstrap CDK (Solo la primera vez)

```bash
cdk bootstrap aws://$CDK_DEFAULT_ACCOUNT/$CDK_DEFAULT_REGION
```

### 4. Validar Configuración

```bash
# Compilar el proyecto
mvn compile

# Sintetizar CloudFormation
cdk synth

# Ver diferencias
cdk diff
```

### 5. Desplegar Infraestructura

```bash
# Desplegar todos los stacks
cdk deploy --all --require-approval never
```

**⏱️ Tiempo estimado**: 15-20 minutos

## 📊 Stacks Desplegados

### 1. `ticketero-dev-network`
- **VPC**: 10.0.0.0/16
- **Subnets Públicas**: 2 AZs (10.0.1.0/24, 10.0.2.0/24)
- **Subnets Privadas**: 2 AZs (10.0.3.0/24, 10.0.4.0/24)
- **NAT Gateway**: Para acceso a internet desde subnets privadas
- **Security Groups**: ALB, App, Database, MQ

### 2. `ticketero-dev-database`
- **RDS PostgreSQL**: db.t3.micro, Multi-AZ
- **Secrets Manager**: Credenciales de base de datos
- **Subnet Group**: Subnets privadas
- **Backup**: 7 días de retención

### 3. `ticketero-dev-messaging`
- **Amazon MQ**: RabbitMQ mq.t3.micro
- **Secrets Manager**: Credenciales de RabbitMQ
- **Deployment Mode**: Single instance (dev)
- **Engine Version**: 3.13.x

### 4. `ticketero-dev-notification`
- **Secrets Manager**: Credenciales de Telegram Bot
- **Configuración**: Bot Token y Chat ID

### 5. `ticketero-dev-application`
- **ECS Cluster**: Fargate
- **ECS Service**: 1-3 tareas (Auto Scaling)
- **Application Load Balancer**: Puerto 80
- **Target Group**: Health checks en /actuator/health
- **Auto Scaling**: Basado en CPU (70%)

## 💰 Costos Estimados

| Recurso | Tipo | Costo Mensual (USD) |
|---------|------|--------------------|
| RDS PostgreSQL | db.t3.micro | ~$25-35 |
| Amazon MQ | mq.t3.micro | ~$40-50 |
| ECS Fargate | 0.25 vCPU, 0.5 GB | ~$15-25 |
| ALB | Standard | ~$20-25 |
| NAT Gateway | 1 instancia | ~$45-50 |
| **Total** | | **~$145-185** |

> **Nota**: Costos aproximados para ambiente de desarrollo en us-east-1

## 🔧 Comandos Útiles

```bash
# Listar todos los stacks
cdk list

# Ver template de un stack específico
cdk synth ticketero-dev-application

# Desplegar un stack específico
cdk deploy ticketero-dev-network

# Ver diferencias de un stack
cdk diff ticketero-dev-application

# Destruir toda la infraestructura
cdk destroy --all

# Ver outputs de los stacks
aws cloudformation describe-stacks --region us-east-1 --query 'Stacks[?starts_with(StackName, `ticketero-dev`)].Outputs'
```

## 🔗 Outputs Importantes

Después del despliegue, obtendrás:

```bash
# URL de la aplicación
ticketero-dev-application.ServiceUrl = http://ticketero-dev-alb-XXXXXXXXX.us-east-1.elb.amazonaws.com

# Endpoint de base de datos
ticketero-dev-database.DbEndpoint = ticketero-dev-postgres.XXXXXXXXX.us-east-1.rds.amazonaws.com

# ID del broker RabbitMQ
ticketero-dev-messaging.MqBrokerId = b-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX

# ARNs de secretos
ticketero-dev-database.DbSecretArn = arn:aws:secretsmanager:us-east-1:XXXXXXXXXXXX:secret:...
ticketero-dev-messaging.MqSecretArn = arn:aws:secretsmanager:us-east-1:XXXXXXXXXXXX:secret:...
ticketero-dev-notification.TelegramSecretArn = arn:aws:secretsmanager:us-east-1:XXXXXXXXXXXX:secret:...
```

## 🔒 Seguridad

### Security Groups
- **ALB**: Puerto 80 desde Internet
- **App**: Puerto 8080 solo desde ALB
- **Database**: Puerto 5432 solo desde App
- **MQ**: Puerto 5671 solo desde App

### Secrets Manager
- Credenciales de base de datos rotadas automáticamente
- Credenciales de RabbitMQ
- Token de Telegram Bot

### Network
- Base de datos y MQ en subnets privadas
- Aplicación en subnets privadas
- Solo ALB en subnets públicas

## 🧹 Limpieza

```bash
# Destruir toda la infraestructura
cdk destroy --all

# Confirmar eliminación cuando se solicite
```

**⚠️ Advertencia**: Esto eliminará todos los recursos y datos. Asegúrate de hacer backup si es necesario.

## 📚 Documentación Adicional

- [AWS CDK Developer Guide](https://docs.aws.amazon.com/cdk/v2/guide/)
- [AWS CDK API Reference](https://docs.aws.amazon.com/cdk/api/v2/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

## 🤝 Contribuir

Este proyecto es parte del programa de capacitación en DevOps con AWS CDK.

---

**🎯 ¿Listo para desplegar?** Sigue los pasos de la sección [Despliegue](#-despliegue) y tendrás tu infraestructura funcionando en AWS.