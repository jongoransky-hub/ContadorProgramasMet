# Programas de mano · Teatro Metropolitan

App para saber cuántos programas de mano quedan de cada obra, para cuánto alcanzan y cuándo hay que reimprimir.

Son dos archivos:

- `index.html`: la app. Va a GitHub Pages.
- `Code.gs`: el motor. Va pegado en el Apps Script de un Google Sheet, que es donde se guardan los datos.

## Las dos entradas

| Quién | Dirección | Qué ve |
|---|---|---|
| Gestión (Luli, Mano, Carla y Jon) | la dirección de la app | Tablero, Aprobar y Obras. Los cambios piden la clave de gestión. |
| Asistentes de sala | la misma dirección terminada en `#sala` | Solo la pantalla de conteo. No pide clave. |

Lo que envían las asistentes queda pendiente y no mueve el tablero hasta que alguien de gestión lo aprueba.

## Instalación

### 1. El Google Sheet y el script (10 minutos)

1. Creá un Google Sheet nuevo, vacío. Ponele un nombre, por ejemplo "Programas de mano MET".
2. En el menú: **Extensiones > Apps Script**.
3. Borrá lo que aparece en el editor y pegá el contenido completo de `Code.gs`. Guardá.
4. Arriba, en el selector de funciones, elegí **setup** y tocá **Ejecutar**.
5. Google pide permisos la primera vez (leer el Sheet, mandar mails en tu nombre, programar una revisión diaria). Aceptalos. Si aparece "Google no verificó esta app", tocá **Configuración avanzada** y después **Ir al proyecto**.
6. Volvé al Sheet: tienen que haber aparecido las pestañas Obras, Conteos, Reimpresiones, Config y Avisos, con las nueve obras y los conteos de la planilla anterior ya cargados.

### 2. La pestaña Config

Los mails ya vienen cargados:

| clave | Valor |
|---|---|
| `mail_gestion` | Jon, Mano y Carla. Reciben el aviso de reimpresión. |
| `mail_conteo` | Luli. Recibe el recordatorio de obras sin contar. |

Para cambiar un destinatario alcanza con editar la celda; no hay que tocar el código.

Falta completar la columna **valor** de estas dos filas:

| clave | Qué poner |
|---|---|
| `clave_acceso` | La clave de gestión, la que van a usar Luli, Mano, Carla y Jon. |
| `url_app` | La dirección de GitHub Pages. Se completa al final (paso 4). |

Y revisar esta, que define hasta cuándo se calcula todo:

| clave | Valor |
|---|---|
| `fin_temporada` | `2026-11-29`. Último día de la temporada. Al armar la temporada siguiente se cambia esta fecha y nada más. |

Las demás filas ya vienen con los valores acordados: 1 semana de actualización, 2 de imprenta, recordatorio a los 10 días sin conteo.

Para comprobar que los mails salen: en el editor del script, elegí la función **probarMails** y tocá **Ejecutar**. Tiene que llegar un mail de prueba a cada casilla.

### 3. Publicar el script

1. En el editor del script: **Implementar > Nueva implementación**.
2. En el engranaje, elegí el tipo **Aplicación web**.
3. **Ejecutar como:** yo. **Quién tiene acceso:** cualquier usuario.
4. Tocá **Implementar** y copiá la **URL de la aplicación web**, la que termina en `/exec`.

### 4. La app en GitHub (10 minutos)

1. Abrí `index.html` con un editor de texto y buscá esta línea, cerca del principio del script:

   ```js
   const API_URL = '';
   ```

   Pegá la URL del paso anterior entre las comillas y guardá.

2. En GitHub, creá un repositorio nuevo y **público** (GitHub Pages gratis lo exige). Por ejemplo `programas-met`.
3. Subí `index.html` al repositorio (**Add file > Upload files**). Subí solo ese archivo: `Code.gs` tiene los mails del equipo y no tiene que quedar en un repositorio público.
4. En el repositorio: **Settings > Pages**. En **Branch** elegí `main` y la carpeta `/ (root)`. Guardá.
5. En uno o dos minutos GitHub muestra la dirección, del tipo `https://TU-USUARIO.github.io/programas-met/`.
6. Copiá esa dirección en `url_app`, en la pestaña Config del Sheet.

