# BrowserTimeZone

Para hacer que la zona horaria esté disponible en toda la aplicación, creamos un servicio **Scoped** y lo inyectamos en el contenedor de dependencias. 

> [!IMPORTANT]
> Esto es diferente de `ViewportInformation`, que es un componente usado para manejar información del viewport y generalmente solo es útil en los componentes de UI.

---

## **Cómo Obtener la Zona Horaria en `MainLayout.razor`**
### **1. Agregar la Función en `wwwroot/js/browserTimeZone.js`**  
Si aún no la tienes, asegúrate de definir la función en un archivo JavaScript accesible por Blazor:  

```javascript
/**
 * Obtiene la zona horaria del navegador.
 *
 * Utiliza `Intl.DateTimeFormat().resolvedOptions()` para determinar
 * la zona horaria configurada en el navegador del usuario.
 *
 * @returns {string} La zona horaria del navegador en formato IANA (ej. "America/New_York").
 */
window.getBrowserTimeZone = function () {
    const options = Intl.DateTimeFormat().resolvedOptions();

    return options.timeZone;
};
```

---

### **2. Llamar la Función desde `MainLayout.razor`**  
Modifica `MainLayout.razor` para usar **`IJSRuntime`** y la estableceremos en `BrowserTimeProvider`.  

```csharp
public partial class MainLayout
    [Inject]
    public required BrowserTimeProvider TimeProvider { get; init; }

    protected override async Task OnInitializedAsync()
    {
        var result = await JS.InvokeAsync<string>("window.getBrowserTimeZone");
        TimeProvider.SetBrowserTimeZone(result);
    }
}
```

---

### **3. Crear un Servicio para la Zona Horaria**
Crea una clase `BrowserTimeProvider.cs` en Services o en una carpeta apropiada.

```csharp
/// <summary>
/// Proveedor de tiempo basado en la zona horaria del navegador, utilizado en Blazor Server con Aspire.
/// </summary>
/// <remarks>
/// Esta clase extiende <see cref="TimeProvider"/> para permitir el uso de la zona horaria del navegador 
/// en lugar de la zona horaria del sistema.
/// 
/// - Se debe registrar como <c>Scoped</c> en el contenedor de dependencias.
/// - Se usa en Blazor Server para sincronizar la zona horaria del cliente con el servidor.
/// - Si la zona horaria del navegador no está establecida, se usa la del sistema.
/// </remarks>
public class BrowserTimeProvider : TimeProvider
{
    private readonly ILogger _logger;
    private TimeZoneInfo? _browserLocalTimeZone;

    /// <summary>
    /// Inicializa una nueva instancia de la clase <see cref="BrowserTimeProvider"/>.
    /// </summary>
    /// <param name="loggerFactory">Fábrica de <see cref="ILoggerFactory"/> utilizada para registrar eventos.</param>
    public BrowserTimeProvider(ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger(typeof(BrowserTimeProvider));
    }

    /// <summary>
    /// Obtiene la zona horaria local del navegador si está disponible; de lo contrario, devuelve la zona horaria del sistema.
    /// </summary>
    public override TimeZoneInfo LocalTimeZone
    {
        get => _browserLocalTimeZone ?? base.LocalTimeZone;
    }

    /// <summary>
    /// Establece la zona horaria basada en la información obtenida del navegador.
    /// </summary>
    /// <param name="timeZone">Identificador de zona horaria en formato IANA (por ejemplo, "America/New_York").</param>
    /// <remarks>
    /// Si la zona horaria proporcionada no es válida, se registrará un mensaje de advertencia y se usará UTC por defecto.
    /// </remarks>
    public void SetBrowserTimeZone(string timeZone)
    {
        if (!TimeZoneInfo.TryFindSystemTimeZoneById(timeZone, out var timeZoneInfo))
        {
            _logger.LogWarning("Couldn't find time zone '{TimeZone}'. Defaulting to UTC.", timeZone);
            timeZoneInfo = TimeZoneInfo.Utc;
        }

        _logger.LogDebug("Browser time zone set to '{TimeZone}' with UTC offset {UtcOffset}.", timeZoneInfo.Id, timeZoneInfo.BaseUtcOffset);
        _browserLocalTimeZone = timeZoneInfo;
    }
}
```
---
---

# Doc

Obtener la zona horaria del navegador (`BrowserTimeZone`) en una aplicación Blazor Server tiene varios propósitos, incluyendo el **soporte multilenguaje**, pero también otros usos importantes. Aquí te explico algunos de ellos:  

---

## 📌 **1. Soporte Multilenguaje (i18n)**
Algunas aplicaciones adaptan la localización (idioma y formato de fecha/hora) según la zona horaria del usuario.  
- Ejemplo: Un usuario en **España (Europe/Madrid)** debería ver las fechas en formato `dd/MM/yyyy`, mientras que un usuario en **EE.UU. (America/New_York)** debería verlas como `MM/dd/yyyy`.  
- Puedes usar la zona horaria junto con **culturas (`CultureInfo`)** en .NET para ajustar automáticamente el formato de fecha/hora.  

```csharp
var culture = new CultureInfo("es-ES");
var fecha = DateTime.UtcNow;
var fechaFormateada = fecha.ToString("f", culture); // "martes, 24 de febrero de 2025 10:30"
```

---

## 📌 **2. Convertir Fechas Correctamente (UTC a Local)**
Blazor Server ejecuta su código en el servidor, donde el `DateTime.Now` usa la zona horaria del servidor (que puede ser UTC o cualquier otra).  
Si envías fechas al cliente sin ajuste, pueden mostrarse incorrectamente.  

### ✅ Solución:
- **Enviar fechas en UTC** desde el servidor.  
- **Convertirlas a la hora local** en el cliente usando `Intl.DateTimeFormat()` o la zona horaria obtenida.  

```javascript
const fechaUtc = "2025-02-24T15:00:00Z";
const fechaLocal = new Date(fechaUtc).toLocaleString("es-ES", { timeZone: getBrowserTimeZone() });
console.log(fechaLocal); // "24/2/2025, 16:00" (si la zona horaria es CET)
```

En Blazor, puedes almacenar la zona horaria del usuario y hacer conversiones en C#:

```csharp
var userTimeZone = "America/New_York"; // Obtenido desde JS
var dateUtc = DateTime.UtcNow;
var dateInUserTimeZone = TimeZoneInfo.ConvertTimeBySystemTimeZoneId(dateUtc, userTimeZone);
```

---

## 📌 **3. Agendar Eventos en la Hora Correcta**
Si tienes una aplicación que maneja citas o reuniones (por ejemplo, una tienda con reservas en línea o un sistema de eventos), necesitas convertir la hora correctamente según la ubicación del usuario.  

Ejemplo:  
- Un usuario en **Londres** agenda una reunión a las **15:00 UTC**.  
- Un usuario en **Nueva York** debe ver esa reunión como **10:00 AM (EST, UTC-5)**.

---

## 📌 **4. Mostrar Tiempos Relativos Correctamente**
Si una aplicación muestra tiempos relativos como "hace 2 horas", necesitas la zona horaria del usuario para calcular correctamente la diferencia de tiempo.  

---
