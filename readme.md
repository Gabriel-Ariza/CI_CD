
# 🚀 Guía de Taller: Pipeline de CI/CD con Calidad Automatizada

Bienvenidos a este taller práctico.  
Hoy aprenderemos a construir una infraestructura que no solo despliega aplicaciones, sino que las protege mediante análisis estático de código.


## 📋 Requisitos Previos

Antes de comenzar, asegúrate de tener:

- ✅ Cuenta en GitHub
- ✅ Cuenta en SonarCloud.io (entrar con GitHub)
- ✅ Node.js instalado localmente


# 🏗️ Fase 1: Configuración del Proyecto Local

Primero prepararemos nuestra aplicación (Calculadora Sumadora) y sus herramientas de calidad.

### 1️⃣ Inicializar el Repositorio

En tu terminal de linux:

```bash
mkdir taller-cicd
cd taller-cicd
npm init -y
```
---

## 🔹 Instalar Prettier

### 2️⃣ Instalación de Herramientas de Estilo (Linting)

Instalaremos Prettier para asegurar que todos tengan el mismo formato de código:

```bash
npm install --save-dev prettier
```
---

## 🔹 Crear Archivos Base

### 3️⃣ Crear los Archivos Base


Entra al repositorio https://github.com/Gabriel-Ariza/CI_CD/ y copia los archivos del proyecto:

```
- index.html → Interfaz de la calculadora
- styles.css → Diseño visual
- main.js → Lógica de la suma
- .prettierrc → Reglas de formato
- package.json -> comandos & dependencias
```

# ☁️ Fase 2: Conexión con SonarCloud

### Crear Proyecto

En SonarCloud:

1. Haz clic en "+"
2. Selecciona "Analyze new project"

### Vincular Repo

Selecciona tu repositorio de GitHub.

### Método de Análisis

Selecciona:

GitHub Actions

### Configurar Credenciales

```bash
1. ve a tu perfil -> seguridad
2. SonarCloud te dará un SONAR_TOKEN.
3. Ve a tu repositorio en GitHub:
   Settings → Secrets and variables → Actions
4. Crea un nuevo secreto:

   Name: SONAR_TOKEN  
   Value: (pega el token)
```

## creamos las propiedades para sonnarCloud
```bash
sonar.projectKey=clave del proyecto Sonnar
sonar.projectName=Nombre del repositorio github
sonar.organization= Nombre de Sonnar
sonar.projectVersion=1.0.0
sonar.sources=.
sonar.exclusions=node_modules/**,coverage/**,.github/**,dist/**
sonar.javascript.lcov.reportPaths=coverage/lcov.info
sonar.sourceEncoding=UTF-8
```

# 🤖 Fase 3: Automatización con GitHub Actions

Crearemos un archivo en la ruta:

.github/workflows/verificacion.yaml


## 🧪 Calidad-y-analisis

Este trabajo:

- Instala dependencias con:

```bash
npm install
```

Verificar instalación con prettier dando formato al código:
```bash
npm run format:check
```

## CREAMOS EL PIPELINE

**Primera Parte:** necesitamos crear un job que me verifique el formato y codigo limpio del proyecto, y una segunda fase que me analice la calidad de este. Que sea aceptado si pasa las 2 fases.

```bash
name: Pipeline Calidad #nombre del pipeline en github
on: [push] # cada vez que hagamos un push en el repositorio

jobs:
  calidad-y-analisis: #nombramos el trabajo a ejecutar
    runs-on: ubuntu-latest # que corra en un servidor virtual de linux
    steps:
      - uses: actions/checkout@v4.2.2
      # usamos una herramienta ya creada en un repositorio oficial de github
      # esta copia nuestros archivos del proyecto y los pone en la maquina virtual


      # 0. INSTALAR DEPENDENCIAS
      - name: Instalar dependencias
        run: npm install
        #le dice a github que corra el comando en la terminal de la maquina virtual


      # 1. VERIFICACIÓN DE ESTILO (Prettier)
      - name: Verificar formato con Prettier
        run: npm run format:check #chequea internamente con la dependencia si es codigo formateado


      # 2. ANÁLISIS DE SONARCLOUD
      - name: SonarCloud Scan
        uses: SonarSource/sonarqube-scan-action@a31c9398be7ace6bbfaf30c0bd5d415f843d45e9
        # descarga y ejecuta la herramienta SonarSouce(lector y analizador de codigo) en la VM
        # usamos SHA para evitar inyeccion de codigo en caso de que actualicen el repositorio oficial, sin SHA falla

        # encriptamos estos datos para evitar vulnerabilidades en el .yaml
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} #variable por defecto de github
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }} #accedemos a la variable que creamos en entornos/github
        with:
          args: >
            -Dsonar.projectKey=Gabriel-Ariza_CI_CD #configuracion en que proyecto debe actuar Sonnar
            -Dsonar.organization=gabriel-ariza
            -Dsonar.qualitygate.wait=true
            # importante, esta es la puerta de Sonnar, si encuentra incosistencias falla el pipeline
            # con false se desactiva la puerta, solo analiza codigo y pasa el pipeline

```

