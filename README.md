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
- **Catálogo de productos**: ahora puedes agregar productos nuevos directo desde el botón "+ Agregar producto nuevo al catálogo" dentro del sistema. Se guardan en una pestaña llamada "Catálogo" en tu misma hoja de Sheets y aparecen como chips automáticamente en todos tus dispositivos — no necesitas tocar el código para esto. **Requiere que actualices tu Apps Script una vez con el código nuevo (ver abajo).**
- Si en algún momento vuelves a publicar (redeploy) tu Apps Script y Google te da una URL nueva, solo reemplaza el valor dentro de `index.html` en la línea del campo `id="sheetsUrl"`, o pégala manualmente en "Configurar conexión" dentro del sistema.
- Cada vez que hagas `git push` con cambios, Vercel vuelve a desplegar automáticamente.
- El folio y el nombre del PDF se generan solos con el formato `AAMMDD.NOMBRECLIENTE`.

## Actualizar tu Apps Script (una sola vez, para que funcione el catálogo)

Entra a tu proyecto de Apps Script (script.google.com) y reemplaza TODO el código por este, cambiando `TU_ID_DE_HOJA_AQUI` por el ID de tu hoja:

```javascript
function doPost(e) {
  var ss = SpreadsheetApp.openById('TU_ID_DE_HOJA_AQUI');
  var data = JSON.parse(e.postData.contents);

  if (data.tipo === 'producto') {
    var hoja = ss.getSheetByName('Catálogo');
    if (!hoja) {
      hoja = ss.insertSheet('Catálogo');
      hoja.appendRow(['Producto', 'Etiqueta', 'Precio', 'Nota']);
    }
    (data.tamanos || []).forEach(function(t) {
      hoja.appendRow([data.nombre, t.etiqueta, t.precio, data.nota || '']);
    });
    return ContentService.createTextOutput(JSON.stringify({status: 'ok'}))
      .setMimeType(ContentService.MimeType.JSON);
  }

  // Recibo normal (comportamiento original)
  var hojaRecibos = ss.getSheets()[0];
  hojaRecibos.appendRow([
    data.folio, data.fecha, data.cliente, data.telefono, data.correo,
    data.conceptos, data.subtotal, data.iva, data.total, data.formaPago, data.notas
  ]);
  return ContentService.createTextOutput(JSON.stringify({status: 'ok'}))
    .setMimeType(ContentService.MimeType.JSON);
}

function doGet(e) {
  var ss = SpreadsheetApp.openById('TU_ID_DE_HOJA_AQUI');
  var hoja = ss.getSheetByName('Catálogo');
  var productos = [];
  if (hoja) {
    var datos = hoja.getDataRange().getValues();
    for (var i = 1; i < datos.length; i++) {
      if (!datos[i][0]) continue;
      productos.push({
        producto: datos[i][0],
        etiqueta: datos[i][1],
        precio: datos[i][2],
        nota: datos[i][3]
      });
    }
  }
  return ContentService.createTextOutput(JSON.stringify(productos))
    .setMimeType(ContentService.MimeType.JSON);
}
```

Guarda (Ctrl+S), luego **Implementar → Administrar implementaciones → ícono de lápiz (editar) → Versión: Nueva versión → Implementar**. Así conservas la misma URL `/exec` que ya tienes integrada — no hace falta cambiar nada en `index.html`.

