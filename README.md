# devsecops-pipeline-demo

Demo repository for a Jenkins CI/CD pipeline that performs SAST, SCA, image scanning, DAST and deploys to staging (docker-compose). This repo includes a deliberately vulnerable Node.js app to be used in labs.

## Contents
- `Jenkinsfile` - declarative pipeline
- `src/` - vulnerable Node.js app
- `Dockerfile` - builds app image
- `docker-compose.yml` - staging deployment
- `scripts/` - helper scripts to run Semgrep, Dependency-Check, Trivy, ZAP

## Prerequisites
- Jenkins server (with Docker-enabled agents) or machine with Docker.
- Docker installed and user allowed to run docker commands.

## Quick local run
1. Build and run app locally:
   ```bash
   docker-compose up --build -d
   ```
2. Visit http://localhost:3000
3. Run semgrep scan:
   ```bash
   ./scripts/run_semgrep.sh
   ```
4. Run dependency-check:
   ```bash
   ./scripts/run_dependency_check.sh
   ```
5. Build image and run trivy:
   ```bash
   docker build -t devsecops-labs-app:local .
   ./scripts/run_trivy.sh devsecops-labs-app:local
   ```
6. Run ZAP scan:
   ```bash
   ./scripts/run_zap.sh http://localhost:3000
   ```

## Notes
- This repository is intentionally insecure — **do not** run in production.
- To enforce gating policies, extend the Jenkinsfile to parse results and fail builds when thresholds are exceeded.

---

---

## 🧪 ¿Cuál es el objetivo de este laboratorio?

Este laboratorio te permite practicar cómo aplicar el enfoque **DevSecOps** en un pipeline real de CI/CD, **automatizando pruebas de seguridad** desde el código fuente hasta la ejecución en un entorno *staging*.

---

## 🧰 Herramientas y tipos de análisis utilizados

| Tipo de análisis                        | Herramienta usada  | Qué analiza                                                                   |
| --------------------------------------- | ------------------ | ----------------------------------------------------------------------------- |
| **SAST** (Static App Security Testing)  | `Semgrep`          | Código fuente (JavaScript/Node.js) en busca de vulnerabilidades               |
| **SCA** (Software Composition Analysis) | `Dependency-Check` | Dependencias y bibliotecas del proyecto (`package.json`)                      |
| **Image Scanning**                      | `Trivy`            | Imagen Docker, en busca de vulnerabilidades del sistema operativo o librerías |
| **DAST** (Dynamic App Security Testing) | `OWASP ZAP`        | La aplicación ya desplegada, como un atacante lo haría (escaneo externo)      |

---

## 📁 Estructura del proyecto

| Carpeta / Archivo    | Descripción                                                   |
| -------------------- | ------------------------------------------------------------- |
| `Jenkinsfile`        | Pipeline declarativo que automatiza todos los análisis        |
| `src/`               | Código fuente de la app vulnerable (Node.js)                  |
| `Dockerfile`         | Define cómo se construye la imagen de la app                  |
| `docker-compose.yml` | Lanza la app localmente para pruebas DAST o visualización     |
| `scripts/`           | Scripts para ejecutar cada herramienta (Semgrep, Trivy, etc.) |

---

## 🚀 ¿Qué se hace en este laboratorio?

### 🔧 **1. Prerrequisitos**

Necesitas:

* Un Jenkins (local o remoto) con agentes Docker.
* Docker instalado en tu máquina (y permisos para ejecutarlo).

---

### 🖥️ **2. Ejecución local rápida (modo standalone)**

#### Paso 1: Construir y levantar la aplicación

```bash
docker-compose up --build -d
```

➡️ Lanza la app en `http://localhost:3000`
*Está vulnerable a propósito para las pruebas.*

---

#### Paso 2: Ejecutar los análisis

| Herramienta                | Comando                                           | Qué hace                                                                       |
| -------------------------- | ------------------------------------------------- | ------------------------------------------------------------------------------ |
| **Semgrep** (SAST)         | `./scripts/run_semgrep.sh`                        | Escanea el código fuente (`src/`) y detecta malas prácticas o vulnerabilidades |
| **Dependency-Check** (SCA) | `./scripts/run_dependency_check.sh`               | Busca CVEs en las dependencias (`package.json`)                                |
| **Trivy** (Image Scan)     | `./scripts/run_trivy.sh devsecops-labs-app:local` | Escanea la imagen Docker ya construida                                         |
| **ZAP** (DAST)             | `./scripts/run_zap.sh http://localhost:3000`      | Hace un escaneo dinámico desde el exterior                                     |

---

## 🔄 ¿Y en Jenkins?

### El `Jenkinsfile` automatiza todo eso:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'docker build -t devsecops-labs-app .'
            }
        }
        stage('SAST - Semgrep') {
            steps {
                sh './scripts/run_semgrep.sh'
            }
        }
        stage('SCA - Dependency Check') {
            steps {
                sh './scripts/run_dependency_check.sh'
            }
        }
        stage('Image Scan - Trivy') {
            steps {
                sh './scripts/run_trivy.sh devsecops-labs-app'
            }
        }
        stage('Deploy to Staging') {
            steps {
                sh 'docker-compose up -d'
            }
        }
        stage('DAST - OWASP ZAP') {
            steps {
                sh './scripts/run_zap.sh http://localhost:3000'
            }
        }
    }
}
```

> 🔐 Este pipeline es un ejemplo completo de cómo **automatizar la seguridad** en todas las fases del ciclo de vida de una aplicación.

---

## ⚠️ Notas importantes

* **La app es insegura a propósito**, así que **NO la uses en producción.**
* Puedes modificar el `Jenkinsfile` para:

  * Fallar el build si una herramienta encuentra vulnerabilidades críticas.
  * Agregar reportes (JSON, HTML).
  * Integrarlo con Slack, JIRA, etc.

---

## 🧠 ¿Qué deberías aprender de este lab?

1. **Cómo aplicar DevSecOps en un pipeline real.**
2. **Qué herramientas usar en cada etapa (SAST, SCA, etc.).**
3. **Cómo automatizar escaneos de seguridad.**
4. **Cómo interpretar los resultados.**
5. **Cómo hacer “shift-left” de seguridad: detectar problemas antes del despliegue.**

---


