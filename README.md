# 🎣 PescaApp — Sistema Inteligente de Registro y Gestión de Capturas

## 📋 1. Supuesto Práctico

**PescaApp** es una plataforma destinada al registro, control y validación de capturas de pesca deportiva.

El principal problema que resuelve es la dificultad de registrar capturas en tiempo real desde el entorno de pesca. En lugar de rellenar formularios complejos, el pescador simplemente envía un mensaje en lenguaje natural a un bot de Telegram.

La información es procesada mediante Inteligencia Artificial (Google Gemini), estructurada automáticamente y almacenada en MongoDB Atlas con estado inicial **"pendiente"**. Posteriormente, un administrador puede validar o rechazar la captura desde una interfaz web.

> Proyecto desarrollado para la asignatura de **Bases de Datos (2º DAM)**.

---

# 🗺️ 2. Modelado de Datos

## A. Modelo Relacional (CFM / Entidad-Relación)

Si se hubiera utilizado una base de datos SQL tradicional, el sistema podría representarse mediante las siguientes entidades:

```text
PESCADORES
│
├── id_pescador (PK)
├── nombre
└── cuenta_telegram

ESPECIES
│
├── id_especie (PK)
├── nombre_comun
└── talla_minima

UBICACIONES
│
├── id_ubicacion (PK)
├── nombre_masa_agua
└── coordenadas

CAPTURAS
│
├── id_captura (PK)
├── id_pescador (FK)
├── id_especie (FK)
├── id_ubicacion (FK)
├── peso
├── tamano
├── fecha
└── estado
```

### Inconvenientes

* Necesidad de múltiples tablas relacionadas.
* Uso frecuente de operaciones `JOIN`.
* Menor flexibilidad ante cambios de estructura.

---

## B. Modelo Documental (MongoDB)

MongoDB permite almacenar toda la información relevante de una captura en un único documento.

### Ejemplo de documento

```json
{
  "_id": "ObjectId(...)",
  "especie": "Black Bass",
  "fecha": "2026-06-13",
  "peso": "2,10 kg",
  "tamano": "49,00 cm",
  "ubicacion": "Embalse de Orellana",
  "pescador": "Marcos",
  "estado": "pendiente"
}
```

### Ventajas

✅ Lecturas más rápidas.

✅ Mayor flexibilidad de esquema.

✅ Posibilidad de añadir nuevos campos sin migraciones.

✅ Integración sencilla con aplicaciones web modernas.

---

# 🏗️ 3. Arquitectura del Sistema

El flujo de procesamiento es completamente automático:

```text
┌─────────────┐
│ Telegram    │
│ Bot         │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ n8n         │
│ Webhook     │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Google      │
│ Gemini      │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ MongoDB     │
│ Atlas       │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Next.js     │
│ Dashboard   │
└─────────────┘
```

### Flujo de trabajo

1. El pescador envía un mensaje al bot de Telegram.
2. n8n recibe el mensaje mediante un Webhook.
3. Google Gemini extrae automáticamente los datos relevantes.
4. Se genera un documento JSON estructurado.
5. MongoDB Atlas almacena la captura con estado `"pendiente"`.
6. El panel web permite validarla o rechazarla.

---

# 🎬 4. Material Audiovisual

## Vídeo Demostración

📹 **Enlace:** *(añadir URL)*

Contenido:

* Registro de captura desde Telegram.
* Procesamiento mediante IA.
* Inserción automática en MongoDB Atlas.
* Validación desde el panel web.

---

## Vídeo Promocional

🎥 **Enlace:** *(añadir URL)*

Contenido:

* Presentación rápida del proyecto.
* Demostración visual del flujo automatizado.
* Aplicación práctica de IA en pesca deportiva.

---

# 💾 5. Operaciones CRUD

## CREATE

```javascript
db.capturas.insertOne({
  especie: "Black Bass",
  fecha: "2026-06-13",
  peso: "2,10 kg",
  tamano: "49,00 cm",
  ubicacion: "Embalse de Orellana",
  pescador: "Marcos",
  estado: "pendiente"
});
```

## UPDATE

```javascript
db.capturas.updateOne(
  {
    pescador: "Anton",
    especie: "Trucha común"
  },
  {
    $set: {
      estado: "confirmada"
    }
  }
);
```

## DELETE

```javascript
db.capturas.deleteOne({
  pescador: "Marcos",
  especie: "Black Bass"
});
```

---

# 🔍 6. Consultas Simples

## Capturas pendientes

```javascript
db.capturas.find({
  estado: "pendiente"
});
```

## Historial de un pescador

```javascript
db.capturas.find({
  pescador: "Anton"
});
```

## Capturas por ubicación

```javascript
db.capturas.find({
  ubicacion: "Río Ebro"
});
```

---

# 📊 7. Consultas Complejas

> Para realizar cálculos sobre los campos `peso` y `tamano`, es necesario transformar previamente los valores de texto a formato numérico mediante `$split`, `$replaceOne` y `$toDouble`.

### Consulta 1: Ranking de capturas pendientes por pescador

```javascript
db.capturas.aggregate([
  { $match: { estado: "pendiente" } },
  { $group: { _id: "$pescador", totalPendientes: { $sum: 1 } } },
  { $sort: { totalPendientes: -1 } },
  { $project: { _id: 0, pescador: "$_id", totalPendientes: 1 } }
]);
```

### Consulta 2: Ubicaciones con peso medio superior a 1 kg

```javascript
// Código de agregación
```

### Consulta 3: Top 3 especies por longitud máxima

```javascript
// Código de agregación
```

---



# 🚀 8. Mejoras Futuras

* Validación avanzada de mensajes mediante nodos condicionales en n8n.
* Geolocalización GPS real de capturas.
* Sistema de autenticación con JWT o Auth0.
* Gestión de fotografías de capturas.
* Estadísticas y mapas de calor geoespaciales.

---

# 📁 9. Estructura del Repositorio

```text
📦 PescaApp
├── capturas.json
├── mi-workflow.json
├── README.md
└── src/
```

| Archivo          | Descripción                         |
| ---------------- | ----------------------------------- |
| capturas.json    | Exportación de la colección MongoDB |
| mi-workflow.json | Flujo de automatización n8n         |
| README.md        | Documentación técnica               |
