# 📋 Pipeline CI/CD para Alumnos

Este proyecto es un ejemplo completo de **CI/CD (Integración Continua / Despliegue Continuo)** usando GitHub Actions.

## 🎯 ¿Qué hace este pipeline?

1. **Instala dependencias** - Descarga prettier
2. **Verifica formato** - Valida que el código esté bien formateado (Prettier)
3. **Análisis de calidad** - Revisa el código con SonarCloud
4. **Despliegue** - Si todo pasa, despliega automáticamente a GitHub Pages

## 🚀 Para empezar (Local)

```bash
# 1. Instalar dependencias
npm install

# 2. Verificar formato
npm run format:check

# 3. Arreglar formato automáticamente
npm run format:fix
```

## 📝 Archivos importantes

- `main.js` - Tu código principal
- `index.html` - Página web que se despliega
- `styles.css` - Estilos de la página
- `.prettierrc` - Configuración de formato
- `.github/workflows/verificacion.yaml` - Pipeline de CI/CD
- `sonar-project.properties` - Configuración de SonarCloud

## ⚙️ Configurar SonarCloud (Opcional)

1. Ir a https://sonarcloud.io
2. Crear cuenta y proyecto
3. Copiar el `SONAR_TOKEN`
4. En GitHub → Settings → Secrets → Nuevo secret: `SONAR_TOKEN`
5. Actualizar `sonar-project.properties` con tu `projectKey`

## ✨ Flujo automático

Cada vez que hagas `git push`:

```
Push al repositorio
    ↓
Instalar dependencias (npm install)
    ↓
Prettier verifica formato ✓
    ↓
SonarCloud analiza calidad ✓
    ↓
¡Despliegue automático a GitHub Pages! 🎉
```

Si Prettier encuentra problemas ❌, el pipeline se detiene y no despliega.

## 🐛 Debugging

Si el pipeline falla:

1. **Prettier** - Corre `npm run format:fix` y haz push
2. **SonarCloud** - Valida que `SONAR_TOKEN` esté configurado en Secrets
