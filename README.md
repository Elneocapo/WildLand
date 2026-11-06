# Wildlands 0.27 · V24 (GitHub Pages)

## Qué subir a GitHub

**Obligatorio: `index.html` en la raíz del repositorio.** Es el juego completo. No subas el ZIP dentro del repositorio esperando que GitHub Pages lo descomprima: Pages no lo hace.

Opcional: `README.md` para conservar estas instrucciones. Puedes guardar el ZIP y el HTML con nombre de versión en tu PC como copias de seguridad.

## Publicarlo en GitHub Pages

1. Descarga `index.html` del paquete y crea un repositorio en GitHub, por ejemplo `wildlands`.
2. Sube `index.html` directamente a la carpeta principal del repositorio (no dentro de otra carpeta).
3. Abre **Settings → Pages**.
4. En **Build and deployment**, selecciona **Deploy from a branch**.
5. Selecciona la rama `main` y la carpeta `/(root)`, y pulsa **Save**.
6. Espera a que GitHub muestre el enlace publicado en esa pantalla. Será parecido a `https://TU-USUARIO.github.io/wildlands/`.
7. Abre ese enlace con Chrome o Edge actualizado. GitHub Pages sirve la web por HTTPS; no hay que ejecutar un servidor local ni instalar nada.

GitHub busca `index.html` en la raíz de la carpeta publicada. La documentación oficial explica el origen de publicación por rama y carpeta: https://docs.github.com/es/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site

## Cómo probarlo

### 1. Prueba individual

1. Abre la URL de GitHub Pages.
2. Pulsa **JUGAR SOLO**.
3. Prueba moverte, mirar, saltar, abrir el inventario, recoger materiales, talar/romper un recurso y colocar una estructura.
4. Comprueba que no aparece ningún error de carga en pantalla.

### 2. Prueba multijugador básica

1. Abre la URL en dos navegadores o dispositivos distintos (no basta con dos pestañas del mismo navegador para una prueba realista de red).
2. En el primero, escribe un nombre y pulsa **CREAR PARTIDA**. Espera a que aparezca un código.
3. En el segundo, pulsa **UNIRSE A PARTIDA**, introduce el código y pulsa **CONECTAR**.
4. Cuando ambos indiquen que están conectados, pulsa **ENTRAR AL MUNDO** en cada uno.
5. Comprueba que se ven los dos jugadores y que las posiciones cambian al caminar.
6. Prueba una acción compartida: talar el mismo árbol, colocar una pieza, abrir/cerrar una puerta, abrir un cofre o dormir en la misma hoguera.

### 3. Prueba específica de Educacyl / instituto

Haz exactamente la misma prueba de dos dispositivos desde la Wi-Fi de alumnado del instituto. Anota si falla al crear sala, al conectar al anfitrión o después de entrar. Luego repite desde una red doméstica o un punto de acceso móvil (solo si está permitido por las normas del centro).

Interpretación rápida:
- **La página ni siquiera carga:** filtro web, DNS o CDN (GitHub Pages/Three.js/PeerJS).
- **No se crea la sala / señalización no disponible:** revisa acceso WSS a `0.peerjs.com` por TCP 443.
- **La sala se crea, pero el invitado no entra y aparece “P2P / ICE”:** el problema está después de señalización; probable filtro UDP/WebRTC, NAT o aislamiento entre clientes.
- **En casa funciona y en Educacyl no:** evidencia fuerte de una restricción de esa red/perfil, no de un fallo general del juego.
- **En la misma Wi-Fi falla pero entre redes distintas funciona:** puede existir aislamiento entre clientes de la Wi-Fi del centro.

Importante: que GitHub Pages se abra no demuestra que WebRTC esté permitido. El juego usa un canal de datos directo de navegador a navegador.

## Red: puertos y dominios que usa esta versión

| Servicio | Destino | Uso |
|---|---|---|
| Web del juego | Dominio `*.github.io` o dominio Pages propio, TCP 443 | Descargar `index.html` por HTTPS |
| Three.js | `cdnjs.cloudflare.com`, TCP 443 | Motor 3D |
| PeerJS | `cdn.jsdelivr.net`, TCP 443 | Librería de multijugador |
| PeerServer Cloud | `0.peerjs.com`, WSS por TCP 443 | Señalización: registrar peers y negociar el inicio de la conexión |
| STUN | `stun.l.google.com`, UDP 19302 | Descubrir direcciones/rutas de conexión |
| WebRTC DataChannel | Entre los dos navegadores, UDP/puertos dinámicos según ICE/NAT | Tráfico real de la partida |
| Fuentes (opcionales) | `fonts.googleapis.com` y `fonts.gstatic.com`, TCP 443 | Tipografía; si se bloquean, se usa una fuente de reserva |

