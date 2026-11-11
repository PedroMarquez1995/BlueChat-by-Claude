BlueChat
Mensajería Bluetooth Low Energy con red Mesh propia, sin Internet, sin Wi-Fi, sin datos móviles, sin SIM, sin cuentas y sin servidores. Android nativo (Kotlin, Jetpack Compose, Material 3). com.bluechat.mesh · versión 1.0.0 · minSdk 26.
Estado real de verificación (léelo primero)


Componente
Estado
Motor Mesh (protocolo, fragmentación, enrutamiento, store-and-forward, ACK, cifrado)
Compilado y probado en JVM: 40 pruebas automatizadas con red simulada, todas OK.
Capa Android (BLE/GATT, servicio, Room, Compose)
Escrita pero NO compilada ni ejecutada por quien generó el proyecto (el entorno no tenía Android SDK ni red). La primera compilación real será la de GitHub Actions. Puede haber errores de compilación que haya que corregir.
Mesh sobre Bluetooth real entre 3 teléfonos
No probado. Solo puede verificarse con teléfonos reales (ver docs/TESTING.md). Las pruebas simuladas no demuestran que la malla funcione sobre hardware real.


APK
No se ha generado ninguna APK todavía. Se genera al ejecutar el workflow.
Si el workflow falla, copia el error del registro (paso «Compilar APK debug») y se corrige.
Qué hace
Perfil local (nombre, avatar) e identidad criptográfica propia (Ed25519 + X25519). El ID de usuario es un hash de las claves públicas: no depende de teléfono, MAC ni hardware.
Descubre otros teléfonos BlueChat por BLE (anuncio + escaneo por UUID de servicio) y se conecta por GATT.
Mensajes de texto cifrados de extremo a extremo y firmados.
Retransmisión multi-salto: un teléfono intermedio guarda y reenvía el mensaje cifrado (Store-and-Forward) hasta que llega al destinatario o caduca.
Confirmaciones reales: «Entregado» solo cuando el destinatario devuelve un ACK firmado; «Leído» cuando abre el chat (configurable).


Historial local (Room), notificaciones locales, servicio en primer plano connectedDevice.
Pantallas: bienvenida, perfil, permisos, inicio/chats, cercanos, malla, chat, contacto/verificación, ajustes. Modo claro/oscuro.
Qué NO hace (limitaciones conocidas)
Sin secreto hacia adelante: el cifrado usa claves estáticas X25519; si una clave privada se compromete, mensajes capturados antes podrían descifrarse.
Metadatos visibles para repetidores: origen, destino, tamaño, hora y saltos de cada paquete van en claro (los necesitan para reenviar). Los HELLO publican alias, avatar y claves públicas a quien esté cerca y visible.
Primer contacto no autenticado (TOFU): hasta que compares la huella en persona, un atacante cercano podría suplantar a un contacto durante el primer intercambio. Se ofrece verificación por huella de 60 dígitos. La lectura/generación de códigos QR NO está implementada en 1.0.0.


Las claves privadas se cifran con una clave AES-GCM del Android Keystore, pero Keystore no ofrece Ed25519/X25519 en API 26–32, así que están en memoria mientras la app corre. Un teléfono con root las puede extraer. Si desinstalas la app o borras sus datos, la identidad se pierde (sin respaldo, a propósito).
Android puede detener la app en segundo plano (ahorro de batería de Xiaomi, Huawei, Samsung, OPPO…). START_STICKY y la cola en disco mitigan, pero no garantizan operación continua. Excluye BlueChat de la optimización de batería.
Hay que tener la malla activada (servicio en primer plano) para recibir. Android 12+ puede impedir que el servicio se inicie desde segundo plano; se arranca desde la app.
Alcance BLE: decenas de metros por salto (muy variable). Rendimiento: unos pocos KB/s; solo texto corto (≤ 600 bytes). Sin archivos, voz ni grupos.


Conexiones GATT simultáneas limitadas (por defecto 5); algunos teléfonos con pila BLE antigua son poco fiables.
No se muestra distancia: el RSSI solo se traduce a «fuerte/media/débil».
Las cadenas de la interfaz están en español (no hay traducción).
Anuncio BLE no soportado en algunos teléfonos: funcionan solo como escáner (se avisa en pantalla).
Los ACK necesitan una ruta de vuelta; si no existe, el emisor verá «En retransmisión · sin confirmar» aunque el mensaje haya llegado.


