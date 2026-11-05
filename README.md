# WineMate — app Android de monitoreo IoT para bodegas

Aplicación Android nativa (Kotlin) del proyecto **WineMate**: monitoreo de temperatura y humedad
de lotes de vino con nodos ESP32, alertas por umbrales y trazabilidad por lote.

## Estado actual

| Módulo | Estado |
|---|---|
| Login / registro / recuperación de contraseña | ✅ funcionando (local cifrado; Firebase opcional) |
| Sesión persistente + cierre de sesión | ✅ |
| Panel de inicio con contadores y tarjetas por lote | ✅ |
| Detalle de lote en vivo | ✅ |
| Alertas con severidad, historial y **filtro por prioridad** | ✅ |
| **Datos personales editables** (agregar / editar / borrar) | ✅ |
| **Lotes editables y eliminables** | ✅ |
| Agregar dispositivo (BLE real + alta manual) | ✅ código listo (requiere nodo ESP32 para probar) |
| Conexión IoT (simulación / MQTT / Firebase) | ✅ configuración persistida cifrada |
| Barra inferior Inicio / Alertas / Perfil | ✅ corregida (insets + sin restaurar subpantallas) |
| Icono de la app con el logo de WineMate | ✅ adaptativo (API 26+) + legacy |
| Modo oscuro del sistema | ✅ la app mantiene siempre su tema claro de marca |
| Pruebas unitarias | ✅ 18 pruebas, 0 fallos |
| Hardware ESP32 + sensores | ⏳ pendiente de construcción |

## Cómo compilar

```powershell
$env:JAVA_HOME = "D:\Android studio\jbr"
.\gradlew.bat :app:assembleDebug        # APK en app\build\outputs\apk\debug\
.\gradlew.bat :app:testDebugUnitTest    # pruebas unitarias
adb install -r app\build\outputs\apk\debug\app-debug.apk
```

## Cómo usar la app

1. Abre la app: si no hay sesión, muestra el login.
2. Puedes **registrarte** con cualquier correo y contraseña (mínimo 8 caracteres, con letras y números).
3. En builds de depuración aparece el atajo **«Entrar en modo demostración»**, que crea la cuenta
   `demo@winemate.cl` (contraseña `WineMate2025`) y entra directo al panel con 3 lotes de ejemplo.
4. La telemetría se actualiza cada 4 segundos y cada lote sigue un ciclo térmico propio: el
   estanque cruza los 15 °C y 18 °C en pocos segundos, así que **en menos de 15 s aparece una alerta
   nueva** (y más tarde la de recuperación).

### Datos personales (Perfil → Datos personales)

Formulario con **nombre, teléfono, bodega y región**. Puedes agregar, editar o borrar la información:

- **Guardar cambios** persiste todo cifrado y actualiza el saludo del inicio y la tarjeta del perfil.
- **Borrar datos** limpia teléfono, bodega y región (el nombre y el correo de la cuenta se mantienen).
- El correo es la identidad de la cuenta: se muestra, pero no se edita desde aquí.

### Lotes (Detalle de un lote)

- **Editar**: reutiliza el formulario de alta con los datos cargados (lote, nodo, contenedor,
  ubicación y cepa). Las mediciones del nodo no se tocan.
- **Eliminar**: pide confirmación y deja una alerta informativa en el historial.

### Alertas

Chips de filtro por prioridad: **Todas · Con atención · Críticas · OK**. «Con atención» agrupa
críticas y advertencias; «OK» agrupa las informativas (nodo estabilizado, nodo conectado).

## Estructura

```
app/src/main/java/com/example/winemate/
├── core/          resultado, validadores, tiempos, ServiceLocator, fábrica de ViewModels
├── security/      CryptoManager (AES-256-GCM + Keystore), SecureStorage, PasswordHasher (PBKDF2)
├── data/
│   ├── model/     WineDevice, WineThresholds, AlertItem, UserSession, UserProfile, IotConfig
│   ├── auth/      AuthRepository + local cifrado + Firebase (opcional)
│   ├── iot/       BleProvisioner (GATT), NetworkMonitor
│   └── repository/WineMateRepository, ProfileRepository, IotConfigRepository
└── ui/            auth, home, alerts, profile, device, settings, adapter
```

## Recursos gráficos

- `tools/make_icons.py` genera los iconos de launcher (legacy + adaptativo + round) y el logo
  circular in-app a partir de `WineMate_logo_500x500.png`.

## Activar Firebase (opcional)

La app funciona sin configuración cloud. Para usar Firebase Authentication:

- Opción A: descarga `google-services.json` y guárdalo en `app/`.
- Opción B: define en `gradle.properties`:
  `WINEMATE_FIREBASE_PROJECT_ID`, `WINEMATE_FIREBASE_APP_ID`, `WINEMATE_FIREBASE_API_KEY`,
  `WINEMATE_FIREBASE_SENDER_ID`.

Si no hay configuración, la app usa autenticación local cifrada y no falla.

## Documentación

- `docs/WineMate_Informe_Tecnico.docx` — informe con los criterios de la rúbrica, evidencias,
  pruebas y decisiones tecnológicas.
- `docs/WineMate_Informe_Tecnico.docx` §5 — mockups y capturas reales de la app.

## Privacidad y seguridad

- Contraseñas: PBKDF2-SHA256, 120.000 iteraciones, salt de 16 bytes (nunca en texto plano).
- Datos y sesión: cifrados con AES-256-GCM usando una llave no exportable del Android Keystore.
- Bloqueo temporal tras 5 intentos fallidos; respaldos deshabilitados (`allowBackup=false`).
