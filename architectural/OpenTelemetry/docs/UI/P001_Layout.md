## `ViewportInformation`

Ahora que tenemos la implementación de JavaScript, podemos ver con total claridad cómo se obtiene y maneja la información del viewport en tu aplicación Blazor.

`App.razor`
```razor
<!DOCTYPE html>
<html lang="en">

<head>
	<meta charset="utf-8" />
	<meta name="viewport" content="width=device-width, initial-scale=1.0" />
	<base href="/" />
	<link rel="stylesheet" href="@Assets["lib/bootstrap/dist/css/bootstrap.min.css"]" />
	<link rel="stylesheet" href="@Assets["app.css"]" />
	<link rel="stylesheet" href="@Assets["Aspire.Dashboard.styles.css"]" />
	<ImportMap />
	<link rel="icon" type="image/png" href="favicon.png" />
	<HeadOutlet />
</head>

<body>
	<Routes @rendermode="@(new InteractiveServerRenderMode(prerender: false))" />

	<script src="_framework/blazor.web.js"></script>
	<script src="js/browserDimensionWatcher.js"></script>
</body>

</html>
```

<br/>

```javascript
/**
 * Obtiene las dimensiones actuales de la ventana del navegador.
 * 
 * Esta función es invocada desde el componente `BrowserDimensionWatcher` 
 * en el método `OnAfterRenderAsync` para obtener el ancho y alto de la ventana.
 * 
 * @returns {Object} Un objeto con las dimensiones de la ventana:
 *  - `width` {number}: Ancho de la ventana en píxeles.
 *  - `height` {number}: Alto de la ventana en píxeles.
 */
window.getWindowDimensions = function () {
    return {
        width: window.innerWidth,
        height: window.innerHeight
    };
};

/**
 * Configura un EventListener para cuando ocurra un redimensionamiento de la ventana.
 * @param {Object} dotnetHelper - Instancia del objeto .NET que maneja el evento.
 */
window.listenToWindowResize = function (dotnetHelper) {

    const throttledResizeListener = throttle(() => handleResize(dotnetHelper), 150);

    window.addEventListener('load', throttledResizeListener);
    window.addEventListener('resize', throttledResizeListener);
};

/**
 * Aplica una función de limitación (throttling) a la función proporcionada.
 * @param {Function} func - La función a ejecutar con throttling.
 * @param {number} timeout - El tiempo de espera en milisegundos antes de permitir otra ejecución.
 * @returns {Function} - Una nueva función con la lógica de throttling aplicada.
 */
function throttle(func, timeout) {

    let currentTimeout = null;

    return function () {

        if (currentTimeout) {
            return;
        }

        const context = this;

        // the values of the arguments passed to that function.
        const args = arguments;

        const later = () => {
            func.call(context, ...args);
            currentTimeout = null;
        };

        currentTimeout = setTimeout(later, timeout);
    };
}

/**
 * Manejador de eventos para el redimensionamiento de la ventana.
 * Llama a un método .NET de manera asincrónica con las 'nuevas dimensiones'.
 * @param {Object} dotnetHelper - Instancia del objeto .NET que maneja el evento.
 */
function handleResize(dotnetHelper) {

    dotnetHelper.invokeMethodAsync('OnResizeAsync', {
        width: window.innerWidth,
        height: window.innerHeight
    });
}
```

---

### **Flujo de Datos del Viewport en la Aplicación**  

`Routes.razor`

```razor
@using Aspire.Dashboard.Components.Resize.Models

<BrowserDimensionWatcher @bind-ViewportInformation="@_viewportInformation" />

<CascadingValue Value="@_viewportInformation">
    <Router AppAssembly="typeof(Program).Assembly">
        <Found Context="routeData">
            <RouteView RouteData="routeData" DefaultLayout="typeof(Layout.MainLayout)" />
            <FocusOnNavigate RouteData="routeData" Selector="h1" />
        </Found>
    </Router>
</CascadingValue>

@code {
    private ViewportInformation? _viewportInformation;
}
```

1. **El navegador obtiene las dimensiones del viewport**  
   - `window.getWindowDimensions()` devuelve un objeto con `width` y `height` (`window.innerWidth` y `window.innerHeight`).  
   - Se usa en `BrowserDimensionWatcher` con:  
     ```csharp
     var viewportSize = await JS.InvokeAsync<ViewportSize>("window.getWindowDimensions");
     ```

2. **Se establece un listener para cambios en el tamaño de la ventana**  
   - `window.listenToWindowResize(dotnetHelper)` escucha los eventos `load` y `resize`.  
   - Usa un **throttle de 150ms** para evitar invocaciones excesivas.  

3. **Cuando cambia el tamaño de la ventana, Blazor recibe una actualización**  
   - `dotnetHelper.invokeMethodAsync('OnResizeAsync', { width: window.innerWidth, height: window.innerHeight })`  
   - Llama al método `OnResizeAsync` en `BrowserDimensionWatcher` con el nuevo tamaño del viewport.

4. **Blazor procesa la nueva información**  
   - `OnResizeAsync(ViewportSize viewportSize)` recibe las nuevas dimensiones.  
   - Se genera un nuevo `ViewportInformation` basado en `viewportSize`.  
   - Se actualiza `DimensionManager`, y se notifican los cambios a los componentes que dependen de `ViewportInformation`.

---

### **Resumen**
- **El navegador** proporciona la información del viewport con `window.innerWidth` y `window.innerHeight`.  
- **JavaScript** la envía a Blazor usando `dotnetHelper.invokeMethodAsync`.  
- **Blazor la convierte en `ViewportInformation`** y la propaga a la aplicación a través de `DimensionManager`.  

Este mecanismo permite que la aplicación se adapte dinámicamente a cambios en el tamaño del viewport. 🚀

¿Necesitas ayuda con algún ajuste o tienes dudas sobre cómo optimizar esto?

---



```razor
@inherits LayoutComponentBase

<div class="page">
    <div class="sidebar">
        <NavMenu />
    </div>

    <main>
        <div class="top-row px-4">
            <a href="https://learn.microsoft.com/aspnet/core/" target="_blank">About</a>
        </div>

        <article class="content px-4">
            @Body
        </article>
    </main>
</div>

<div id="blazor-error-ui" data-nosnippet>
    An unhandled error has occurred.
    <a href="." class="reload">Reload</a>
    <span class="dismiss">🗙</span>
</div>
```
