# 🛡️ Análisis de Seguridad – Plataforma Escolar EduNova

## 🧩 1. Identificación de Vulnerabilidades

### 🔐 Almacenamiento en S3
- Buckets con acceso público habilitado.
- Archivos accesibles mediante URLs sin autenticación.
- Falta de cifrado en reposo y en tránsito.
- Estructura por carpetas sin control granular por institución/usuario.
- No se usan presigned URLs para compartir archivos.

### 🔑 IAM y Control de Acceso
- Políticas IAM excesivamente permisivas (`s3:*`).
- Roles sin rotación de credenciales.
- Ausencia de segregación de ambientes (prod/dev/test).
- No se aplica el principio de menor privilegio.

### 🧭 Monitoreo y Trazabilidad
- No se usa AWS CloudTrail para registrar acciones sobre S3.
- CloudWatch en modo básico sin alarmas relevantes.
- No se usa AWS Macie, Access Analyzer o GuardDuty.

---

## ⚠️ 2. Evaluación de Riesgos

| Vulnerabilidad                     | Riesgo                              | Impacto |
|-----------------------------------|-------------------------------------|---------|
| Acceso público a archivos en S3   | Robo de información personal        | Alto    |
| IAM excesivamente permisivo       | Acceso no autorizado                | Alto    |
| Falta de cifrado                  | Exposición de datos sensibles       | Alto    |
| No hay separación de ambientes    | Contaminación y errores humanos     | Medio   |
| Falta de logs y trazabilidad      | Incidentes no detectados a tiempo   | Alto    |

---

## 🛠️ 3. Mejora desde el Diseño

### Seguridad en S3
- Activar **Block Public Access** en todos los buckets.
- Implementar **presigned URLs** para acceso temporal.
- Cifrado obligatorio: **SSE-S3 o SSE-KMS**, + HTTPS.
- Aplicar políticas bucket granulares por institución.

### IAM y Control de Acceso
- Redefinir políticas con **principio de menor privilegio**.
- Eliminar `s3:*` y limitar permisos específicos (`s3:GetObject`, `s3:PutObject`).
- Automatizar rotación de credenciales IAM.
- Segregación de ambientes por cuentas o perfiles.

### Monitoreo y Seguridad
- Activar **CloudTrail** para auditar acciones sobre S3.
- Integrar **AWS Macie** para identificar datos sensibles.
- Usar **Access Analyzer** para revisar accesos externos.
- Configurar alarmas con **CloudWatch**.

---

## 🧰 4. Herramientas AWS Recomendadas

| Objetivo                     | Servicio AWS              |
|-----------------------------|---------------------------|
| Cifrado de datos            | Amazon S3 + SSE-KMS       |
| Acceso restringido          | IAM Policies, S3 Policies |
| Enlaces seguros             | Presigned URLs            |
| Auditoría de acciones       | AWS CloudTrail            |
| Detección de datos sensibles| AWS Macie                 |
| Revisión de accesos         | AWS Access Analyzer       |
| Análisis de amenazas        | AWS GuardDuty             |

---

## 🚨 5. Plan de Acción Priorizado

### Fase 1: Urgente (0–2 semanas)
1. Activar **Block Public Access** en todos los buckets S3.
2. Reemplazar `s3:*` por permisos mínimos necesarios.
3. Habilitar **AWS CloudTrail** para todos los buckets.
4. Aplicar **cifrado SSE-KMS**.
5. Segregar ambientes (prod/dev/test).
6. Usar presigned URLs en lugar de enlaces públicos.

### Fase 2: Evolutiva (1–3 meses)
1. Revisar y refinar políticas IAM.
2. Implementar AWS Macie + Access Analyzer.
3. Automatizar rotación de credenciales.
4. Crear dashboard de seguridad con CloudWatch.
5. Configurar expiración automática de archivos sensibles.
6. Aplicar revisiones trimestrales de seguridad.

---

## 📊 6. Diagrama de Arquitectura Segura (ASCII)