Requisitos
Android 8.0+ (API 26) con Bluetooth Low Energy. Para que un teléfono sea visible necesita soporte de BLE advertising (la mayoría desde 2016).
Permisos: Android 12+: «Dispositivos cercanos» (escanear, conectar, anunciar) y notificaciones (13+). Android 8–11: ubicación (exigida por Android solo para escanear BLE; no se usa el GPS).
No se solicita el permiso INTERNET (se elimina explícitamente del manifiesto fusionado).
Compilar la APK con GitHub Actions (sin instalar nada)
Crea un repositorio en GitHub (puede ser privado): New repository.
Sube el contenido de la carpeta BlueChat (no el ZIP en sí): en el repositorio vacío pulsa uploading an existing file, arrastra TODO el contenido descomprimido (incluida la carpeta oculta .github) y pulsa Commit changes. Asegúrate de que en la raíz del repositorio queden app/, gradle/, gradlew, settings.gradle.kts… y .github/workflows/build.yml.
Si el navegador no sube carpetas ocultas (.github), usa Add file → Create new file, escribe .github/workflows/build.yml como nombre y pega el contenido del archivo.
Asegúrate de subir gradle/wrapper/gradle-wrapper.jar (binario).
Abre la pestaña Actions. Si GitHub pide habilitar workflows, acéptalo.
Elige Build BlueChat APK en la lista izquierda → Run workflow → rama main → Run workflow. (También se ejecuta solo con cada push a main/master.)


Espera 5–10 minutos. Si termina en verde ✅, abre la ejecución y, abajo en Artifacts, descarga BlueChat-Debug-APK (ZIP que contiene BlueChat-debug.apk). Si falla ❌, abre el paso en rojo y copia el error.
El workflow ejecuta antes las pruebas unitarias y publica también BlueChat-Test-Reports.
El workflow usa la firma debug automática de Android Gradle Plugin; no necesita claves ni secretos. El gradlew recibe chmod +x en el workflow por si se perdió el permiso al subir un ZIP, y el wrapper se valida con gradle/actions/wrapper-validation.


Instalar en Android
Descomprime el ZIP del artefacto en el teléfono (o en el PC y pásalo por cable).
Abre BlueChat-debug.apk; Android pedirá permitir instalar desde esa fuente (Ajustes → Instalar apps desconocidas) → permite y confirma.
Abre BlueChat → Comenzar → nombre y avatar → Crear identidad → Conceder permisos → Continuar.
En Inicio, si aparece el aviso, pulsa Activar malla. Verás la notificación persistente «BlueChat activo».
Prueba con tres teléfonos (A–B–C)
Ver docs/TESTING.md (procedimiento completo y qué anotar). Resumen: instala en A, B, C; activa la malla en los tres; coloca A y B cerca y C fuera del alcance de A pero cerca de B; espera a que A vea a C en «Cercanos» como «Alcanzable vía malla · 2 saltos»; envía de A a C y comprueba que el estado pasa a «En retransmisión» y, cuando C responde, a «Entregado».


Estructura
app/src/main/java/com/bluechat/mesh/
  core/        Motor Mesh sin dependencias de Android (probado en JVM)
    MeshPacket, Fragmenter, MeshRouter, MeshEngine, MeshStore, MeshTransport, CryptoManager, MeshModels
  bluetooth/   BleSupport, BluetoothDiscoveryManager (anuncio+escaneo), BluetoothGattManager (servidor+cliente GATT)
  data/        Room (Database, RoomMeshStore), IdentityManager (Keystore), SettingsRepository (DataStore), MeshRuntime
  service/     MeshService (foreground), NotificationHelper
  ui/          Compose: temas, navegación, pantallas, ViewModels
app/src/test/  Pruebas del motor con red simulada (SimNetwork)
docs/          PROTOCOL.md, TESTING.md
Pruebas automatizadas (JVM, sin hardware)
./gradlew testDebugUnitTest ejecuta 40 pruebas: serialización y validación de paquetes (incl. entrada basura), fragmentación y reconstrucción con distintos MTU (desordenadas, duplicadas, caducadas), criptografía (cifrado, manipulación, terceros, firmas), enrutamiento, duplicados, límite de saltos, caducidad, store-and-forward, persistencia de cola tras reiniciar el motor, ACK de entrega/lectura, ACK falsificado, repetidor malicioso, repetición tardía, bloqueo, límite de tasa, topologías A–B–C, A–B–C–D y con ciclo.


Decisiones de diseño
Protocolo propio versionado (docs/PROTOCOL.md), no el estándar Bluetooth Mesh (Android no lo expone entre teléfonos).
Anti-inundación: unicast si el destino es vecino; ruta aprendida (camino inverso) si existe; si no, difusión limitada (fanout 2) con exclusión del vecino de origen y de los ya servidos, más TTL, caché de duplicados, límite de cola y limitador de tasa por origen. Reintentos en «olas» espaciadas.
Cripto (Bouncy Castle bcprov-jdk18on 1.78.1): Ed25519, X25519, HKDF-SHA256, ChaCha20-Poly1305. Sin algoritmos inventados.
Sin DI framework: contenedor manual en BlueChatApplication (suficiente aquí).
