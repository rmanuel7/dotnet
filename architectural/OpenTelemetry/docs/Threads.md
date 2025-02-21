# Conceptos
<br/>
<br/>
<br/>
<br/>

# `ExecutionContext.Capture()`

`ExecutionContext.Capture()` en .NET se usa para capturar el contexto de ejecución actual, incluyendo información sobre seguridad, cultura y flujo de datos asíncronos, para que pueda propagarse a otro hilo o contexto de ejecución.

### Uso y Ejemplo

Cuando trabajas con operaciones asincrónicas o múltiples hilos, el contexto de ejecución ayuda a mantener la coherencia del estado lógico de la aplicación. 

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;

class Program
{
    static void Main()
    {
        // Captura el contexto de ejecución actual
        ExecutionContext? capturedContext = ExecutionContext.Capture();

        Thread thread = new Thread(() =>
        {
            if (capturedContext != null)
            {
                // Restaura el contexto de ejecución en el nuevo hilo
                ExecutionContext.Run(capturedContext, _ =>
                {
                    Console.WriteLine("Ejecutando con contexto capturado");
                }, null);
            }
            else
            {
                Console.WriteLine("No se capturó ningún contexto.");
            }
        });

        thread.Start();
        thread.Join();
    }
}
```

### Casos de Uso

1. **Mantenimiento del contexto lógico:** útil en escenarios donde el código debe ejecutarse en diferentes hilos pero debe conservar el contexto original (ejemplo: seguridad, `AsyncLocal<T>`).
2. **Depuración avanzada:** permite inspeccionar el contexto de ejecución en un punto específico.
3. **Ejecución en hilos manuales:** cuando se crean hilos con `Thread` en lugar de `Task`, es posible que el contexto no fluya automáticamente.

### Consideraciones

- **No siempre es necesario capturarlo manualmente.** `Task.Run` y `await` generalmente manejan el contexto automáticamente.
- **Puede impactar el rendimiento.** Capturar y restaurar el contexto en operaciones de alto rendimiento puede introducir sobrecarga innecesaria.
- **No captura el `SynchronizationContext`.** Si necesitas conservar este contexto, considera `SynchronizationContext.Current`.


---
<br/>
<br/>
<br/>
<br/>

# `lock`

En .NET (y en la programación concurrente en general), un **lock** es un mecanismo de sincronización que evita que múltiples hilos accedan simultáneamente a un recurso compartido, evitando así condiciones de carrera y garantizando la consistencia de los datos.

### 🔹 ¿Qué es un recurso en este contexto?
Un **recurso** es cualquier entidad compartida que puede ser accedida por múltiples hilos, como:
- Un objeto en memoria (ej., una lista, un diccionario).
- Un archivo en el sistema.
- Una conexión a la base de datos.
- Una variable compartida.

### 🔹 ¿Cómo funciona `lock` en C#?
En C#, el **bloque `lock`** se usa para restringir el acceso a un recurso a un solo hilo a la vez. Ejemplo:

```csharp
private static readonly object _lock = new object();
private static int contador = 0;

