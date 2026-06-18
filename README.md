# GDO Call Center — Gases de Occidente

Plataforma de gestión de órdenes y marcaciones para el call center, integrada con Power Automate + SharePoint (lista *Consolidado Prueba1*).

La aplicación es **un solo archivo** (`index.html`), sin dependencias externas — ideal para publicar en GitHub Pages.

---

## 🚀 Abrir desde GitHub (GitHub Pages)

1. Crea un repositorio (p. ej. `gdo-callcenter`) y sube `index.html` a la raíz.
2. En el repo ve a **Settings → Pages**.
3. En **Build and deployment → Source** elige **Deploy from a branch**.
4. Branch: `main` · carpeta: `/ (root)` → **Save**.
5. A los ~1–2 min la app queda publicada en:
   `https://<tu-usuario>.github.io/gdo-callcenter/`

Como el archivo se llama `index.html`, abre directamente sin agregar nada a la URL.

> **Antes de publicar**, edita el bloque `CFG` dentro de `index.html` y reemplaza las 4 URLs de los flows (`FLOW_LOGIN`, `FLOW_ORDENES`, `FLOW_UPDATE`, `FLOW_CREAR`) por las tuyas.

---

## ⚡ Qué se mejoró en rendimiento y robustez

El problema principal era la espera larga mientras el flow carga la información. Cambios aplicados:

- **Carga instantánea desde caché (stale-while-revalidate).** Al reabrir la app, las órdenes guardadas aparecen al instante y la sincronización con el flow ocurre en segundo plano. Ya no se ve la pantalla en blanco con spinner.
- **Sesión persistente.** Si recargas la página (F5) no tienes que volver a iniciar sesión.
- **Refresco no bloqueante.** “↻ Actualizar” ya no congela la pantalla: muestra una barra de progreso superior y deja seguir trabajando.
- **Timeout / cancelación de peticiones.** Si el flow se cuelga, la app no se queda esperando para siempre: aborta a los 90 s y muestra los datos guardados con un aviso.
- **Skeleton loaders** en la primera carga (en vez de tabla vacía).
- **Indicador de frescura** (“Actualizado hace X min”) en la barra superior.
- **Búsqueda con debounce** para que el filtrado no trabe con listas grandes.

---

## 🔧 Recomendaciones para un funcionamiento más profesional

Estas requieren cambios en los **flows de Power Automate** o en la arquitectura — son las que más impacto tendrán:

1. **Filtrar las órdenes en el flow, no en el navegador.**
   Hoy el flow devuelve *toda* la lista y la app filtra por agente en el cliente. Para un agente eso es traer miles de filas para mostrar 30. Agrega el usuario como parámetro y usa un **OData filter** (`$filter`) en la acción *Obtener elementos* de SharePoint para devolver solo sus órdenes. Esto reduce drásticamente el tamaño de la respuesta y es el mayor acelerador posible.

2. **Paginación / `$top`.** Limita la cantidad de filas por respuesta (p. ej. 200) y trae el resto bajo demanda. SharePoint pagina de a 5.000; pedir todo en una llamada es lento.

3. **Selecciona solo las columnas necesarias (`$select`).** Traer todas las columnas del *Consolidado* pesa. Pide únicamente las que la app usa.

4. **Separar login de carga de datos.** Hoy el login trae usuario + órdenes juntos, así que el ingreso es tan lento como la consulta completa. Si el login solo valida credenciales y las órdenes se piden aparte (con skeleton), el ingreso es casi inmediato.

5. **Seguridad de credenciales.** Las contraseñas están en texto plano en el JS (lista `USUARIOS`) y la sesión se guarda en el navegador. Para una versión robusta: mover la validación 100 % al flow, no embeber contraseñas en el HTML, y usar un token de sesión con expiración en lugar de guardar la contraseña.

6. **Confirmaciones del servidor antes de actualizar la UI.** Hoy las marcaciones y asignaciones se reflejan localmente aunque el flow falle silenciosamente. Conviene mostrar éxito solo cuando el flow responde OK, y revertir si falla.

7. **Asignación masiva en una sola llamada.** Hoy `ejecutarAsignacion` llama al flow una vez por orden (lento y frágil). Mejor enviar el lote completo en un solo POST y que el flow recorra el array con *Apply to each*.

8. **Registrar quién y cuándo.** Guardar auditoría (agente, timestamp, IP) de cada marcación/asignación para reportes y trazabilidad.

---

## 📁 Estructura

```
index.html      ← la aplicación completa (subir a GitHub)
README.md       ← este archivo
```
