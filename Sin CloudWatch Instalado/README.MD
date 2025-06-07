# CloudWatch Agent SSM Document

## Descripción

Este documento de AWS Systems Manager (SSM) automatiza la instalación y configuración del **Amazon CloudWatch Agent** en instancias EC2 con sistemas operativos basados en Debian/Ubuntu.

## ¿Qué hace este documento?

### Recopilación de Métricas del Sistema
El agente recopila las siguientes métricas cada 60 segundos:

- **CPU**: Uso de CPU (idle, iowait, user, system)
- **Memoria**: Porcentaje de memoria utilizada
- **Disco**: Porcentaje de uso de disco e I/O time
- **Red**: Conexiones TCP establecidas y en espera
- **Swap**: Porcentaje de uso de memoria swap

### Recopilación de Logs del Sistema
- Configura rsyslog para generar logs del sistema
- Convierte logs binarios del sistema (syslog) a formato texto plano
- Envía los logs a CloudWatch Logs con estructura organizada por instancia

### Características Técnicas

#### Gestión Robusta de Permisos
- Verificación automática de usuarios del sistema
- Manejo de diferencias entre distribuciones Ubuntu/Debian
- Fallback automático a configuraciones seguras

#### Logging Estructurado
- Logs organizados por Instance ID: `/aws/ec2/{instance-id}/syslog`
- Métricas categorizadas en namespace personalizable
- Timestamps UTC para consistencia global

## Parámetros Configurables

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `logGroupPrefix` | String | `/aws/ec2` | Prefijo para los grupos de logs en CloudWatch |
| `metricsNamespace` | String | `CWAgent` | Namespace para las métricas en CloudWatch |
| `includeSystemLogs` | String | `true` | Incluir logs del sistema (syslog) |

## Estructura de Salida

### CloudWatch Logs
```
/aws/ec2/
├── i-0123456789abcdef0/
│   └── syslog
├── i-fedcba9876543210/
│   └── syslog
```

### CloudWatch Metrics
```
CWAgent/
├── CPU/
├── Memory/
├── Disk/
├── Network/
└── Swap/
```

## Requisitos

### Permisos IAM
La instancia EC2 debe tener un rol con las siguientes políticas:
- `CloudWatchAgentServerPolicy`
- Permisos para crear Log Groups y Log Streams

### Sistema Operativo Orientado
- Debian 9 (Stretch) o superior

### Conectividad
- Acceso a internet para descargar el agente
- Acceso a los endpoints de CloudWatch
- SSM Agent instalado y configurado

## Uso con Terraform

```hcl
resource "aws_ssm_document" "cloudwatch_agent" {
  name          = "install-cloudwatch-agent"
  document_type = "Command"
  document_format = "JSON"
  
  content = file("${path.module}/cloudwatch-agent-document.json")
  
  tags = {
    Environment = "production"
    Purpose     = "monitoring"
  }
}

resource "aws_ssm_association" "cloudwatch_agent" {
  name = aws_ssm_document.cloudwatch_agent.name
  
  targets {
    key    = "tag:Environment"
    values = ["production"]
  }
  
  parameters = {
    logGroupPrefix    = "/aws/ec2/production"
    metricsNamespace  = "Production/CWAgent"
    includeSystemLogs = "true"
  }
}
```