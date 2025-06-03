# 🛡️ Análisis de Seguridad de Arquitectura AWS – Contoso.pe

## 📋 Presentación

Analizaremos una arquitectura de aplicaciones en la nube con servicios distribuidos y expuestos mediante subdominios. A pesar de tener protección perimetral básica, se detectan varios riesgos que comprometen la información y la disponibilidad del sistema.

---

## 🌐 Contexto

**Dominios productivos:**

- `https://prod-api.contoso.pe`: expone APIs públicas utilizadas por aplicaciones web, móviles y backoffice.
- `https://prod-bet.com.pe`: aloja servicios internos y buckets de datos en AWS.

Los entornos están separados pero interconectados.

---

## 👥 Equipo A – Análisis de Vulnerabilidades

### 🔹 Capa de Red
- ❌ Conexión al data center **sin NACL**.
- ❌ Subred pública con servicios internos expuestos.
- ❌ Falta de separación por entornos (dev/test/prod).

### 🔹 Capa de Aplicación
- ❌ Validación insegura por `X-Application` (fácil de falsificar)/ Realizar trafico para interceptar esa validación de header y ver los valores de headers.
- ❌ Protección insuficiente con solo WAF (SQLi y DDoS)/ Ataces de DDos y SQLI hacia el WAF para saber si existen las reglas de configuración.
- ❌ No se evidencia uso de autenticación/autorización real/ Atacar por phishing.

### 🔹 Capa de Autenticación
- ❌ Sin MFA, Cognito, JWT ni tokens de acceso válidos.
- ❌ No hay control granular de acceso.
- ❌ Un ataque de sniffing, o "intercepción de paquetes" para interceptar contraseñas o cambio de credenciales para poder acceder a la aplicación.

### 🔹 Capa de Almacenamiento
- ❌ **Bucket S3 con acceso público**.
- ❌ Base de datos sin mención de cifrado o control de acceso IAM.

---

### ✅ Mejoras inmediatas (sin rediseño completo)
1. Bloquear acceso público a S3 y registrar accesos.
2. Crear reglas de NACL para limitar tráfico entrante/saliente.
3. Aplicar autenticación basada en JWT (API Gateway + Cognito).
4. Ampliar reglas WAF (XSS, rate limiting, path filtering).
5. Activar CloudTrail, Config, y GuardDuty para auditoría.

---

## 🏗️ Equipo B – Rediseño de Arquitectura Segura

### 🔐 Principios aplicados: Defensa en profundidad

| Capa        | Recomendación |
|-------------|----------------|
| Red         | Subredes segmentadas (privadas/públicas), NACLs, SGs mínimos |
| Autenticación | Cognito con MFA, JWT, federación de identidad |
| Aplicación  | API Gateway con validación, roles IAM por servicio |
| Almacenamiento | S3 con políticas, cifrado KMS, acceso mínimo |
| Monitoreo   | GuardDuty, CloudTrail, Config, VPC Flow Logs |

---

### 🧱 Servicios recomendados de AWS

- **Amazon Cognito** – Autenticación/MFA
- **API Gateway** – Control de tráfico y autenticación
- **AWS WAF** – Protección OWASP capa 7
- **AWS KMS** – Cifrado de datos en reposo
- **IAM Roles y Policies** – Acceso mínimo y segmentado
- **Secrets Manager** – Gestión de credenciales
- **GuardDuty** – Detección de amenazas
- **CloudWatch / CloudTrail / Config** – Logs y monitoreo continuo

---

## ❓ Respuestas a Preguntas Guía

### 1. ¿Qué problemas específicos pueden derivarse del acceso público al bucket S3?
- Fuga de información sensible o confidencial
- Alojamiento de malware o phishing
- Ataques indirectos desde archivos maliciosos

### 2. ¿Cómo podría un atacante evadir la validación basada solo en headers?
- Spoofing con herramientas como Postman o curl
- Reutilización de headers legítimos
- Falta de firma, timestamps o tokens válidos

### 3. ¿Qué servicios de AWS podrías usar para reforzar esta arquitectura?
- Cognito, API Gateway, IAM, KMS, GuardDuty, Config, WAF, Secrets Manager

### 4. ¿Cómo podrías aplicar autenticación multifactor en este flujo?
- Cognito con MFA (TOTP o SMS)
- Federación de identidad con proveedores externos que implementen MFA
- Validación de JWT firmados antes de acceder a recursos

### 5. ¿Cuál sería el orden de prioridades para mitigar los riesgos detectados?
1. Bloquear acceso público a S3
2. Implementar autenticación robusta (Cognito/JWT)
3. Aislar subredes con NACL y SG
4. Configurar reglas WAF más completas
5. Implementar monitoreo continuo y alertas

---

## ✅ Conclusión

La arquitectura tiene elementos clave, pero requiere reforzar la seguridad en **todas las capas**. Aplicar defensa en profundidad, control de acceso estricto y monitoreo constante es fundamental para garantizar la protección de los servicios en producción.
