# AWS CloudWatch Agent Deployment para instancias Debian/Ubuntu

Este repositorio contiene dos mecanismos complementarios para la instalación, configuración y puesta en marcha del **Amazon CloudWatch Agent** en sistemas basados en **Debian**, permitiendo recopilar métricas y logs del sistema de manera centralizada.

## Estructura del Proyecto

```bash
.
├── Sin CloudWatch Instalado/
│   └── document.json        # Documento de Automation para ejecutar vía SSM
├── Sin CloudWatch ni SSM instalado/
│   └── userdata.sh          # Script de UserData para instancias EC2
└── README.md
```

---

## 📄 `Sin CloudWatch Instalado/document.json`

Este archivo es un documento de **AWS Systems Manager Automation (SSM)** que:

- Instala Amazon CloudWatch Agent en instancias Debian.
- Configura `rsyslog` para capturar los logs del sistema (`/var/log/syslog`).
- Genera dinámicamente la configuración del agente con métricas y logs personalizados.
- Utiliza parámetros configurables:
  - `logGroupPrefix`: Prefijo para los CloudWatch Log Groups (por defecto: `/aws/ec2`).
  - `metricsNamespace`: Namespace para las métricas en CloudWatch (por defecto: `CWAgent`).
  - `includeSystemLogs`: Indica si se deben incluir logs de sistema (`true` o `false`).

Este documento está diseñado para ejecutarse como un **Run Command** desde la consola de AWS o mediante AWS CLI.

---

## 🖥️ `Sin CloudWatch ni SSM instalado/userdata.sh`

Este script está diseñado para ser usado como **User Data** al lanzar una nueva instancia EC2.

Su objetivo es automatizar desde el arranque inicial la instalación y configuración del **CloudWatch Agent**, garantizando que las nuevas instancias comiencen a reportar métricas y logs a CloudWatch sin intervención manual.

---

## 🚀 Casos de uso

- **Automatización en producción**: Aplicar el documento SSM a múltiples instancias en caliente sin necesidad de reinicio.
- **Instancias nuevas**: Usar el `userdata.sh` en `Launch Templates`, `Auto Scaling Groups` o despliegues manuales.

---

## 🔒 Requisitos previos

- Permisos adecuados en IAM para ejecutar documentos de Systems Manager y acceso a CloudWatch.
- Acceso de red desde las instancias a los endpoints de CloudWatch y SSM (directamente o mediante VPC endpoints).

---

## 🧰 Referencias

- [Documentación oficial de CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)
- [Documentación oficial de AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/)

---

## 📬 Autor

**Francesc Castell**  
AWS Cloud Solutions Architect  
GitHub: [@FranckyCastell](https://github.com/FranckyCastelluario)

--- 