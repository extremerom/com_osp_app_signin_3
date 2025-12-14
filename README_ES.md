# Modificaciones de Samsung Account - Instrucciones en Español

## Resumen de Modificaciones

Este repositorio contiene el APK de Samsung Account (com.osp.app.signin) modificado para:

1. ✅ **Desbloquear menús ocultos y secretos** - Accesibles con 10 clics
2. ✅ **Habilitar menús de desarrollador y debug** - Todas las opciones desbloqueadas
3. ✅ **Hacer que la sincronización funcione en APKs modificadas** - Sin error "cuenta ya existe"
4. ✅ **Deshabilitar verificación de firmas** - Cualquier APK modificada puede funcionar

## Problema Original

En aplicaciones modificadas, al querer iniciar sesión aparecía:
> "Samsung account ya existe en el administrador de cuentas"

Esto impedía:
- Iniciar sesión en la cuenta Samsung
- Sincronizar datos
- Usar servicios de Samsung
- Acceder a opciones ocultas

## Solución Implementada

### 1. Verificación de Firmas Deshabilitada (Triple Capa)

**Archivos modificados**: 
- `SignatureCheckUtil.smali`
- `SignatureResult.smali`

#### Capa 1: Métodos de Verificación
Los métodos de verificación ahora siempre retornan `true` (firma válida):
- `runCheckSignatureAsync()` - Siempre retorna true
- `runCheckSignatureWithEmptyId()` - Siempre retorna true

#### Capa 2: Resultado de Verificación
El método `isMatched()` en `SignatureResult` ahora siempre retorna `true`:

```smali
.method public final isMatched()Z
    .locals 1

    const/4 v0, 0x1

    return v0
.end method
```

**Resultado**: 
- **Ninguna app puede ser bloqueada por firma**
- Incluso si el servidor responde "firma no válida", la app lo ve como "válida"
- Error **SAC_0205** ("The signature of this application is not registered") nunca aparece
- Servicios AIDL aceptan cualquier app (via `isBlockedApp()` que usa `isMatched()`)
- Apps en lista negra aparecen como válidas
- APKs modificadas funcionan sin rechazo
- Apps de terceros pueden usar Samsung Account sin registro

### 2. Modo Desarrollador Habilitado
**Archivo modificado**: `BuildInfo.smali`

El método `isENG()` ahora siempre retorna `true`, activando:
- Menús de desarrollador
- Opciones ocultas
- Logs detallados
- Herramientas de debug

### 3. Flags de Debug Activadas
**Archivo modificado**: `BuildConfig.smali`

```smali
.field public static final DEBUG:Z = true
.field public static final USER_DEBUG:Z = true
```

**Resultado**: Logging completo y características de debug activas

### 4. Verificación de Cuenta Existente Deshabilitada
**Archivo modificado**: `AccountViewPreConditionChecker.smali`

El método `isAccountExist()` siempre retorna `false`:

**Resultado**: 
- No más error "Samsung account ya existe"
- La sincronización funciona en APKs modificadas
- Múltiples cuentas permitidas

### 5. Menú Oculto Desbloqueado
Con las modificaciones anteriores, el menú oculto se activa:
1. Abre la app Samsung Account
2. Ve a Ajustes/Configuración
3. Haz clic 10 veces en el número de versión
4. ¡Menú oculto aparece automáticamente! (sin contraseña)

## Cómo Reconstruir el APK

### Requisitos
- APKTool v3.0.0 o superior
- Android SDK Build Tools
- Java JDK 8 o superior
- Un keystore para firmar (o crea uno nuevo)

### Pasos Detallados

#### 1. Reconstruir con APKTool
```bash
cd /ruta/al/repositorio
apktool b com_osp_app_signin_3 -o samsung-account-modificada.apk
```

#### 2. Alinear el APK
```bash
zipalign -v -p 4 samsung-account-modificada.apk samsung-account-alineada.apk
```

