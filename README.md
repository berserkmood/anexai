# Anexai — web pública

Sitio estático (HTML + CSS puro, sin frameworks ni dependencias) para
`anexai.es`. Sirve como web pública de la verificación OAuth de
Google (consent screen) y como landing del producto.

## Estructura

```
anexai-web/
├── index.html          Landing
├── privacidad.html      Política de privacidad (incluye la cláusula de Google API Services User Data Policy)
├── terminos.html         Términos y condiciones
├── assets/
│   ├── styles.css
│   └── img/               Logo e imágenes (añadir aquí)
└── README.md
```

No hay build step, backend, formularios, cookies ni analítica: son tres
páginas HTML servidas tal cual.

## Pendiente de rellenar antes de publicar

- **Logo**: colocar `assets/img/logo.svg` (o `.png`) — hay un `<img>` con
  fallback silencioso si el archivo no existe todavía.
- **Imágenes de referencia** en `index.html`: dos placeholders marcados con
  comentarios `<!-- IMAGEN DE REFERENCIA ... -->` con las medidas sugeridas.
- **Vídeo de YouTube** en `index.html` (sección `#video`): el bloque
  `<iframe>` está comentado con instrucciones paso a paso para pegar el ID
  del vídeo una vez grabado. Este mismo vídeo es el que hay que adjuntar en
  la verificación OAuth de Google (ver más abajo).
- **Datos legales** en `privacidad.html` y `terminos.html`: buscar los
  textos resaltados tipo `[Nombre o razón social]`, `[NIF/CIF]`,
  `[Dirección completa]`, `[fecha]`, `[plazo de conservación]`.

## Probar en local

No requiere instalación. Basta con abrir `index.html` en el navegador, o
servir la carpeta con cualquier servidor estático, por ejemplo:

```bash
npx serve .
```

## Despliegue: GitHub + Vercel + dominio anexai.es

### 1. Crear el repositorio en GitHub

```bash
cd anexai-web
git init
git add .
git commit -m "Web pública de Anexai"
```

Crea un repo nuevo y vacío en GitHub (sin README ni .gitignore, ya están
aquí), por ejemplo `anexai-web`, y conéctalo:

```bash
git remote add origin https://github.com/<tu-usuario>/anexai-web.git
git branch -M main
git push -u origin main
```

### 2. Conectar el repo a Vercel

1. Entra en [vercel.com](https://vercel.com) e inicia sesión (puedes usar
   tu cuenta de GitHub).
2. **Add New → Project** y selecciona el repositorio `anexai-web`.
3. Framework Preset: **Other** (no hay build step).
   - Build Command: dejar vacío.
   - Output Directory: dejar vacío o `.` (la raíz del repo).
4. Deploy. Vercel te dará una URL provisional del tipo
   `anexai-web.vercel.app` para verificar que todo carga bien.

### 3. Configurar el dominio anexai.es

1. En el proyecto de Vercel: **Settings → Domains → Add**, escribe
   `anexai.es` (y opcionalmente `www.anexai.es`).
2. Vercel te mostrará los registros DNS que hay que crear en el proveedor
   donde compraste el dominio. Normalmente son:

   **Si usas el dominio raíz (`anexai.es`):**
   | Tipo | Nombre | Valor |
   |------|--------|-------|
   | A     | `@`    | `76.76.21.21` |

   **Si usas `www.anexai.es`:**
   | Tipo  | Nombre | Valor |
   |-------|--------|-------|
   | CNAME | `www`  | `cname.vercel-dns.com` |

   > Estos son los valores estándar de Vercel; la propia interfaz de
   > **Settings → Domains** muestra los valores exactos y actualizados
   > para tu proyecto — usa siempre los que te indique ahí.

3. Añade esos registros en el panel DNS de tu registrador de dominio
   (donde compraste `anexai.es`).
4. Espera a la propagación DNS (puede tardar desde minutos hasta unas
   horas) y Vercel emitirá automáticamente el certificado SSL.
5. Si quieres que `www.anexai.es` redirija a `anexai.es`
   (o al revés), márcalo como dominio principal en **Settings → Domains**
   y Vercel configura la redirección automáticamente.

## Requisitos para que Google verifique la app OAuth

Como la app pide scopes de Google Calendar (acceso considerado
**sensible/restringido**), Google exige pasar por un proceso de
verificación antes de poder usarla en producción con usuarios externos.
Esto es lo que hay que tener listo, aparte del código:

### 1. Dominio verificado

