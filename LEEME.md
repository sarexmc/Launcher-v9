# Sarex Studio Launcher 5.1

Minecraft 1.21.1 + Fabric con tus mods en un botón. Pestañas: Inicio, Mods, Skins, Seguridad y Ajustes.

## Crear el instalador (.exe) con GitHub
Sube todo el contenido de esta carpeta al repo (reemplaza los archivos con el mismo nombre) y ejecuta Actions > Crear exe > Run workflow. El instalador sale en Artifacts.

## Probarlo en tu PC
`npm install` y `npm start` (necesitas Node.js LTS).

## Mods
1. Junta los .jar (1.21.1 Fabric) en una carpeta `mods/` y súbelos a un Release de GitHub.
2. `node tools/make-manifest.js ./mods https://github.com/USUARIO/REPO/releases/download/mods/`
3. Firma la lista (ver abajo) y sube `manifest.json` y `manifest.json.sig` al repo.
4. Pon el enlace raw del manifest en `config.json` (`manifestUrl`).

## Logo "Sarex Studio" en vez de Mojang
El launcher instala solo un resource pack con tu logo. Minecraft no deja cambiar esa pantalla sin un mod, así que añade **Puzzle** a tus mods (Fabric 1.21.1): https://modrinth.com/mod/puzzle
Mira en su página qué otros mods pide y mételos también. Si el logo sale deformado o sigue el de Mojang, avísame.

## Firma de la lista de mods (muy recomendado)
Así nadie puede meter mods falsos aunque le roben la cuenta de GitHub a quien administra el repo.
1. Una sola vez: `node tools/sign-manifest.js keygen`. Crea `sarex-private.pem` (guárdalo en un pendrive, NUNCA en GitHub) y muestra una `publicKey`.
2. Pega esa publicKey en `config.json` y crea el instalador de nuevo.
3. Cada vez que cambies mods: `node tools/make-manifest.js ...` y luego `node tools/sign-manifest.js sign`. Sube `manifest.json` y `manifest.json.sig`.
Si la firma no coincide, el launcher no instala nada y avisa en la pestaña Seguridad.

## Skins para premium y sin cuenta
- Mete el mod **CustomSkinLoader** (Fabric 1.21.1): https://modrinth.com/mod/customskinloader
- Cada jugador importa su skin (.png 64x64) en la pestaña Skins, o ve la de cualquier jugador premium por su nombre.
- Para que todos vean la skin de cada amigo, sube los PNG al repo (carpeta `skins/`) y añádelos al manifest:
  `"skins": [ { "player": "Amigo", "url": "https://raw.githubusercontent.com/USUARIO/REPO/main/skins/Amigo.png" } ]`

## Códigos de acceso (150 códigos de 6 dígitos)
Después del EULA, el launcher pide un código de 6 números. Los 150 códigos están en `codigos_acceso_sarexsafio.txt` (privado: reparte uno a cada jugador; no lo subas a GitHub).
- **Modo local (ya funciona así):** el launcher trae las huellas de los códigos y acepta cualquiera de la lista. Es cómodo, pero **no impide** que un mismo código se use en varios PCs, y alguien con conocimientos podría sacar los códigos del programa (son solo 6 dígitos).
- **Modo servidor (recomendado, gratis):** cada código sirve **una sola vez en todo el mundo** y queda ligado al primer PC que lo usa. Pasos: crea una cuenta gratis en https://dash.cloudflare.com, entra a Workers & Pages > Create > Worker y pega `worker/redeem.js`. En Settings > Variables pon `SALT` y `CODES` con los valores de `worker_codes_hashes.txt`. En Storage & Databases > KV crea un namespace y enlázalo al Worker con el nombre `USED`. Copia la URL del Worker a `config.json`, en `codes.url`. Al crear el instalador en GitHub, la lista local de códigos se quita sola.
- Para crear otros códigos: `node tools/make-codes.js 150` (invalida los anteriores).
- "Borrar mis datos" conserva el acceso y el EULA, para que nadie pierda su código.

## Proteger la carpeta de mods (código de administrador)
- Crea tu código (solo tú lo sabrás, 8 o más caracteres, mejor una frase): `node tools/set-admin-pin.js "tu frase secreta"`. Se guarda solo su huella en `config.json`. Después vuelve a crear el instalador.
- Con eso, los botones "Carpeta de mods" y "Carpeta del juego" piden tu código (5 fallos bloquean 15 minutos). Sin código configurado quedan cerrados para todos.
- Mientras el launcher está abierto vigila la carpeta: si alguien borra o cambia un mod, se restaura solo en segundos. Y en cada partida se vuelve a comprobar todo.
- **Límite real:** nadie puede hacer *imposible* borrar archivos en el PC de otra persona; el usuario es dueño de su carpeta. Esto lo hace inútil e incómodo, pero un usuario experto podría usar otro launcher. Para obligar de verdad a tener tu mod, el **servidor** debe comprobarlo (que el servidor expulse a quien no tenga tu mod). Dime y lo programamos en tu mod.