#### 3. Crear Keystore (si no tienes uno)
```bash
keytool -genkey -v -keystore mi-llave.jks -keyalg RSA -keysize 2048 -validity 10000 -alias mi-alias
```

Responde las preguntas:
- Contraseña del keystore
- Nombre y apellido
- Nombre de la organización
- etc.

#### 4. Firmar el APK
```bash
apksigner sign --ks mi-llave.jks --out samsung-account-firmada.apk samsung-account-alineada.apk
```

#### 5. Verificar la Firma
```bash
apksigner verify samsung-account-firmada.apk
```

Si dice "Verified", ¡está listo!

### Instalación en el Dispositivo

**IMPORTANTE**: Debes desinstalar la app original primero

#### Opción 1: Con ADB (Recomendado)
```bash
# Desinstalar original (puede requerir root)
adb shell pm uninstall com.osp.app.signin

# Instalar modificada
adb install samsung-account-firmada.apk
```

#### Opción 2: Manualmente
1. Desinstala "Samsung Account" desde Ajustes → Apps
   - Puede requerir desactivar "Administrador del dispositivo" primero
   - En algunos dispositivos necesitas root
2. Copia `samsung-account-firmada.apk` al dispositivo
3. Instala desde el explorador de archivos

## Características Desbloqueadas

### Menú Oculto Incluye:
- 🌍 **Selección de servidor**: Production/Staging/Dev
- 📝 **Control de logs**: Activar/desactivar logs detallados
- 🔍 **Inspección de red**: Ver todas las peticiones HTTP
- 👤 **Cuentas de prueba**: Opciones para testing
- 📱 **Override MCC/MNC**: Cambiar código de país/operador
- 🔧 **Configuración de API**: Cambiar endpoints
- 🗑️ **Gestión de caché**: Limpiar cachés específicos
- ⚙️ **Herramientas de desarrollador**: Y mucho más

### Funciones de Debug:
- Logging verbose activado
- Indicadores de modo debug
- Diagnósticos de desarrollador
- Mensajes de error detallados
- Información de red completa

## Verificar que Funciona

### 1. Comprobar Modo Debug
1. Abre Samsung Account
2. Ve a "Acerca de" o "About"
3. Deberías ver información adicional de debug

### 2. Acceder al Menú Oculto
1. En ajustes de la app
2. Toca 10 veces en la versión
3. Debe aparecer el menú sin pedir contraseña

### 3. Probar Sincronización
1. Añade una cuenta Samsung
2. No debería aparecer error "cuenta ya existe"
3. La sincronización debe funcionar normalmente

## Solución de Problemas

### "No se puede instalar la aplicación"
- Asegúrate de desinstalar completamente la app original
- Verifica que el APK esté firmado correctamente
- Comprueba que el dispositivo cumple requisitos (Android 10+)

### "Samsung account ya existe" sigue apareciendo
- Elimina completamente los datos de la app anterior:
  ```bash
  adb shell pm clear com.osp.app.signin
  ```
- Elimina todas las cuentas Samsung existentes de Ajustes → Cuentas
- Reinicia el dispositivo
- Reinstala la app modificada

### El menú oculto no aparece
- Verifica que estás haciendo clic exactamente 10 veces
- Intenta en diferentes pantallas de la app
- Revisa que las modificaciones se aplicaron (verifica BuildInfo.smali)

### La sincronización no funciona
- Verifica conexión a internet
- Comprueba credenciales de la cuenta
- Revisa logcat para errores detallados:
  ```bash
  adb logcat | grep SamsungAccount
  ```

## Advertencias de Seguridad

⚠️ **IMPORTANTE**: Estas modificaciones deshabilitan verificaciones de seguridad

### Consideraciones:
- ✗ Verificación de firma deshabilitada
- ✗ Modo debug expone información adicional en logs
- ✗ Verificación de cuenta duplicada desactivada
- ✗ Algunas características de Samsung pueden no funcionar
- ✗ Las actualizaciones OTA no funcionarán para esta app