### 5. Probar el circuito

1. Abrí la dirección terminada en `#sala`, poné un nombre, anotá un número de prueba en una obra y enviá.
2. Abrí la dirección sin `#sala`, entrá a **Aprobar**: el conteo de prueba tiene que estar ahí. Descartalo (va a pedir la clave de gestión).
3. Mirá la pestaña Conteos del Sheet: la fila de prueba quedó como `descartado`.

Si todo eso funciona, ya se puede repartir la dirección `#sala` a las asistentes.

## Cómo funcionan los avisos

El script revisa solo, todos los días a las 9 de la mañana, y también después de cada aprobación, corrección o reimpresión.

- **Reimprimir pronto:** sale cuando a una obra le quedan 6 semanas de programas. Va a `mail_gestion`.
- **Reimprimir ya:** sale cuando le quedan 4 semanas, es decir, una semana antes del último día para empezar. Va a `mail_gestion`.
- **Falta conteo:** sale cuando una obra en cartel lleva más de 10 días sin un conteo aprobado. Va a `mail_conteo` y se repite cada 7 días mientras siga sin contarse.

Cada aviso de reimpresión sale una sola vez por obra y por nivel. El mail trae el calendario (actualización y imprenta), la cantidad sugerida y un párrafo para comercial, así se puede reenviar tal cual.

En la pestaña Avisos queda el registro de todo lo que se mandó.

## Cosas a saber

- **La cuenta, función por función.** Cada obra tiene su calendario: días de la semana con función, fechas sin función, estreno y última función. El consumo por función sale de dividir los programas usados entre dos conteos aprobados por las funciones que hubo en el medio (últimos cuatro períodos). Después la app recorre las funciones que quedan y marca la primera que se queda sin programas. Con un solo conteo no hay proyección.
- **Obras sin días cargados.** Si una obra no tiene días de función marcados, el consumo se reparte parejo por día. Sirve como aproximación para las que tienen cuatro o cinco funciones por semana.
- **Cuándo contar.** Un conteo hecho el día de una función se toma como previo a esa función. Conviene contar siempre antes de la función.
- **Función cubierta.** Una función cuenta como cubierta si queda al menos la mitad de los programas que suele usar.
- **Cuánto pedir.** Es el consumo por función multiplicado por las funciones que van desde que llega la reimpresión hasta la última función de la obra (o hasta el fin de temporada, lo que llegue antes), redondeado para arriba a la centena. Si el stock alcanza, no se pide nada. Nunca calcula más allá de `fin_temporada`: una obra que sigue al año siguiente va con programa nuevo (logos, auspicios y ficha técnica).
- **Si una reimpresión ya no llega.** Cuando las tres semanas de plazo terminan después de la última función, la app lo dice y no sugiere reimprimir.
- **Reimpresiones.** Cada tanda que llega de la imprenta hay que registrarla en Aprobar > Reimpresión recibida. Si no, el próximo conteo aparece como un stock que subió solo.
- **Correcciones.** En el tablero, tocá una obra y después **Corregir** en el conteo que quieras cambiar. Se puede cambiar el número, la fecha o eliminarlo. Nada se borra del Sheet: un conteo eliminado queda como `descartado`.
- **Obras.** Altas, días de función, fechas sin función, estreno o regreso y última función se editan en la pestaña Obras de la app.
- **Calendario 2026.** La función `cargarTemporada2026` del script carga de una vez las últimas funciones, los días y las pausas de la temporada. Se ejecuta a mano una sola vez desde el editor del script.
- **Qué queda público.** El repositorio es público y la dirección del script se ve en el código. Cualquiera que la tenga puede leer el stock y enviar un conteo pendiente. Aprobar, corregir y editar piden la clave. La clave y los mails nunca salen del Sheet.
- **Si se cambia `Code.gs`:** Implementar > Gestionar implementaciones > editar > **Nueva versión**. La URL no cambia.
