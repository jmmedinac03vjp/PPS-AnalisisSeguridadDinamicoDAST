# Actividad 2: Análisis de Seguridad Dinámico con DAST (Dynamic Application Security Testing)

## 🎯 Objetivo

Usar DAST para detectar vulnerabilidades en una aplicación en ejecución.

---

## 🔍 ¿Qué es DAST?

**Dynamic Application Security Testing (DAST)** analiza una aplicación en ejecución para identificar vulnerabilidades explotables, como:
- Inyecciones SQL
- Configuraciones inseguras
- Exposición de información sensible

No requiere acceso al código fuente.

---

## 🛠️ Herramienta: Nikto

Nikto es un escáner open-source para aplicaciones web.

### Instalación

```bash
sudo apt install nikto
nikto -Version
```

---

## 🧪 Aplicaciones vulnerables para pruebas

### 1. OWASP Juice Shop
```bash
git clone https://github.com/juice-shop/juice-shop.git
cd juice-shop
npm install
npm start
```
Disponible en `http://localhost:3000`.

### 2. Damn Vulnerable Web Application (DVWA)
```bash
docker run --rm -it -p 3000:80 vulnerables/web-dvwa
```

### 3. OWASP WebGoat
```bash
docker run --rm -it -p 3000:8080 webgoat/webgoat-8.0
```

---

## 🔍 Escaneos con Nikto

### Escaneo básico
```bash
nikto -h http://localhost:3000
```

### Escaneo detallado
```bash
nikto -h http://localhost:3000 -Tuning 123bde
```

### Escaneo agresivo
```bash
nikto -h http://localhost:3000 -C all
```

### Autenticación
```bash
nikto -h http://localhost:3000 -id usuario:contraseña
```

### Cambiar User-Agent
```bash
nikto -h http://localhost:3000 -useragent "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
```

### Guardar resultados
```bash
nikto -h http://localhost:3000 -o resultado_scan.html -Format html
```

### Múltiples objetivos
```bash
nikto -h lista_de_objetivos.txt
```

### Bypass de firewalls
```bash
nikto -h http://localhost:3000 -useragent "Mozilla..." -useproxy http://127.0.0.1:8080
```

---

## ✅ Mitigación y Buenas Prácticas

### Cabeceras de seguridad (Apache)

```apacheconf
Header always set X-Frame-Options "DENY"
Header always set X-XSS-Protection "1; mode=block"
Header always set Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline';"
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
```

### Cookies seguras en PHP
```php
session_set_cookie_params([
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Strict'
]);
```

### Ocultar versión del servidor (Apache)
```apacheconf
ServerTokens Prod
ServerSignature Off
```

---

## 🔁 Automatización con Nikto

Ejemplo de cronjob:
```bash
0 2 * * * nikto -h http://localhost:3000 -o /var/logs/nikto_scan.log
```

---

## 🧰 OWASP ZAP CLI

### Instalación
```bash
sudo apt install zaproxy -y
```

### Escaneo rápido
```bash
zaproxy -cmd -quickurl http://localhost:3000 -quickout scan_result.html
```

---

## ⚙️ CI/CD con ZAP

### GitHub Actions
`.github/workflows/zap_scan.yml`:

```yaml
name: DAST Scan
on: push

jobs:
  zap_scan:
    runs-on: ubuntu-latest
    steps:
      - name: Instalar OWASP ZAP
        run: sudo apt-get install zaproxy -y

      - name: Ejecutar escaneo
        run: zaproxy -cmd -autorun zap_scan.yaml

      - name: Guardar reporte
        uses: actions/upload-artifact@v3
        with:
          name: ZAP-Report
          path: zap_report.html
```

`zap_scan.yaml`:
```yaml
jobs:
  - type: spider
    parameters:
      url: "http://localhost:3000"
      maxDuration: 2
  - type: activeScan
    parameters:
      url: "http://localhost:3000"
      recurse: true
      maxDuration: 5
  - type: report
    parameters:
      template: traditional-html
      reportFile: zap_report.html
      reportTitle: "ZAP Scan Report"
```

---

### GitLab CI/CD

`.gitlab-ci.yml`:
```yaml
stages:
  - dast

zap_scan:
  stage: dast
  image: owasp/zap2docker-stable
  script:
    - zap.sh -cmd -autorun zap_scan.yaml
  artifacts:
    paths:
      - zap_report.html
```

---

### Jenkins Declarativo

`Jenkinsfile`:
```groovy
pipeline {
  agent any
  stages {
    stage('Download OWASP ZAP') {
      steps {
        sh 'sudo apt-get install zaproxy -y'
      }
    }
    stage('Run OWASP ZAP Scan') {
      steps {
        sh 'zaproxy -cmd -autorun zap_scan.yaml'
      }
    }
    stage('Save Report') {
      steps {
        archiveArtifacts artifacts: 'zap_report.html', fingerprint: true
      }
    }
  }
}
```

---

## 📚 Créditos

Adaptado de la actividad "Análisis de Seguridad Dinámico con DAST".