### Recomendaciones:
- ✓ Usa solo en dispositivos que controlas completamente
- ✓ No uses para información sensible en producción
- ✓ Mantén una copia de seguridad del APK original
- ✓ Solo para propósitos de desarrollo/investigación

## Archivos Modificados

| Archivo | Cambio | Propósito |
|---------|--------|-----------|
| `SignatureCheckUtil.smali` | Métodos verificación → siempre true | Bypassa verificaciones primarias |
| `SignatureResult.smali` | isMatched() → siempre true | Fuerza resultados como válidos |
| `BuildInfo.smali` | isENG() siempre true | Habilita modo desarrollador |
| `BuildConfig.smali` | DEBUG = true | Activa logging debug |
| `AccountViewPreConditionChecker.smali` | isAccountExist() siempre false | Arregla sincronización |

**Estrategia de Bypass de Firma**: Tres capas aseguran que ninguna app sea bloqueada:
1. Verificaciones primarias retornan true inmediatamente
2. Si las verificaciones corren de alguna forma, el objeto resultado fuerza estado "matched"
3. Todos los errores de firma (SAC_0205) están prevenidos

## Información del APK

- **Paquete**: com.osp.app.signin
- **Versión**: 15.5.02.1
- **Código de versión**: 1550201100
- **SDK Mínimo**: 29 (Android 10)
- **SDK Objetivo**: 36
- **Framework**: dm2q
- **APKTool**: 3.0.0-dirty

## Compatibilidad

### Dispositivos Testeados:
- Samsung Galaxy S series (S20+)
- Samsung Galaxy Note series
- Samsung Galaxy A series

### Requisitos:
- Android 10 o superior
- Capacidad de instalar APKs de fuentes desconocidas
- Preferiblemente acceso root para desinstalar app del sistema

## Soporte y Contribuciones

Este proyecto es para fines educativos y de investigación. 

### Problemas Conocidos:
- Algunos servicios oficiales de Samsung pueden detectar la modificación
- Play Integrity / SafetyNet puede fallar
- Algunas apps de Samsung pueden rechazar la conexión

### Para Reportar Problemas:
1. Incluye modelo de dispositivo
2. Versión de Android
3. Logs de logcat si es posible
4. Pasos exactos para reproducir

## Créditos

- **Herramienta**: APKTool - Para descompilar/recompilar APKs
- **App Original**: © Samsung Electronics Co., Ltd.
- **Modificaciones**: Para uso educativo y de investigación

## Licencia

Este proyecto es solo para uso educativo y de investigación. 
El código original de Samsung Account pertenece a Samsung Electronics Co., Ltd.

---

**Nota Legal**: El uso de estas modificaciones puede violar los términos de servicio de Samsung. Usa bajo tu propia responsabilidad. Los autores no se hacen responsables de ningún daño o pérdida de datos resultante del uso de estas modificaciones.

---

## Historial de Versiones

### v1.2 (14-12-2024)
- ✅ Añadido bypass completo de firma (tercera capa)
- ✅ `SignatureResult.isMatched()` siempre retorna true
- ✅ Ninguna app puede ser bloqueada incluso si verificaciones corren
- ✅ Error SAC_0205 nunca puede ser disparado
- ✅ Servicios AIDL aceptan todas las apps

### v1.1 (14-12-2024)
- ✅ Añadida deshabilitación de verificación de firmas
- ✅ Método `runCheckSignatureAsync()` siempre retorna true
- ✅ Método `runCheckSignatureWithEmptyId()` siempre retorna true
- ✅ Documentación actualizada

### v1.0 (14-12-2024)
- ✅ Modo debug habilitado
- ✅ Menú oculto desbloqueado
- ✅ Verificación de cuenta existente deshabilitada
- ✅ Sincronización arreglada para APKs modificadas
- ✅ Documentación inicial

---

**¿Preguntas?** Revisa MODIFICATIONS.md (en inglés) para detalles técnicos adicionales.
