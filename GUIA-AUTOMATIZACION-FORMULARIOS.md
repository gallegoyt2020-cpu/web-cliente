# Guía: Automatización de los formularios del sitio

El sitio web ya está listo técnicamente: cada vez que alguien llena el
formulario (home o página de Contacto), el sitio envía automáticamente
los datos a un "Webhook" (una URL receptora). Falta un solo paso:
**crear las cuentas** en las herramientas que van a recibir esos datos
y conectar todo con Zapier (sin necesidad de programar).

Flujo que vamos a armar:

```
Formulario del sitio
   -> Zapier (recibe el envío)
        -> Twilio          (manda el SMS automático al cliente potencial)
        -> Klaviyo         (agrega el email a la secuencia de correos)
        -> HubSpot         (guarda el contacto como registro del CRM)
```

Ninguna de las 4 cuentas existe todavía, así que hay que crearlas en este orden.

---

## Paso 1 — Crear cuenta en Zapier (el conector)

1. Ir a https://zapier.com y crear una cuenta gratuita (con el correo del negocio).
2. El plan gratuito alcanza para empezar (hasta 100 tareas/mes); si el
   volumen de formularios crece, se sube de plan más adelante.
3. Dentro de Zapier, crear un "Zap" nuevo.
4. Como **Trigger** (disparador), elegir la app **"Webhooks by Zapier"** →
   evento **"Catch Hook"**.
5. Zapier va a generar una URL única, algo como:
   `https://hooks.zapier.com/hooks/catch/1234567/abcd123/`
6. **Copiar esa URL** — la vamos a pegar en el archivo `index.html` del sitio
   (te explico dónde exactamente en el Paso 5).

## Paso 2 — Crear cuenta de Twilio (SMS automático)

1. Ir a https://www.twilio.com/try-twilio y crear una cuenta.
2. Comprar/activar un número de teléfono de EE. UU. dentro de Twilio
   (tiene costo mensual bajo, aprox. $1/mes + costo por SMS enviado).
3. Anotar: **Account SID**, **Auth Token** (están en el Dashboard de Twilio)
   y el **número de Twilio** comprado.
4. En Zapier, agregar una acción nueva dentro del mismo Zap → app **Twilio**
   → evento **"Send SMS"**.
5. Conectar la cuenta de Twilio con el Account SID / Auth Token.
6. Configurar el mensaje, por ejemplo:
   `Hola {{name}}, gracias por contactar a Four Seasons Construction. Te llamaremos pronto al {{phone}}.`
   (Los campos entre `{{ }}` los llena Zapier automáticamente con los datos
   del formulario: nombre, teléfono, email, etc.)
7. El destinatario del SMS es el campo `phone` que llega del formulario.

## Paso 3 — Crear cuenta de Klaviyo (secuencia de email)

1. Ir a https://www.klaviyo.com y crear cuenta gratuita (el plan gratis
   alcanza para varios miles de contactos, más que suficiente para empezar).
2. Dentro de Klaviyo, crear una **Lista** (ej. "Leads del sitio web") y un
   **Flow** (secuencia automática) que se dispare cuando alguien se une a
   esa lista — ahí se diseñan los 2-3 correos de seguimiento.
3. En Zapier, agregar otra acción en el mismo Zap → app **Klaviyo** →
   evento **"Subscribe Profile to List"** (o "Update Profile" según la
   versión disponible).
4. Conectar la cuenta con la API Key de Klaviyo (se genera dentro de
   Klaviyo en Settings → API Keys).
5. Mapear los campos: `email`, `name`, `phone` del formulario hacia el
   perfil de Klaviyo, y seleccionar la lista creada en el punto 2.

## Paso 4 — Crear cuenta de HubSpot (CRM)

1. Ir a https://www.hubspot.com/products/crm y crear la cuenta gratuita
   de CRM (el CRM base de HubSpot es gratis para siempre).
2. En Zapier, agregar una tercera acción en el mismo Zap → app **HubSpot**
   → evento **"Create or Update Contact"**.
3. Conectar la cuenta (Zapier pedirá iniciar sesión con la cuenta de
   HubSpot y autorizar el acceso).
4. Mapear los campos del formulario a las propiedades del contacto en
   HubSpot: `name` → Nombre, `email` → Email, `phone` → Teléfono,
   `interest` → una propiedad personalizada tipo "Interés" (se puede crear
   dentro de HubSpot: Settings → Properties → Create property), y
   `message` → Nota o propiedad "Mensaje".

## Paso 5 — Conectar la URL del Webhook al sitio

Una vez que tengas la URL del Paso 1 (la de Zapier), avísame o edítala tú
mismo así:

1. Abrir `index.html` con un editor de texto.
2. Buscar esta línea (aprox. cerca de la función `paintBrandNames`):
   ```js
   const FORM_WEBHOOK_URL='https://hooks.zapier.com/hooks/catch/REEMPLAZAR/CON-TU-URL/';
   ```
3. Reemplazar esa URL de ejemplo por la URL real que copiaste en el Paso 1.
4. Guardar, subir el archivo actualizado a Hostinger/Netlify.

## Paso 6 — Probar

1. Activar el Zap ("Turn on Zap" en Zapier).
2. Llenar el formulario real del sitio (con un número/correo tuyo de prueba).
3. Verificar que:
   - Llega el SMS a tu teléfono.
   - Tu email de prueba entra a la lista/flow de Klaviyo.
   - Aparece el contacto nuevo en HubSpot.
   - Zapier muestra el "Zap history" con la corrida exitosa (History → Zap Runs).

---

### Notas importantes

- Todo el trabajo de "detrás de cámaras" (Twilio, Klaviyo, HubSpot) vive
  dentro de Zapier — el sitio web nunca guarda ni expone esas claves
  secretas, así que es seguro.
- Si en el futuro cambian de opinión (por ejemplo, Klaviyo por otra
  herramienta de email, o Make.com en lugar de Zapier), **no hay que tocar
  el sitio web** — solo se reconfigura el Zap/Scenario, porque el sitio
  solo manda los datos a una URL fija.
- Mientras el Paso 1-5 no esté hecho, el formulario del sitio sigue
  funcionando (muestra el mensaje de "Thank You!" al visitante), pero los
  datos no llegan a ningún lado todavía — por eso es importante completar
  esta guía antes de lanzar el sitio en producción.
