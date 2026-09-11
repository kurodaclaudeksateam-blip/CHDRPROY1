# Checador — Control de Asistencia

App web de control de asistencia para pruebas: un **checador público en pantalla completa** (sin necesidad de iniciar sesión, pensado para una tablet en la entrada) donde cualquier colaborador elige su nombre y valida su checada por reconocimiento facial simulado, 4 checadas por día (entrada, salida a comer, regreso de comida, salida), alta de empleados con foto y tienda(s) asignada(s), calendario mensual de checadas, y solicitudes de justificación/permiso dirigidas a jefe inmediato y RH.

## Archivos

- **`index.html`** — versión standalone. Ábrela directamente en el navegador (doble clic o `file://`) y funciona sola en **modo local**: los datos se guardan en el `localStorage` de ese navegador (clave `checador_local_db_v1`), sin necesidad de servidor ni backend. Ideal para pruebas rápidas.
- **`checador-fragment.html`** — el mismo código sin las etiquetas `<!DOCTYPE>/<html>/<head>/<body>`, pensado para publicarse como Artifact en claude.ai (con las capacidades `db` y `downloads`, para que todos los que abran el enlace compartan los mismos datos en tiempo real).

## Probar en local

1. Descarga o clona el repo.
2. Abre `index.html` en el navegador (Chrome/Edge recomendado).
3. Verás el aviso "Modo local" en la pantalla de inicio de sesión — es normal, indica que estás en modo `localStorage`.
4. Código de acceso por defecto para entrar como **RH / Admin** o **Jefe inmediato**: `1234` (configurable luego en Ajustes).
5. Da de alta un empleado (con foto, puesto y tienda) desde RH → Empleados.

### El checador (tablet de entrada)

El botón **"Abrir checador en pantalla completa"** en la pantalla de inicio de sesión (o el enlace con `#kiosk` al final, p. ej. `index.html#kiosk`) abre el checador **sin pedir login** — es la pantalla que se deja fija en la tablet de la entrada. La cámara ocupa toda la pantalla y **nadie selecciona nada**: el checador compara el rostro frente a la cámara contra las fotos de perfil de los empleados y, en cuanto encuentra una coincidencia, captura, verifica y registra la checada sola, mostrando nombre, tipo de checada (entrada/salida a comer/regreso/salida), hora y **% de similitud facial**. Si no reconoce a nadie (o el reconocimiento no está disponible), aparece un botón "¿No te reconoce? Busca tu nombre" como respaldo manual. Un botón pequeño de pantalla completa y otro de salida (✕) quedan en la esquina superior derecha para que RH/Jefe puedan reconfigurar la tablet.

RH y Jefe también pueden abrirlo desde dentro de su sesión con el botón **"Abrir checador"** en la barra lateral.

### Reconocimiento facial: cómo funciona y sus límites

El checador usa [face-api.js](https://github.com/justadudewhohacks/face-api.js) (se carga desde `cdn.jsdelivr.net`) para detectar el rostro y comparar su "huella" con la de la foto de perfil de cada empleado, calculando un % de similitud a partir de la distancia entre ambos vectores faciales — **no es biometría de nivel empresarial**, es un modelo ligero pensado para correr en el navegador, y funciona mejor con buena luz y la foto de perfil de frente.

- **Los modelos se descargan de un CDN externo en tiempo real** (`justadudewhohacks.github.io/face-api.js/models`). En `index.html`, servido normal (https/localhost) o abierto como archivo local, esto funciona sin problema — ya lo probamos: los tres modelos cargan correctamente.
- **Dentro del Artifact publicado en claude.ai no va a funcionar** — el sandbox del Artifact bloquea las descargas de red hacia hosts fuera de su lista permitida (los modelos no están en esa lista). El checador lo detecta solo y **cae de forma automática** al buscador de nombre manual, con un aviso claro ("Reconocimiento facial no disponible aquí — busca tu nombre"). Nunca se rompe, solo pierde el paso automático.
- Un empleado sin foto de perfil no puede reconocerse automáticamente — hay que darlo de alta con foto desde RH → Empleados.
- Para que la tablet reconozca a alguien que se dio de alta después de abrir el checador, hay que volver a abrirlo (los rostros se calculan una vez al iniciar).

### Checadas por día

Cada colaborador tiene 4 checadas por día, siempre en este orden: **Entrada → Salida a comer → Regreso de comida → Salida**. Si dos checadas quedan muy cerca en tiempo (menos de 5 minutos), se marcan con ⚠️ como posible error de checada, visible tanto en la tabla de actividad de RH como en el calendario del empleado.

### Empleado — "Mi tarjeta"

Ya no incluye cámara (eso vive en el checador público). Es un resumen de solo lectura con accesos directos a **Mi calendario** (historial de entradas/salidas) y **Mis solicitudes** (justificaciones y permisos) — ahí es donde el empleado revisa su información, después de iniciar sesión con su nombre.

## Limitaciones (demo)

- El login por rol y el código de acceso son una barrera de **interfaz**, no autenticación real — no uses contraseñas verdaderas.
- Sin backend: en `index.html` los datos viven solo en el navegador donde se usan (no se comparten entre dispositivos). Para compartir datos entre RH, jefes y empleados en tiempo real, publica `checador-fragment.html` como Artifact en claude.ai.
