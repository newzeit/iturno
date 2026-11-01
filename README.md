# ITurno

MVP de recepción para servicio técnico IT. HTML, CSS y JavaScript vanilla, servidor Node.js en un único `server.js`. Sin paquetes externos ni compilación.

## Inicio rápido

Requiere Node.js 22.9 o superior (probado con 24.22.1).

1. Descomprimí el ZIP y entrá en la carpeta `iturno`.
2. Editá `.env` (ya incluido, sin credenciales). Asigná una contraseña propia de al menos 12 caracteres a `ADMIN_PASSWORD`.
3. Ejecutá `npm start`.
4. Abrí http://localhost:3000. El panel técnico usa tu `ADMIN_PASSWORD`.

```bash
cd iturno
nano .env
npm start
```

No hace falta `npm install`: se utilizan módulos nativos de Node.js. No abras el HTML con file://; se sirve desde Node.

## Activar OpenAI

En `.env` configurá:

```dotenv
OPENAI_API_KEY=tu_clave_real
OPENAI_MODEL=gpt-4.1-mini
DEMO_MODE=false
ADMIN_PASSWORD=tu_contrasena_privada_de_al_menos_12_caracteres
PORT=3000
HOST=127.0.0.1
PUBLIC_ORIGIN=http://localhost:3000
COOKIE_SECURE=false
```

Reiniciá el servidor. El modelo se puede cambiar por uno disponible en tu cuenta compatible con Responses API. La llamada a `https://api.openai.com/v1/responses` está implementada en `server.js` con `fetch`, límite de respuesta y timeout. La clave nunca se entrega al navegador. El modo demo usa respuestas programadas y se identifica en pantalla; no cambia silenciosamente a demo si OpenAI falla. Los turnos y solicitudes funcionan sin IA.

Referencia utilizada: https://developers.openai.com/api/docs/guides/text

## Funciones

- Asistente de recepción; recoge síntomas sin prometer diagnósticos ni precios.
- Formulario independiente con nombre, contacto, equipo, problema y modalidad.
- Copia opcional de la conversación al detalle, con revisión del cliente.
- Turnos de una hora, lunes a viernes, 9–12 y 14–17, hora de Argentina. Ventana de 15 días y anticipación mínima de una hora. Se reserva una sola atención por horario entre todas las modalidades.
- Reservas sin duplicados: validación y escritura síncronas dentro de un único proceso.
- Solicitud sin turno para quien prefiera que lo contacten.
- Código privado aleatorio y enlace de seguimiento; quien tenga el enlace puede consultar estado y mensaje público.
- Panel protegido con cookie HttpOnly y sesión de 8 horas, filtro por estado y actualizaciones públicas.
- Persistencia en `data/tickets.json`, creado al guardar el primer caso; escritura mediante archivo temporal y renombrado.

## Archivos

- `server.js`: servidor, API, sesiones, validación, turnos y OpenAI.
- `public/index.html`, `style.css`, `app.js`: interfaz responsive.
- `.env`: configuración local sin secretos incluidos.
- `.env.example`: plantilla para otros entornos.
- `.gitignore`: excluye secretos y datos de clientes.
- `test/api.test.js`: pruebas del flujo principal.

## Pruebas

```bash
npm test
```

Las pruebas usan datos temporales y modo demo: reservas simultáneas, conflicto, validaciones, panel no autorizado, login, actualización, seguimiento sin datos de contacto, rechazo de origen externo, rechazo de roles no permitidos, protección de `.env` y persistencia después de reiniciar. No consumen OpenAI.

Prueba manual: enviar consulta → completar formulario → reservar → guardar enlace → entrar al panel → cambiar estado → volver al enlace de seguimiento.

## Publicar en VPS

Este entregable es una aplicación Node.js para ejecutar en tu equipo o VPS. No es una página estática para copiar sola a public_html.

- Guardá el proyecto fuera de la raíz pública y ejecutalo como usuario sin privilegios.
- Configurá Apache/Nginx como proxy HTTPS hacia `127.0.0.1:3000`.
- En `.env`, fijá `PUBLIC_ORIGIN=https://tu-dominio` (sin barra final) y `COOKIE_SECURE=true`.
- Ejecutá un único proceso con systemd, WorkingDirectory apuntando al proyecto y ExecStart a la ruta de Node seguida de la ruta a `server.js`.
- Conservá `.env` privado (`chmod 600 .env`) y respaldá `data/`.
- No subas `.env` ni `data/` a GitHub. Las sesiones se pierden al reiniciar: basta volver a iniciar sesión.

## Alcance del MVP

No incluye WhatsApp, emails/SMS, pagos, calendario externo, adjuntos ni reprogramación. El contacto es un dato guardado: el sistema no envía notificaciones automáticamente. La IA no escribe en la agenda. El cliente debe confirmar el formulario.

La persistencia JSON está pensada para una instancia pequeña con un único proceso. Para varios procesos o mayor volumen, migrar a una base de datos con restricción única de turnos. No editar el JSON mientras el servidor está en ejecución. Un archivo corrupto impide arrancar para evitar sobrescribir datos.

Hay límites por IP de conexión (sin confiar en X-Forwarded-For), tamaño de solicitudes y longitud del chat. Detrás de un proxy, el límite puede compartirse entre usuarios. Antes de abrirlo a mucho tráfico, configurar límites en el proxy y protección contra abuso de la API de IA; establecer también presupuesto en la cuenta de OpenAI. El formulario puede registrar duplicados si una respuesta se pierde y el cliente reintenta; el bloqueo de horario sigue vigente.

La integración real con OpenAI necesita tu clave y conectividad; no se ha probado con una cuenta real. No se realizó una revisión visual automatizada en navegador en este entorno.
