# Muestreo Íctiológico prueba app

App de toma de datos de campo para muestreos ícticos en ríos de la cuenca del Duero.

## Funcionalidades

- Registro de puntos de muestreo con ID automático y manual
- Captura de coordenadas GPS con enriquecimiento automático (localidad + cauce vía Overpass/Nominatim)
- Registro de pasadas y capturas (especie, longitud mm, peso g)
- Dictado por voz en observaciones (Chrome Android)
- Funcionamiento offline completo — sincroniza con Google Sheets al recuperar conexión
- Exportación a CSV

## Despliegue en GitHub Pages (gratuito)

### Paso 1 — Crear repositorio

1. Entra en [github.com](https://github.com) e inicia sesión (o crea cuenta gratuita)
2. Clic en **New repository**
3. Nombre: `muestreo-ictico` (o el que prefieras)
4. Visibilidad: **Public** (necesario para GitHub Pages gratuito)
5. Clic en **Create repository**

### Paso 2 — Subir los ficheros

Opción A — Interfaz web (sin instalar nada):
1. En el repositorio recién creado, clic en **Add file → Upload files**
2. Arrastra los 4 ficheros: `index.html`, `manifest.json`, `sw.js`, `README.md`
3. Añade dos iconos PNG: `icon-192.png` e `icon-512.png` (cualquier imagen cuadrada renombrada)
4. Clic en **Commit changes**

Opción B — Git desde terminal:
```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/TU_USUARIO/muestreo-ictico.git
git push -u origin main
```

### Paso 3 — Activar GitHub Pages

1. En el repositorio → **Settings → Pages**
2. Source: **Deploy from a branch**
3. Branch: **main** → carpeta **/ (root)**
4. Clic en **Save**
5. En 1-2 minutos la app estará en: `https://TU_USUARIO.github.io/muestreo-ictico/`

### Paso 4 — Instalar en Android

1. Abre Chrome en el móvil Android
2. Navega a la URL de GitHub Pages
3. Chrome mostrará un banner "Añadir a pantalla de inicio" — aceptar
4. O bien: menú (⋮) → **Añadir a pantalla de inicio**
5. La app se instala como nativa, sin Google Play

## Iconos

Necesitas dos ficheros PNG cuadrados:
- `icon-192.png` — 192×192 px
- `icon-512.png` — 512×512 px

Puedes usar cualquier imagen y renombrarla, o generar iconos en [favicon.io](https://favicon.io).

## Google Sheets — Script

En el Google Sheet destino, crea una hoja llamada **Capturas**.
Luego ve a [script.google.com](https://script.google.com) y pega:

```javascript
const SHEET_ID = 'ID_DE_TU_GOOGLE_SHEET';

function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const ss = SpreadsheetApp.openById(SHEET_ID);
    let sheet = ss.getSheetByName('Capturas');
    if (!sheet) sheet = ss.getSheets()[0];

    if (sheet.getLastRow() === 0) {
      sheet.appendRow([
        'ID_auto','ID_manual','Num_punto','Lat','Lon',
        'Rio_CHD','Localidad','Fecha','Pasada','Num_captura',
        'Especie','Longitud_mm','Peso_g','Observaciones'
      ]);
    }

    const rows = data.rows || [];
    rows.forEach(r => sheet.appendRow([
      r.autoId, r.manualId, r.numPunto, r.lat, r.lon,
      r.rio, r.localidad, r.fechaHora, r.pasada, r.numCaptura,
      r.especie, r.longitud, r.peso, r.observaciones
    ]));

    return ContentService
      .createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch(err) {
    return ContentService
      .createTextOutput(JSON.stringify({ ok: false, error: err.message }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

Implementar → Nueva implementación → Aplicación web:
- Ejecutar como: **Yo**
- Quién tiene acceso: **Cualquier persona**

La URL del webhook ya está preconfigurada en la app.

## Estructura de datos en Google Sheets

| Columna | Descripción |
|---|---|
| ID_auto | Identificador automático (PM-0001...) |
| ID_manual | Identificador introducido por el técnico |
| Num_punto | Número de punto de muestreo |
| Lat / Lon | Coordenadas GPS (WGS84) |
| Rio_CHD | Nombre del cauce (fuente Overpass/OSM) |
| Localidad | Localidad más cercana (fuente Nominatim) |
| Fecha | ISO 8601 fecha/hora de captura |
| Pasada | Número de pasada |
| Num_captura | Número correlativo de captura en la pasada |
| Especie | Especie (lista cerrada) |
| Longitud_mm | Longitud total en mm |
| Peso_g | Peso en gramos |
| Observaciones | Texto libre / dictado por voz |
