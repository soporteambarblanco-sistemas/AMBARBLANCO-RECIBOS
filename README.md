# Sistema de Recibos — Ámbar Blanco

Generador de recibos con vista previa en vivo, exportación a PDF y guardado automático en Google Sheets.

## Qué contiene esta carpeta

- `index.html` — genera cotizaciones y recibos en PDF (logo, firma y conexión a Google Sheets ya integrados).
- `venta.html` — formulario para registrar el perfil completo del cliente cuando una venta ya se concretó.
- `clientes.html` — panel de historial, alertas de cumpleaños y control de clientes, alimentado por la hoja "Clientes".
- `dashboard.html` — gráfica de ventas mensuales, crecimiento vs. mes anterior, y ranking de productos más vendidos.

Sube los cuatro archivos juntos a la raíz de tu repositorio — están enlazados entre sí desde sus menús de navegación.

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

## Actualizar tu Apps Script (una sola vez, para que funcione el catálogo y el panel de clientes)

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

  if (data.tipo === 'cliente') {
    var hojaClientes = ss.getSheetByName('Clientes');
    if (!hojaClientes) {
      hojaClientes = ss.insertSheet('Clientes');
      hojaClientes.appendRow(['Nombre', 'Telefono', 'Correo', 'Direccion', 'Cumpleanos', 'Canal', 'FechaCompra', 'Productos', 'Total', 'Notas', 'CuentaDestino']);
    }
    hojaClientes.appendRow([
      data.nombre, data.telefono, data.correo, data.direccion, data.cumpleanos,
      data.canal, data.fechaCompra, data.productos, data.total, data.notas, data.cuentaDestino
    ]);
    return ContentService.createTextOutput(JSON.stringify({status: 'ok'}))
      .setMimeType(ContentService.MimeType.JSON);
  }

  // Recibo o cotización normal (comportamiento original)
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
  var tipo = (e.parameter.tipo || 'catalogo');

  if (tipo === 'clientes') {
    var hojaC = ss.getSheetByName('Clientes');
    var clientes = [];
    if (hojaC) {
      var datosC = hojaC.getDataRange().getValues();
      for (var k = 1; k < datosC.length; k++) {
        if (!datosC[k][0]) continue;
        clientes.push({
          nombre: datosC[k][0],
          telefono: datosC[k][1],
          correo: datosC[k][2],
          direccion: datosC[k][3],
          cumpleanos: datosC[k][4],
          canal: datosC[k][5],
          fechaCompra: datosC[k][6],
          productos: datosC[k][7],
          total: datosC[k][8],
          notas: datosC[k][9],
          cuentaDestino: datosC[k][10]
        });
      }
    }
    return ContentService.createTextOutput(JSON.stringify(clientes))
      .setMimeType(ContentService.MimeType.JSON);
  }

  if (tipo === 'recibos') {
    var hoja = ss.getSheets()[0];
    var datos = hoja.getDataRange().getValues();
    var recibos = [];
    for (var i = 1; i < datos.length; i++) {
      if (!datos[i][0]) continue;
      recibos.push({
        folio: datos[i][0],
        fecha: datos[i][1],
        cliente: datos[i][2],
        telefono: datos[i][3],
        correo: datos[i][4],
        conceptos: datos[i][5],
        subtotal: datos[i][6],
        iva: datos[i][7],
        total: datos[i][8],
        formaPago: datos[i][9],
        notas: datos[i][10]
      });
    }
    return ContentService.createTextOutput(JSON.stringify(recibos))
      .setMimeType(ContentService.MimeType.JSON);
  }

  // catálogo (comportamiento existente)
  var hoja2 = ss.getSheetByName('Catálogo');
  var productos = [];
  if (hoja2) {
    var datos2 = hoja2.getDataRange().getValues();
    for (var j = 1; j < datos2.length; j++) {
      if (!datos2[j][0]) continue;
      productos.push({
        producto: datos2[j][0],
        etiqueta: datos2[j][1],
        precio: datos2[j][2],
        nota: datos2[j][3]
      });
    }
  }
  return ContentService.createTextOutput(JSON.stringify(productos))
    .setMimeType(ContentService.MimeType.JSON);
}
```

Guarda (Ctrl+S), luego **Implementar → Administrar implementaciones → ícono de lápiz (editar) → Versión: Nueva versión → Implementar**. Así conservas la misma URL `/exec` que ya tienes integrada — no hace falta cambiar nada en ninguno de los archivos HTML.

**Importante:** si tu hoja "Clientes" ya existe (ya la usaste antes), el encabezado de la columna K no se va a poner solo — agrégalo tú a mano: abre tu Sheets, ve a la hoja "Clientes", y escribe `CuentaDestino` en la celda K1. Las ventas nuevas van a caer ahí correctamente.

La hoja "Clientes" se crea sola la primera vez que registres una venta desde `venta.html` — no necesitas crearla a mano.

## Registro de venta (`venta.html`)

- Úsalo **solo cuando una cotización se concreta en venta real** — así tu base de clientes no se mezcla con cotizaciones que no se cerraron.
- Captura perfil completo: nombre, teléfono, correo, dirección, cumpleaños, canal de venta (directa/web/WhatsApp/otro).
- Los productos se agregan con los mismos chips del catálogo que en `index.html`.
- Cada envío es una fila nueva en la hoja "Clientes" — si el mismo cliente compra de nuevo, se captura otra vez y el panel de clientes junta automáticamente su historial.

## Panel de clientes (`clientes.html`)

- Agrupa automáticamente todas tus ventas confirmadas por cliente (nombre + teléfono).
- Muestra perfil completo (teléfono, correo, dirección, cumpleaños, canal habitual), total gastado, número de compras, última compra, e historial detallado con lista clara de productos y fecha de cada compra.
- **Alerta de cumpleaños**: si algún cliente cumple años en los próximos 30 días, aparece un aviso destacado arriba — ideal para dar seguimiento postventa.
- Buscador por nombre + ordenar por mayor gasto / más reciente / más compras / alfabético.

## Dashboard de ventas (`dashboard.html`)

- Lee las ventas confirmadas de tu hoja "Clientes" (no las cotizaciones) y las agrupa por mes.
- Gráfica de barras por mes, con botón para cambiar a vista "Acumulado" (línea de crecimiento total).
- Tarjetas arriba: ventas totales, compras registradas, ventas del mes actual, y % de crecimiento vs. el mes anterior.
- Tabla de los productos más vendidos (contando cuántas veces aparece cada uno en tus ventas).
- No necesita configuración — usa la misma URL de Google Sheets ya integrada, y el mismo `doGet` que ya tienes.