No hace falta abrir puertos entrantes manualmente en el PC del alumno. Pero sí tiene que estar permitido el tráfico saliente y el tráfico de respuesta necesario para WebRTC. V24 **no incluye TURN**: si no se puede crear un canal P2P, no hay retransmisión alternativa por TCP 443.

### Lo que he encontrado públicamente sobre RCE / Educacyl

He encontrado un documento especialmente pertinente: el **Plan Digital del IES Camino de Santiago (Burgos), curso 2023–2024**, publicado en la red de centros `centros.educa.jcyl.es`. En el apartado de infraestructura describe la Red de Centros Educativos (RCE) como gestionada centralmente y especifica que, en esa configuración, los únicos puertos habilitados por defecto son **TCP 80 y TCP 443**. También dice que una apertura adicional debe justificarse y tramitarse a través del CAU. El documento identifica `CED_INTERNET` como el perfil general destinado también al alumnado y diferencia redes/perfiles de docencia, administración e invitados.

Esto **no demuestra que todos los institutos ni todos los perfiles de alumnado tengan hoy exactamente la misma ACL**, porque el plan describe un centro concreto y una arquitectura documentada para ese periodo; pueden existir cambios por sede, VLAN, perfil o actualización posterior. Pero es un indicio muy fuerte y además procede de un IES: si el centro usa esa política RCE, UDP 19302 y las conexiones UDP dinámicas que necesita WebRTC probablemente no estarán permitidos por defecto.

Documento público: https://iescaminodesantiago.centros.educa.jcyl.es/sitio/upload/Plan_Digital_09001694_IES_CAMINO_DE_SANTIAGO.pdf (apartado B.7, página impresa 47 y siguientes).

La información pública no permite consultar la lista exacta de reglas activas del firewall de tu instituto. Para confirmarlo, el coordinador TIC o la dirección puede elevar la consulta al CAU/CORS de la red educativa. No intentes saltarte el filtro: pide que confirmen si permiten WebRTC/STUN o si existe una red de prácticas con esa salida autorizada.

## Guía del código para continuar en otro chat

El HTML contiene comentarios de arquitectura y mantenimiento. Resumen de los puntos clave:

- **Motor Three.js:** toda la partida local: mundo, jugador, cámara, físicas, inventario, recursos, construcción, supervivencia, combate, enemigos y renderizado.
- **Semilla procedural:** los jugadores de una sala calculan la misma semilla a partir del código para que el mapa base coincida.
- **`window.WildlandsGame`:** API puente entre motor y adaptador de red. Contiene getters, funciones para aplicar estado/eventos, avatares remotos y transporte.
- **`window.WildlandsMP`:** estado de rol/protocolo del multijugador.
- **`mpSnapshot()` / `getMPWorldState()`:** convierten el mundo en objetos planos aptos para JSON.
- **`applyMPWorldState()` / `applyMPWorldEvent()`:** aplican snapshots o cambios puntuales en clientes invitados.
- **`handleMPWorldAction()`:** validación autoritativa de acciones que llegan al anfitrión. Mantener validaciones de distancias, herramientas, recursos y estado.
- **Último script (PeerJS):** crea/une salas, maneja `hello`, `welcome`, `state`, `world-action`, `world-rejected`, `world-event`, `snapshot`, `player-damage` y coordina el envío periódico.
- **`MP_PROTOCOL_VERSION` y `MP_STATE_SCHEMA_VERSION`:** identificadores del formato de datos. Revisar al modificar el protocolo.

### Reglas para futuras modificaciones

1. No enviar objetos Three.js por red; enviar solo datos planos serializables.
2. Las acciones de invitados se validan siempre en el anfitrión.
3. El anfitrión simula la IA; los invitados interpolan los estados recibidos.
4. Si cambias una estructura de paquete, revisa compatibilidad y versión del protocolo.
5. Después de cada cambio, probar modo solo y dos equipos en red real.
6. Mantener IDs HTML que utiliza el JavaScript y el contrato público de `window.WildlandsGame`.

## Limitación conocida de V24

La detección ICE ayuda a distinguir una señalización disponible de un P2P fallido, pero el navegador no puede saber siempre qué regla del firewall lo bloqueó. Además, sin TURN, una red que permita solamente TCP 443 puede cargar la página y PeerJS, pero no garantizar el canal de juego.
