# EC2 UserData Bootstrap Script - SSM + CloudWatch Agent

## Descripción

Script de **User Data** para instancias EC2 que automatiza la instalación y configuración completa de **AWS Systems Manager (SSM) Agent** y **Amazon CloudWatch Agent** durante el primer arranque de la instancias DEBIAN.

## ¿Qué hace este script?

### Preparación del Sistema
- Actualiza todos los paquetes del sistema
- Instala dependencias críticas (curl, wget, rsyslog, etc.)

### Instalación de SSM Agent
- Descarga e instala la última versión del SSM Agent para Debian/Ubuntu
- Configura el servicio para iniciarse automáticamente

### Instalación de CloudWatch Agent
- Descarga e installa el CloudWatch Agent más reciente
- Configura recopilación automática de métricas del sistema
- Establece envío de logs del sistema a CloudWatch Logs
- Convierte datos binarios del sistema a formato texto plano legible

### Configuración Automática de Monitoreo

#### Logs Recopilados
- **System Logs** (`/var/log/syslog`): Logs generales del sistema

#### Métricas Recopiladas (cada 60 segundos)
- **CPU**: Utilización por estado (idle, iowait, user, system)
- **Memoria**: Porcentaje de uso de RAM
- **Disco**: Porcentaje de uso y tiempo de I/O
- **Red**: Conexiones TCP activas y en espera
- **Swap**: Porcentaje de uso de memoria de intercambio

## Estructura de Salida en CloudWatch

### CloudWatch Logs
```
/aws/ec2/{instance-id}/
├── syslog          # Logs del sistema
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

## Uso con Terraform

### Configuración Básica
```hcl
resource "aws_instance" "monitored_instance" {
  ami           = "ami-0c7217cdde317cfec"  # Ubuntu 22.04 LTS
  instance_type = "t3.micro"
  
  # IAM role con permisos necesarios
  iam_instance_profile = aws_iam_instance_profile.ec2_monitoring_profile.name
  
  # User data script
  user_data = file("${path.module}/userdata.sh")
  
  tags = {
    Name        = "monitored-instance"
    Environment = "production"
    Monitoring  = "enabled"
  }
}
```

### IAM Role Requerido
```hcl
resource "aws_iam_role" "ec2_monitoring_role" {
  name = "EC2-Monitoring-Role"
  
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ec2.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "ssm_managed_instance" {
  role       = aws_iam_role.ec2_monitoring_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
}

resource "aws_iam_role_policy_attachment" "cloudwatch_agent" {
  role       = aws_iam_role.ec2_monitoring_role.name
  policy_arn = "arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy"
}

resource "aws_iam_instance_profile" "ec2_monitoring_profile" {
  name = "EC2-Monitoring-Profile"
  role = aws_iam_role.ec2_monitoring_role.name
}
```

## Verificación Post-Instalación

### Verificar Servicios
```bash
# SSM Agent
sudo systemctl status amazon-ssm-agent

# CloudWatch Agent
sudo systemctl status amazon-cloudwatch-agent