- Verificar la propiedad de `anexai.es` en
  [Google Search Console](https://search.google.com/search-console).
- El mismo dominio (o uno de sus subdominios) debe usarse en la home,
  la política de privacidad y añadirse como **Authorized domain** en la
  pantalla de consentimiento OAuth (Google Cloud Console → APIs y
  servicios → Pantalla de consentimiento).

### 2. Pantalla de consentimiento OAuth completa

En Google Cloud Console, sección **OAuth consent screen**:

- Nombre de la app: `Anexai` (debe coincidir con la marca mostrada
  en la web y en el propio flujo de login).
- Logo de la app (120×120 px), el mismo `assets/img/logo.svg`/`.png` de
  esta web, subido en formato PNG/JPG.
- Correo de soporte visible para el usuario.
- **Enlace a la página principal**: `https://anexai.es/`
- **Enlace a la política de privacidad**: `https://anexai.es/privacidad`
- **Enlace a los términos del servicio**: `https://anexai.es/terminos`
- Datos de contacto del desarrollador (email donde Google puede
  escribir sobre el estado de la verificación).

### 3. Página principal (home) que cumpla los requisitos de Google

La home (`index.html`) debe, y ya lo hace:

- Explicar con claridad qué hace la app (ver sección "Cómo funciona").
- Estar alojada en el dominio verificado.
- Enlazar visiblemente a la política de privacidad (footer).
- No pedir login para poder ver esta información (debe ser pública).

### 4. Política de privacidad (`privacidad.html`)

Debe estar publicada, ser accesible sin login, y mencionar
explícitamente (ya está redactado en el archivo):

- Qué datos de Google se solicitan y para qué (sección 3, "Uso de los
  datos de Google").
- Que el uso cumple la
  [Google API Services User Data Policy](https://developers.google.com/terms/api-services-user-data-policy),
  incluido el requisito de **Limited Use**.
- Cómo puede el usuario revocar el acceso.

### 5. Justificación de cada scope solicitado

En el formulario de verificación, Google pide justificar **por qué** se
necesita cada scope pedido (ej. `calendar.events`,
`calendar.readonly`). Hay que explicar en una frase el uso concreto de
cada uno — coincide con lo ya descrito en `privacidad.html` sección 3.
Regla general: pedir solo los scopes estrictamente necesarios (principio
de mínimo privilegio); pedir de más ralentiza o bloquea la aprobación.

### 6. Vídeo de demostración (obligatorio para scopes sensibles/restringidos)

**Esto es lo que falta añadir por tu parte.** Google exige un vídeo que
demuestre el uso real de los scopes solicitados. Requisitos del vídeo:

- Subido a YouTube como **público** o **no listado** (unlisted) — no
  privado, porque el revisor de Google necesita poder abrir el enlace
  sin permisos especiales.
- Debe mostrar, en pantalla y con audio o subtítulos explicando:
  1. El proceso de login/consentimiento OAuth completo, incluida la
     pantalla donde Google muestra los permisos que la app solicita.
  2. Cómo la app usa cada scope pedido en la práctica (por ejemplo:
     mostrar cómo Anexai consulta la disponibilidad real del
     Google Calendar de la clínica y cómo crea/cancela una cita).
  3. Que el flujo termina correctamente (cita creada visible en el
     Calendar real).
- Debe verse claramente la URL/dominio de la app y el nombre tal como
  aparece en la pantalla de consentimiento, para que el revisor pueda
  relacionar vídeo, app y dominio.
- El enlace de YouTube se pega directamente en el formulario de
  verificación de Cloud Console (**OAuth consent screen → Scopes → In
  production → verification**) y, además, conviene enlazarlo también en
  la sección `#video` de `index.html` (el `<iframe>` ya está preparado
  y comentado, solo falta pegar el ID del vídeo).

### 7. Revisión de seguridad (CASA) si aplica

Si el scope de Calendar solicitado es de tipo **restringido** (no solo
sensible) y la app maneja datos de muchos usuarios, Google puede exigir
además una **Cloud Application Security Assessment (CASA)**, un tier de
auditoría de seguridad independiente de pago. Esto solo aplica a partir
de cierto volumen de usuarios/uso — Google lo indica automáticamente en
el propio formulario de verificación si es necesario, no hay que
solicitarlo de antemano.

### 8. Consistencia de marca

El nombre, logo y dominio deben ser **idénticos** en:

- La pantalla de consentimiento OAuth de Google Cloud Console.
- Esta web (`index.html`, `privacidad.html`, `terminos.html`).
- El vídeo de demostración.

Cualquier discrepancia (ej. la app se llama "Anexai" en un sitio y con
otro nombre distinto en otro) es motivo típico de rechazo en la revisión.

### Resumen de lo que queda pendiente por tu parte

- [ ] Verificar `anexai.es` en Google Search Console.
- [ ] Completar la pantalla de consentimiento OAuth en Cloud Console con
      los enlaces de esta web y el logo.
- [ ] Grabar el vídeo de demostración del flujo OAuth + uso de Calendar.
- [ ] Subirlo a YouTube como público o no listado.
- [ ] Pegar el enlace del vídeo en el formulario de verificación y en
      `index.html` (sección `#video`).
- [ ] Rellenar los placeholders legales de `privacidad.html` y
      `terminos.html`.
- [ ] Enviar la app a revisión desde Cloud Console.

### 9. Verificación OAuth de Google — URLs finales

Una vez el dominio esté activo con HTTPS, usa estas URLs en la
configuración de la consent screen de Google Cloud Console:

- Página principal: `https://anexai.es/`
- Política de privacidad: `https://anexai.es/privacidad`
- Términos del servicio: `https://anexai.es/terminos`
