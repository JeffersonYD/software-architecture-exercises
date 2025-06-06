# 🛡️ Caso de Estudio: Cifrado Fuerte, Configuración Débil

## 🧩 1. Contexto

Una startup ha lanzado su MVP en AWS con la siguiente arquitectura:

- **API Gateway** expone `/catalog` protegido solo por una **API Key**.
- **AWS Lambda** consulta productos en **DynamoDB** y envía correos vía **SES**.
- **IAM Role** con políticas `AmazonDynamoDBFullAccess` y `AmazonSESFullAccess`.
- **CloudTrail** activo solo en `us-east-1`.
- Sin **WAF** ni controles de tráfico.

---

## 🔍 2. Análisis de vulnerabilidades

### 1. Protección débil de la API
- API Key fácilmente filtrable, sin autenticación por usuario ni control de acceso.

### 2. Permisos excesivos en IAM
- Lambda con permisos totales a DynamoDB y SES → Riesgo alto si se compromete.

### 3. CloudTrail mal configurado
- Solo monitorea una región → Actividad maliciosa fuera de `us-east-1` no se detecta.

### 4. Sin protección contra tráfico anómalo
- Exposición total a DDoS, scraping y ataques de inyección.

---

## ✅ 3. Rediseño seguro (manteniendo simplicidad)

### 🔐 Autenticación de API

| Elemento        | Cambio Propuesto |
|------------------|------------------|
| API Gateway      | Usar **Amazon Cognito** o **Lambda Authorizer** con tokens JWT |
| Sesiones         | Flujo de login seguro con expiración y contexto de usuario |

---

### 🔒 Principio de menor privilegio

| Recurso    | Cambio Propuesto |
|------------|------------------|
| IAM Role   | Crear política personalizada: solo `GetItem`, `Query`, `Scan` en una tabla específica |
| SES        | Permitir solo `SendEmail` a destinos autorizados |

---

### 📈 Mejorar trazabilidad

| Elemento     | Cambio Propuesto |
|--------------|------------------|
| CloudTrail   | Activar en **todas las regiones** y enviar logs a **S3 cifrado** |
| Monitoreo    | Integrar con **CloudWatch Logs** y **SNS** para alertas |

---

### 🛡️ Protección contra tráfico anómalo

| Elemento    | Cambio Propuesto |
|-------------|------------------|
| WAF         | Implementar en API Gateway |
| Reglas WAF  | - Rate limiting por IP<br>- Protección contra bots<br>- Validación de input<br>- Bloqueo de patrones maliciosos |

---

## 🧠 Preguntas guía

**¿Qué pasa si se filtra la API Key?**  
➡️ Acceso irrestricto a la API sin forma de identificar o limitar al atacante.

**¿Por qué evitar FullAccess?**  
➡️ Permite a un atacante comprometer completamente la base de datos y servicios de correo.

**¿Qué limita CloudTrail regional?**  
➡️ No se audita actividad fuera de `us-east-1`, generando puntos ciegos.

**¿Ventajas de Cognito o Lambda Authorizer?**  
➡️ Autenticación robusta basada en usuarios, tokens expirables y control granular.

**¿Qué puede hacer AWS WAF?**  
➡️ Mitigar DDoS, limitar peticiones, bloquear tráfico malicioso y proteger contra inyecciones.

---

## 🔄 4. Diagrama Comparativo

| ❌ Arquitectura Insegura | ✅ Arquitectura Segura |
|--------------------------|------------------------|
| API Key en API Gateway | Cognito + WAF en API Gateway |
| IAM FullAccess para Lambda | IAM con permisos mínimos |
| CloudTrail solo en us-east-1 | CloudTrail en todas las regiones |
| Sin control de tráfico | WAF con reglas específicas |
| Sin autenticación de usuario | JWT + control por usuario |

---

## 📌 5. Conclusión

Este caso demuestra que incluso usando servicios seguros (DynamoDB, SES, API Gateway), una **configuración débil** puede comprometer todo el sistema. Aplicar **autenticación fuerte**, **principio de menor privilegio**, y **observabilidad global** es esencial para proteger MVPs desde el inicio.