## Edición y logos
- `config.json` tiene `"edition": "1"`: sale en Discord ("Jugando SarexSafio 1"). Cámbialo ahí si algún día cambia.
- Logo del estudio: `src/studio-logo.png` y `src/otter.png`. Ícono de la barra de tareas: `assets/icon.ico`.

## Aviso de Windows ("SmartScreen") y antivirus
El aviso "Windows protegió su PC / editor desconocido" sale con cualquier programa sin firma digital, aunque sea 100% seguro. La única forma de quitarlo del todo es **firmar el instalador** con un certificado de firma de código:
- Gratis para proyectos de código abierto: https://signpath.org/ (el repo debe ser público y tener licencia abierta).
- De pago: certificado de firma de código (por ejemplo Azure Trusted Signing, si tu país lo permite) o de una autoridad como SSL.com o Certum.
Mientras tanto, este instalador ya se instala solo para tu usuario (sin pedir administrador) y trae nombre y datos del programa. Para tus amigos: en el aviso azul pulsan **Más información → Ejecutar de todas formas**, y pueden comprobar la huella SHA-256 (sale en Actions).
Si Defender o un antivirus lo marca por error: súbelo a https://www.virustotal.com y envíalo como falso positivo en https://www.microsoft.com/wdsi/filesubmission.

## Estado de Discord ("Jugando a SarexSafio")
1. Entra a https://discord.com/developers/applications y crea una aplicación llamada **SarexSafio** (ese nombre es el que sale como "Jugando a SarexSafio").
2. En General Information copia el **Application ID** y pégalo en `config.json`, en `discord.clientId`.
3. En Rich Presence > Art Assets sube `discord/logo.png` con el nombre exacto `logo`.
4. Opcional: pon tu invitación (`https://discord.gg/...`) en `discord.invite` y saldrá el botón "Unirse al Discord", dentro del launcher y en tu estado.
Cada jugador puede apagarlo en Ajustes. Solo se comparte el nombre del juego y la hora de inicio. Mientras juegas, el launcher queda en la bandeja de Windows para mantener el estado y se cierra solo cuando cierras Minecraft. Discord debe estar abierto en el PC.

## Fondo y logos
- El fondo es `src/bg.jpg`. Puedes cambiarlo por una captura tuya del mundo (mejor sin logos).
- El logo de la intro y la barra superior es `src/studio-logo.png`; el de la pantalla de carga de Minecraft va dentro de `assets/SarexStudio-Pack.zip`.

## Al abrir Minecraft
El launcher se cierra solo cuando el juego ya abrió (se puede cambiar en Ajustes). Si el juego falla al iniciar, el launcher vuelve a aparecer con un aviso.

## Servidor
En `config.json`, `"server": { "name": "Mi servidor", "host": "ip.o.dominio", "port": 25565 }`. Con eso, JUGAR entra directo al servidor (se puede apagar en Ajustes, "Entrar directo al servidor"). El servidor ya viene configurado para SarexSafio.

## Seguridad y privacidad
- Sesión de Microsoft cifrada con el sistema (DPAPI en Windows). Nunca se guarda ni se muestra contraseña ni token.
- Mods con sha1, Java con SHA-256, descargas solo por https y solo de los sitios de `allowedHosts` en `config.json`.
- Lista de mods firmada (ed25519) y copia local para cuando no hay internet.
- Ventana aislada (sandbox, sin acceso a Node, CSP estricta). Sin telemetría.
- Modo privado: difumina tu nombre y el servidor para grabar o hacer stream.
- Registro seguro: oculta usuario de Windows, IP, correo, ids y tokens antes de compartir un registro.
- Borrar registros y reportes de fallos, y borrar todos tus datos del equipo, con un clic.
- Servidores sin verificación premium: cualquiera puede entrar con el nombre de un premium. Pon **EasyAuth** en el servidor: https://modrinth.com/mod/easyauth
- Las fuentes van incluidas en el launcher: no se conecta a Google ni a nadie para dibujarse.
- Solo la ventana del launcher puede dar órdenes al programa; las herramientas de desarrollo están apagadas en el instalador y solo se abre una copia a la vez.
- Al crear el instalador en GitHub, la huella SHA-256 aparece en el resumen de la ejecución (Actions). Tus amigos pueden comprobarla con `certutil -hashfile Instalador.exe SHA256` antes de instalar.
- Windows puede mostrar un aviso de SmartScreen al instalar porque el instalador no tiene certificado de pago. Pasa el instalador solo por tu enlace oficial.
