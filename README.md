# Sistema de Recibos — Ámbar Blanco

Generador de recibos con vista previa en vivo, exportación a PDF y guardado automático en Google Sheets.

## Qué contiene esta carpeta

- `index.html` — el sistema completo (logo, firma y conexión a Google Sheets ya integrados). Es el único archivo que necesitas.

## Subir a GitHub

1. Crea un repositorio nuevo en GitHub (puede ser privado), por ejemplo `recibos-ambar-blanco`.
2. Sube este archivo `index.html` (y este `README.md` si quieres) al repositorio. Puedes arrastrar el archivo directo en la página de GitHub con "Add file → Upload files", o usar:

```bash
git init
git add index.html README.md
git commit -m "Sistema de recibos Ámbar Blanco"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/recibos-ambar-blanco.git
git push -u origin main
```

## Desplegar en Vercel

1. Entra a [vercel.com](https://vercel.com) e inicia sesión con tu cuenta de GitHub.
2. Clic en "Add New..." → "Project".
3. Selecciona el repositorio `recibos-ambar-blanco`.
4. Vercel detecta que es un sitio estático automáticamente — no necesitas cambiar ninguna configuración (Framework Preset: "Other" está bien).
5. Clic en "Deploy". En menos de un minuto te da una URL tipo `recibos-ambar-blanco.vercel.app`.
6. Esa URL ya es tu sistema de recibos funcionando en internet, accesible desde tu celular o computadora, en cualquier momento.

## Notas importantes

- La URL de tu Google Apps Script ya está integrada en el archivo — cada recibo que guardes desde cualquier dispositivo caerá en tu misma hoja de Google Sheets.
- Si en algún momento vuelves a publicar (redeploy) tu Apps Script y Google te da una URL nueva, solo reemplaza el valor dentro de `index.html` en la línea del campo `id="sheetsUrl"`, o pégala manualmente en "Configurar conexión" dentro del sistema.
- Cada vez que hagas `git push` con cambios, Vercel vuelve a desplegar automáticamente.
- El folio y el nombre del PDF se generan solos con el formato `AAMMDD.NOMBRECLIENTE`.