public void Incrementar()
{
    lock (_lock) // Solo un hilo puede ejecutar esto a la vez
    {
        contador++;
        Console.WriteLine($"Contador: {contador}");
    }
}
```

### 🔹 Explicación:
1. `_lock` es un objeto usado como **candado** para la sincronización.
2. **Cuando un hilo entra en `lock (_lock)`**, bloquea ese objeto para que ningún otro hilo pueda acceder a ese código hasta que el bloqueo se libere.
3. Una vez que el hilo termina la ejecución dentro del `lock`, otro hilo puede entrar.

### 🔹 Problemas que resuelve:
✅ Evita **condiciones de carrera** (cuando dos hilos intentan modificar el mismo recurso al mismo tiempo).  
✅ Garantiza **consistencia** en los datos compartidos.  
✅ Facilita la programación concurrente sin conflictos.

### ⚠️ Consideraciones:
- **Nunca uses `lock` en `this` o en un tipo de datos público** porque otro código podría bloquearlo externamente, causando **deadlocks**.
- Si el código dentro de `lock` es muy largo o realiza operaciones bloqueantes (como llamadas a la base de datos), podría causar problemas de rendimiento.
- Para escenarios más avanzados, se pueden usar **`Mutex`**, **`SemaphoreSlim`** o **`Monitor`**, que ofrecen mayor control.

---

<br/>
<br/>
<br/>
<br/>

# `ReaderWriterLockSlim`

`ReaderWriterLockSlim` es una clase en .NET que proporciona un mecanismo de sincronización más eficiente que `Monitor` o `lock` cuando se trata de escenarios en los que múltiples hilos necesitan leer datos al mismo tiempo, pero solo uno debe poder escribir. Se usa principalmente para mejorar el rendimiento en estructuras de datos compartidas con muchas lecturas y pocas escrituras.

---

## 🔹 Características principales:
1. **Permite múltiples lecturas concurrentes**, pero:
   - Solo permite una escritura a la vez.
   - Un escritor bloquea a los lectores y a otros escritores.
2. **Tres niveles de bloqueo:**
   - **Modo de lectura (`EnterReadLock`)**: Permite múltiples lectores simultáneos.
   - **Modo de actualización (`EnterUpgradeableReadLock`)**: Permite leer y, si es necesario, actualizar de forma segura.
   - **Modo de escritura (`EnterWriteLock`)**: Solo un hilo puede escribir y bloquea todo acceso hasta que finalice.

---

## 🔹 Ejemplo de uso:
```csharp
using System;
using System.Threading;

class Program
{
    static ReaderWriterLockSlim _lock = new ReaderWriterLockSlim();
    static int _data = 0;

    static void ReadData()
    {
        _lock.EnterReadLock();
        try
        {
            Console.WriteLine($"[Read] Valor actual: {_data}");
        }
        finally
        {
            _lock.ExitReadLock();
        }
    }

    static void WriteData(int value)
    {
        _lock.EnterWriteLock();
        try
        {
            _data = value;
            Console.WriteLine($"[Write] Nuevo valor: {_data}");
        }
        finally
        {
            _lock.ExitWriteLock();
        }
    }

    static void Main()
    {
        var reader1 = new Thread(ReadData);
        var reader2 = new Thread(ReadData);
        var writer = new Thread(() => WriteData(42));

        reader1.Start();
        reader2.Start();
        writer.Start();

        reader1.Join();
        reader2.Join();
        writer.Join();
    }
}
```
---

## 🔹 Modos de bloqueo en detalle:

| Método                        | Descripción |
|--------------------------------|-------------|
| `EnterReadLock()`              | Bloquea en modo lectura (permite múltiples lectores simultáneos). |
| `EnterWriteLock()`             | Bloquea en modo escritura (exclusivo, bloquea todo). |
| `EnterUpgradeableReadLock()`   | Permite leer y luego actualizar con `EnterWriteLock()`. |
| `ExitReadLock()`               | Libera el bloqueo de lectura. |
| `ExitWriteLock()`              | Libera el bloqueo de escritura. |
| `ExitUpgradeableReadLock()`    | Libera el bloqueo de actualización. |

---

## 🔹 Cuándo usarlo:
✅ Cuando hay muchas lecturas y pocas escrituras.  
✅ Para mejorar el rendimiento frente a `lock`, evitando bloqueos innecesarios.  
✅ Cuando se necesita leer y luego decidir si escribir sin bloquear a otros lectores (`UpgradeableReadLock`).  

🚫 **No es recomendable** cuando las operaciones son rápidas o el acceso a los datos es poco frecuente; en estos casos, `lock` es más simple y suficiente.  

---

### 🔥 Alternativa moderna:
Desde .NET Core 3.0+, **`ConcurrentDictionary<TKey, TValue>`** y otros tipos de colecciones concurrentes eliminan la necesidad de `ReaderWriterLockSlim` en muchos casos.

