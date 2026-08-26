# Ruta y Cuenta — gastos del coche compartido

Web para repartir los gastos del coche entre el grupo que va junto al trabajo:
quién conduce cada día, quién viaja, cuánto cuesta la ida y la vuelta, y quién
le debe a quién. Vive en GitHub Pages y guarda los datos en una base de datos
gratuita de Firebase, así que se actualiza para todo el grupo (con ~4 segundos
de retraso, no es al instante).

**Web:** https://olbaple.github.io/gastos-coche/

## Cómo se usa

1. Al entrar, pide una contraseña compartida (la del grupo, no una personal).
2. Elige tu nombre en el desplegable de arriba ("¿quién eres?") — se recuerda
   en tu móvil/ordenador, no hace falta repetirlo cada vez.
3. Según tu rol, ves y puedes hacer cosas distintas:
   - **Administrador/a**: ve el balance y las cuentas de todo el grupo,
     gestiona quién está en el grupo y qué rol tiene cada uno, puede borrar
     cualquier trayecto o pago, y puede marcar como pagada cualquier fila.
     Para ejercer estos permisos, además de tener el rol asignado, hace
     falta introducir una segunda contraseña (solo la conoce el admin) que
     desbloquea el modo administrador en ese dispositivo.
   - **Conductor/a**: añade trayectos nuevos y puede marcar como pagada una
     deuda solo cuando es a él/ella a quien le deben ese dinero (no puede
     marcar como pagado lo que él mismo debe — eso lo confirma quien cobra).
     Solo ve su propio balance, sus propias cuentas y su propio historial.
   - **Solo lectura**: solo consulta lo suyo (balance, cuentas, historial),
     no puede añadir ni tocar nada.
   - Mientras no haya ningún administrador/a asignado todavía (nadie tiene
     ese rol en "Personas"), cualquiera puede gestionar personas y roles,
     para poder arrancar el grupo la primera vez. En cuanto alguien tiene el
     rol de administrador/a, hace falta la contraseña de admin para poder
     gestionar nada, aunque esa persona todavía no la haya introducido.
4. Un trayecto = un conductor/a para ida y vuelta ese día, con un coste y una
   lista de pasajeros por tramo (pueden cambiar entre ida y vuelta, por si
   alguien se queda a medio camino).
5. "Para saldar cuentas" ya viene compensado por pareja: si en distintos días
   cada uno ha llevado al otro, solo se muestra la diferencia neta entre
   ambos, no dos pagos cruzados.
6. Al pagar de verdad (Bizum, efectivo...), quien puede saldar esa fila pulsa
   "Marcar pagado" — queda anotado en el historial y se descuenta del balance.

**Importante:** esto es una barrera de confianza, no una separación de
cuentas real. Cualquiera con la contraseña compartida puede, en teoría,
elegir el nombre de otra persona en el selector. Vale para un grupo de
compañeros de confianza; no lo uses para nada más sensible.

## Cómo está montado (para quien lo mantenga)

- **Número de versión:** debajo del subtítulo, en pequeño, la web muestra
  "Ruta y Cuenta · vN" (variable `APP_VERSION` en el código). Se sube un
  número cada vez que se entrega una actualización de `index.html`, para
  poder comprobar de un vistazo si ya está desplegada la última versión sin
  tener que mirar el código.

- **`index.html`** es toda la web: una sola página con el HTML, el CSS y el
  JavaScript juntos. No hay build ni dependencias — se edita y se sube tal
  cual.
- **Alojamiento:** GitHub Pages, rama `main`, carpeta raíz. Cualquier commit
  que toque `index.html` se publica solo en 1-2 minutos.
- **Datos compartidos:** Firebase Realtime Database (proyecto `gastos-coche`,
  plan gratuito Spark). Toda la información vive en un único nodo `/state`
  con esta forma:
  ```json
  {
    "people": ["Nombre", "..."],
    "trips": [{ "id", "date", "driver", "legs": { "ida": {...}, "vuelta": {...} } }],
    "lastCost": { "ida": 4, "vuelta": 4 },
    "roles": { "Nombre": "admin" | "conductor" | "usuario" },
    "payments": [{ "id", "from", "to", "amount", "date" }]
  }
  ```
  La web la lee cada 4 segundos (variable `POLL_MS` en el código) y la
  reescribe entera cada vez que alguien guarda un cambio.
- **Acceso:** Firebase Authentication, proveedor "Correo electrónico/
  contraseña", con una única cuenta compartida (`admin@admin.com` + la
  contraseña que se decidió en el grupo — no está guardada en ningún sitio
  del código, se escribe cada vez). Las reglas de la base de datos
  (Realtime Database → Rules) solo dejan leer/escribir a esa cuenta:
  ```json
  {
    "rules": {
      ".read": "auth.uid === '8bCek6ugijXuv1jaQUAoMou6NXX2'",
      ".write": "auth.uid === '8bCek6ugijXuv1jaQUAoMou6NXX2'"
    }
  }
  ```
  El `apiKey` que aparece en el código (`API_KEY`) no es secreto — es el
  identificador público del proyecto de Firebase, la seguridad la dan estas
  reglas, no ese valor.

### Cambiar la contraseña compartida

Firebase → Authentication → pestaña "Users" → los tres puntos junto al
usuario → cambiar contraseña. No hace falta tocar el código.

### Contraseña de administrador

Es una segunda clave, distinta de la contraseña compartida de arriba, que
desbloquea el rol de administrador/a en el dispositivo desde el que se
introduce (se guarda en ese navegador, no hay que repetirlo cada vez). Vive
en el propio código (`ADMIN_PASSWORD`), no en Firebase — es una barrera de
confianza, no una comprobación real del servidor: cualquiera con acceso al
código fuente de la página podría leerla. Para cambiarla, hay que editar esa
constante en `index.html` y volver a subir el archivo.



### Añadir a alguien nuevo al grupo

Basta con que un administrador/a lo añada desde la sección "Personas" de la
propia web — no requiere tocar Firebase ni el código.

### Limitaciones conocidas

- La sincronización es por sondeo cada 4 segundos, no instantánea.
- El modo de prueba original de Firebase caducaba a los 30 días; ya no aplica
  porque las reglas de arriba son permanentes (no tienen fecha de caducidad).
- No hay recuperación de contraseña por email real (el correo de la cuenta no
  existe de verdad) — si se pierde la contraseña, hay que cambiarla a mano
  desde la consola de Firebase como se explica arriba.
