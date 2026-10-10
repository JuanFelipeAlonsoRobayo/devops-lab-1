# Lab 01: CI/CD con GitHub Actions y Amazon ECS

> Proyecto para automatizar el ciclo de integración y despliegue continuo (CI/CD) de una aplicación contenerizada escrita en Python con FastAPI hacia AWS ECS (Elastic Container Service)[cite: 1].

## 📖 Arquitectura y Pipeline CI/CD

El pipeline de GitHub Actions se compone de cuatro etapas principales para garantizar la calidad y el despliegue de la infraestructura:

*   **Linter:** Análisis estático de código utilizando Pylint[cite: 1].
*   **Pruebas Unitarias:** Ejecución de pruebas automatizadas mediante Pytest[cite: 1].
*   **Construcción y Publicación de Contenedor:** Empaquetado de la imagen Docker y publicación en Amazon ECR (Elastic Container Registry)[cite: 1].
*   **Despliegue a AWS ECS:** Despliegue automatizado e idempotente sobre un clúster ECS en instancias EC2 `t3.micro` administradas por un Auto Scaling Group[cite: 1].

## ⚙️ Configuración de Entorno y Secretos

Para que el pipeline de GitHub Actions pueda autenticarse y provisionar la infraestructura de forma segura en AWS, es necesario configurar las siguientes credenciales en la ruta `Settings > Secrets and variables > Actions` del repositorio en GitHub[cite: 4]:

**Repository Secrets:**
*   `AWS_ACCESS_KEY_ID`: ID de la llave de acceso de AWS entregado para el laboratorio[cite: 4].
*   `AWS_SECRET_ACCESS_KEY`: Llave de acceso secreta de AWS entregada para el laboratorio[cite: 4].

**Repository Variables:**
*   `STUDENT_SUFFIX`: Identificador o nombre de usuario de IAM asignado para aislar recursos y evitar colisiones[cite: 4].
*   `AWS_REGION`: Región de despliegue, por defecto `us-east-1`[cite: 4].

## 💻 Aplicación FastAPI

La API REST está construida con FastAPI, empaquetada en una imagen basada en `python:3.12-slim` y configurada para ejecutarse con Uvicorn en el puerto 8000[cite: 5]. Gestiona sus dependencias a través de Poetry[cite: 5].

Expone los siguientes endpoints:
*   `GET /`: Retorna un saludo personalizado recibiendo `first_name` y `last_name` a través de query parameters[cite: 5].
*   `GET /health`: Endpoint de monitoreo de estado para el orquestador de contenedores[cite: 5].

## 🚀 Despliegue y Validación

1.  La subida de cambios (push) a la rama `main` disparará automáticamente el flujo de trabajo de CI/CD definido en `.github/workflows/ci.yml`[cite: 6].
2.  Tras el despliegue, la imagen del contenedor quedará alojada en el repositorio de Amazon ECR bajo el nombre `greeting-<STUDENT_SUFFIX>`[cite: 9].
3.  El servicio orquestado en Amazon ECS, denominado `service-greeting-<STUDENT_SUFFIX>`, pasará a estado *ACTIVE* con 1 tarea en ejecución (*RUNNING*)[cite: 10].
4.  La validación del funcionamiento puede realizarse copiando la IP pública de la instancia EC2 desde los logs de GitHub Actions y ejecutando la siguiente petición HTTP[cite: 10]:
    ```bash
    curl "http://<EC2_PUBLIC_IP>:8000/?first_name=TuNombre&last_name=TuApellido"
    ```

## 🧹 Limpieza de Recursos (Teardown)

Para evitar el consumo de créditos en la cuenta compartida de AWS una vez registrado el laboratorio, la infraestructura debe ser eliminada[cite: 12]:

1.  Diríjase a la pestaña de **Actions** en su repositorio y seleccione el flujo de trabajo manual **Cleanup Student Infrastructure**[cite: 12].
2.  Haga clic en **Run workflow**, verifique que su `STUDENT_SUFFIX` sea el correcto y ejecute el pipeline[cite: 12].
3.  El proceso invocará el script `destroy.sh`, el cual se encargará de destruir todos los recursos asociados a su usuario, incluyendo el servicio ECS, Auto Scaling Group, instancia EC2, clúster, launch template, grupo de seguridad y repositorio ECR[cite: 12].