## SEGUNDA PARTE DEL PIPELINE
al validar y pasar el primer job, esta se encarga de desplegar automáticamente el **push** en github pages
que es de lo que trata nuestro proyecto, una pagina web

```bash
  despliegue: #nombramos pipeline en github(apartado de actions)
    needs: calidad-y-analisis # llamamos al pipeline anterior y solo compila si pasó en verde
    runs-on: ubuntu-latest
    permissions:
      contents: write # otorgamos permisos al pipeline en el repositorio para que despliegue en github pages
    steps:
      - uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd
      # Usamos la version Pin v4.2.2

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@4f9cc6602d3f66b9c108549d475ec49e8ef4d45e
        # usamos la herramienta comunitaria que sube los archivos a github pages
        # esta crea una rama gh-pages que toma los archivos y los sube a pages automaticamente

        # Configuracion Propiedades
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./
```



Dato: cuando hagas **git push** se envia el código al servidor de SonarCloud

Usamos `-Dsonar.qualitygate.wait=true` para que el pipeline se detenga si la calidad es baja.


---

## 🔹 Trabajo 2: Despliegue


## 🚀 Despliegue (CD)

Este trabajo:

- Solo se ejecuta si el análisis fue exitoso:

```yaml
needs: calidad-y-analisis
```
---
está en 2 fases: la primera evalua el codigo limpio y la calidad, la segunda despliega los cambios en Github Pages


# 🛠️ Fase 4: Comandos de Operación

### Corregir formato del código automáticamente

Sin esto, el pipeline fallará ya que está configurado para permitir solo código limpio con prettier

```bash
npm run format:fix
```
---

## 🔹 Verificar formato

### Simular verificación de pipeline

```bash
npm run format:check
```
---


## revisamos la estructura del proyecto
```
- verificacion.yaml este es el pipeline
- index.html → Interfaz de la calculadora
- styles.css → Diseño visual
- main.js → Lógica de la suma
- .prettierrc → Reglas de formato codigo limpio
- .prettierignore → evitamos estilos en modulos
- .gitignore → que se suban solo archivos necesarios 
- package.json → comandos & dependencias
- package-lock-json → creado al compilar npm
- sonar-project.properties → configuracíon sonar
```


## subimos todo a github
```bash
git add .
git commit -m "primera subida"
git push
```


# 📘 PARTE 7 — Test de Estrés

# 🧪 Fase 5: Test de Estrés (Demostración)
### Inyectar Vulnerabilidad

Modificar la suma por:

```javascript
// Código Peligroso
const resultado = eval(num1 + '+' + num2);
```
---

## 🔹 Hacer push para verificar

### Push a GitHub

```bash
git add .
git commit -m "test error"
git push
```
---

## 🔹 Resultado Esperado

### Observar el Bloqueo

- SonarCloud detectará Code Injection
- El Quality Gate pasará a estado FAILED
- Fallará el pipeline
- El trabajo de Despliegue se cancelará automáticamente


# 🌐 Fase 6: Acceso a la Aplicación

Una vez que el pipeline pase en verde:

https://<TU-USUARIO>.github.io/<NOMBRE-REPO>/

## Nota

Si sale error 404:

1. Ir a Settings → Pages
2. Verificar que la rama sea gh-pages