# Ruta y Cuenta — gastos del coche compartido

Web para repartir los gastos del coche entre el grupo que va junto al trabajo:
quién conduce cada día, quién viaja, cuánto cuesta la ida y la vuelta, y quién
le debe a quién. Vive en GitHub Pages y guarda los datos en una base de datos
gratuita de Firebase, así que se actualiza para todo el grupo (con ~4 segundos
de retraso, no es al instante).

**Web:** https://olbaple.github.io/gastos-coche/ (con dominio propio:
https://pablillo.com/gastos-coche/)

## Cómo se usa

1. Al entrar, pide una contraseña compartida (la del grupo, no una personal).
2. Elige tu nombre en el desplegable de arriba ("¿quién eres?") — se recuerda
   en tu móvil/ordenador, no hace falta repetirlo cada vez.
3. Según tu rol, ves y puedes hacer cosas distintas:
   - **Administrador/a**: ve el balance y las cuentas de todo el grupo,
     gestiona quién está en el grupo, sus roles y sus nombres, puede borrar
     cualquier trayecto o pago, y puede marcar como pagada cualquier fila.
     Para ejercer estos permisos, además de tener el rol asignado, hace
     falta introducir una segunda contraseña (solo la conoce el admin) que
     desbloquea el modo administrador en ese dispositivo.
   - **Conductor/a**: añade trayectos nuevos, puede borrar los trayectos en
     los que él/ella fue quien condujo (no los de otros), y puede marcar
     como pagada una deuda solo cuando es a él/ella a quien le deben ese
     dinero (no puede marcar como pagado lo que él mismo debe — eso lo
     confirma quien cobra). Solo ve su propio balance, sus propias cuentas y
     su propio historial.
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
5. El **Balance** de cada persona se desglosa en dos cifras: cuánto le deben
   (por haber conducido) y cuánto debe (por haber ido de pasajero) — no un
   único número neto, porque una misma persona puede ser las dos cosas a la
   vez en días distintos (conducir unos días, ir de pasajera otros).
6. "Para saldar cuentas" sí va compensado por pareja: si en distintos días
   cada uno ha llevado al otro, solo se muestra la diferencia neta entre
   ambos, no dos pagos cruzados.
7. Al pagar de verdad (Bizum, efectivo...), quien puede saldar esa fila pulsa
   "Marcar pagado" — queda anotado en el historial y se descuenta del balance.
8. El historial se pagina de 10 en 10, con números de página abajo (como en
   cualquier buscador), no todo junto.

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
- **Alojamiento:** GitHub Pages, rama `main`, carpeta raíz, con un dominio
  propio (pablillo.com) apuntando ahí. Cualquier commit que toque
  `index.html` se publica solo en 1-2 minutos.
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
  **Ojo:** si alguna vez Realtime Database vuelve a mostrar las reglas de
  "modo de prueba" (con un comentario tipo `// 2026-x-x` y una fecha), esas
  reglas **caducan solas a los 30 días** y desde ese momento nadie puede
  entrar aunque la contraseña sea correcta. Hay que sustituirlas por las de
  arriba y pulsar "Publish" — esto ya nos pasó una vez (24 de septiembre de
  2026) y el síntoma fue un error de "no se ha podido conectar" al hacer
  login que en realidad era esto.

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

### Renombrar a alguien

El administrador/a tiene un botón con un lápiz junto a cada nombre en
"Personas". Actualiza el nombre en la lista de personas y también en todos
los trayectos y pagos ya guardados (no rompe el historial ni el balance).

### Diagnosticar un fallo de login

Desde v6, la web distingue en el mensaje de error si la contraseña está mal
o si el problema es de conexión con Firebase (`identitytoolkit.googleapis.com`
/ `securetoken.googleapis.com`) — por ejemplo, un bloqueador de anuncios,
una VPN o una red que filtre dominios de Google pueden dar ese segundo caso.
Si el mensaje de conexión aparece incluso en redes distintas (wifi y datos
móviles), lo primero a revisar es que las reglas de Realtime Database no
hayan caducado (ver el aviso más arriba) antes de sospechar del dispositivo.

### Limitaciones conocidas

- La sincronización es por sondeo cada 4 segundos, no instantánea.
- No hay recuperación de contraseña por email real (el correo de la cuenta no
  existe de verdad) — si se pierde la contraseña, hay que cambiarla a mano
  desde la consola de Firebase como se explica arriba.
