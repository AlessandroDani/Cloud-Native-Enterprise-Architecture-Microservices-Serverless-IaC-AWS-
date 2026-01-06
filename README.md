# Cloud-Native AWS Ecosystem: Microservices, Serverless & IaC 🚀

Este proyecto representa una arquitectura completa desplegada en **AWS**, integrando microservicios en contenedores, funciones serverless políglotas y una sólida base de **Infraestructura como Código (IaC)**. El objetivo principal fue construir un sistema escalable, seguro y altamente disponible. Este proyecto forma parte del reto 2 de Pragma

## 🔗 Repositorios del Proyecto
* **Backend Java (Spring Boot 3):** [https://github.com/AlessandroDani/person-deploy]
* **Backend JavaScript (Node.js):** [https://github.com/AlessandroDani/person-serverless]

## 🏗️ Arquitectura General
El ecosistema se divide en dos grandes bloques gestionados por un único punto de entrada: **Amazon API Gateway**.

1.  **Cómputo en Contenedores:** API de gestión de personas desplegada en **Amazon ECS (Fargate)** con persistencia en **RDS**.
2.  **Ecosistema Serverless:** CRUD de usuarios utilizando **AWS Lambda (Node.js & Java)** con persistencia en **DynamoDB** y arquitectura dirigida por eventos (SQS/SNS).

## 📦 1. Microservicios y Contenedores (ECS & Docker)

Se implementó un flujo de empaquetado y despliegue continuo utilizando **Amazon ECR** como registro de imágenes.

### Dockerización
```dockerfile
FROM public.ecr.aws/docker/library/openjdk:17-jdk-slim
# Copia del artefacto generado por Gradle
COPY build/libs/deploy-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```
### 🚀 Procedimiento de Despliegue (ECS)
Estos son los comandos manuales para actualizar el microservicio en ECR:

1. **Login en ECR:** Autentica Docker con tu registro de AWS.
```bash
aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin [ACCOUNT_ID].dkr.ecr.us-east-2.amazonaws.com
```

2. **Build de la imagen:** Construye la imagen localmente.
```bash
docker build -t pragma-api-alessandro .
```

3. **Tagging:** Etiqueta la imagen para vincularla al repositorio remoto.
```bash
docker tag pragma-api-alessandro:latest [ACCOUNT_ID][.dkr.ecr.us-east-2.amazonaws.com/pragma-api-alessandro:latest](https://.dkr.ecr.us-east-2.amazonaws.com/pragma-api-alessandro:latest)
```

4. **Push:** Sube la imagen a Amazon ECR.
```
docker push [ACCOUNT_ID][.dkr.ecr.us-east-2.amazonaws.com/pragma-api-alessandro:
```

## ⚡ 2. Ecosistema Serverless (Poliglotismo)

Implementación de una arquitectura **Serverless Políglota** utilizando el **Serverless Framework** (IaC) para la orquestación.

- **Node.js 20:** Endpoints ligeros para operaciones GET y POST.
- **Java 17 (Spring Boot 3):** Endpoints robustos para PUT y DELETE, utilizando el adaptador `aws-serverless-java-container`.
- **Persistencia NoSQL:** Aprovisionamiento de **DynamoDB** con escalado automático (**Pay-Per-Request**).

## 🔐 3. Seguridad y Gestión de Configuración

### Autenticación con AWS Cognito

Rutas protegidas mediante **JWT Authorizers** en API Gateway.

- **User Pools:** Configurados con flujo `USER_PASSWORD_AUTH`.
- **Seguridad:** Los endpoints no son accesibles sin un token Bearer válido.

### Gestión de Secretos (Zero Hardcoding)

Se eliminaron todas las credenciales del código fuente:

- **AWS Secrets Manager:** Almacena la contraseña de la base de datos RDS.
- **SSM Parameter Store:** Almacena la URL de conexión (`db_url`).
- **Inyección en Runtime:** ECS inyecta estos valores como variables de entorno al iniciar el contenedor.
## 📡 4. Mensajería y Observabilidad

### Arquitectura Dirigida por Eventos (EDA)

- **Amazon SQS:** Cola para desacoplar el registro de usuarios. Garantiza que el sistema responda rápido aunque el proceso de notificación sea lento.
- **Amazon SNS:** Sistema Pub/Sub que dispara el envío de correos electrónicos al procesar mensajes de la cola SQS.

### Monitoreo (CloudWatch)

- **Metric Filters:** Se configuraron filtros para contar ocurrencias de errores `5XX` (Server Error) y `4XX` (Auth Error).
- **Alarmas:** Si los errores superan el umbral definido, CloudWatch dispara una alerta a SNS para notificar al equipo de ingeniería.

## 📜 6. CloudFormation (IaC Nativo)

Desarrollo de templates en **YAML** puro para demostrar el dominio de funciones intrínsecas sin frameworks externos:

- **`!Ref`**: Referencia dinámica a recursos (ej. ID de Tabla DynamoDB).
- **`!GetAtt`**: Extracción de atributos complejos (ej. ARN de un IAM Role).
- **`!Sub`**: Construcción de cadenas dinámicas para garantizar portabilidad entre regiones (`arn:aws:apigateway:${AWS::Region}:...`).
### 🚀 Comandos Adicionales

- **Compilar Java (Fat JAR):** `./gradlew clean shadowJar`
- **Desplegar Stack Serverless:** `npx serverless deploy`
- **Eliminar Stack Serverless:** `npx serverless remove`
