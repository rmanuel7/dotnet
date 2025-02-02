```csharp
/// <summary>
/// Registra y configura el sistema de eventos de dominio en la aplicación.
/// </summary>
/// <param name="services">La colección de servicios de la aplicación.</param>
/// <returns>La colección de servicios con la configuración de eventos de dominio aplicada.</returns>
/// <remarks>
/// - Registra el bus de eventos de dominio como un servicio de ámbito (<see cref="IDomainEventBus"/>).
/// - Asocia eventos de dominio con sus respectivos manejadores.
/// </remarks>
public static IServiceCollection AddDomainEventExtension(this IServiceCollection services)
{
    // Registrar el bus de eventos de dominio con un ciclo de vida Scoped
    services.AddScoped<IDomainEventBus, DefaultDomainEventBus>();

    // Registrar manejadores de eventos de dominio
    services.AddDomainEventHandler<OrderStartedDomainEvent, BuyerOrderStartedDomainEventHandler>();
    services.AddDomainEventHandler<PaymentMethodVerifiedDomainEvent, UpdateOrderPaymentVerifiedDomainEventHandler>();
    services.AddDomainEventHandler<OrderAwaitingDomainEvent, OrderAwaitingDomainEventHandler>();

    return services;
}

```



```csharp
    /// <summary>
    /// Registra un manejador de eventos de dominio en el contenedor de dependencias y configura la información de suscripción para el evento.
    /// </summary>
    /// <typeparam name="TDomainEvent">El tipo específico del evento de dominio que será manejado.</typeparam>
    /// <typeparam name="TDomainEventHandler">La implementación concreta de <see cref="IDomainEventHandler{TDomainEvent}"/> que manejará el evento.</typeparam>
    /// <param name="services">El contenedor de servicios en el que se registrarán las dependencias.</param>
    /// <returns>El mismo <see cref="IServiceCollection"/> recibido como parámetro, para permitir encadenamiento fluido.</returns>
    public static IServiceCollection AddDomainEventHandler<TDomainEvent, TDomainEventHandler>(this IServiceCollection services)
        where TDomainEvent : DomainEvent
        where TDomainEventHandler : class, IDomainEventHandler<TDomainEvent>
    {
        // Obtiene el tipo del evento de dominio.
        Type EventType = typeof(TDomainEvent);

        // Registra el manejador de eventos como un servicio transitorio y lo asocia con la clave específica del tipo del evento.
        // Esto permite la resolución de múltiples manejadores dependiendo del tipo de evento.
        services.AddKeyedTransient<IDomainEventHandler, TDomainEventHandler>(serviceKey: EventType);

        // Configura las opciones de suscripción a eventos para incluir el tipo actual del evento.
        services.Configure<DomainEventSubscriptionInfo>(configureOptions: subInfo =>
        {
            // Almacena el tipo de evento en un diccionario, donde la clave es el nombre del evento
            // y el valor es el tipo del evento.
            subInfo.EventTypes[key: EventType.Name] = EventType;
        });

        // Devuelve el contenedor de servicios actualizado.
        return services;
    }
```



```csharp
public class DomainEventSubscriptionInfo
{
    public Dictionary<string, Type> EventTypes { get; } = [];
}
```




```csharp
public interface IDomainEventBus
{
    Task PublishAsync<TDomainEvent>(TDomainEvent @event) where TDomainEvent : DomainEvent;
}
```


```csharp

public class DefaultDomainEventBus : IDomainEventBus
{
    private readonly DomainEventSubscriptionInfo _subscriptionInfo;
    private readonly IServiceProvider _serviceProvider;

    public DefaultDomainEventBus(IOptions<DomainEventSubscriptionInfo> subscriptionInfo, IServiceProvider serviceProvider)
    {
        _subscriptionInfo = subscriptionInfo.Value;
        _serviceProvider = serviceProvider;
    }

    /// <summary>
    /// Publica un evento de dominio y lo distribuye a los manejadores registrados para ese tipo de evento.
    /// </summary>
    /// <typeparam name="TDomainEvent">El tipo específico del evento de dominio que se va a publicar.</typeparam>
    /// <param name="event">El evento de dominio que se va a publicar.</param>
    /// <returns>Una tarea que representa la operación asincrónica de la publicación del evento.</returns>
    public async Task PublishAsync<TDomainEvent>(TDomainEvent @event) where TDomainEvent : DomainEvent
    {
        // Obtiene el nombre del tipo del evento para su uso como clave en la búsqueda de manejadores.
        String eventName = @event.GetType().Name;
        try
        {
            // Verifica si el evento está registrado en el diccionario de tipos de eventos (EventTypes).
            if (_subscriptionInfo.EventTypes.TryGetValue(key: eventName, value: out var eventType))
            {
                // Obtiene los manejadores registrados para el tipo de evento mediante la clave (eventType).
                var handlers = _serviceProvider.GetKeyedServices<IDomainEventHandler>(serviceKey: eventType);

                // Itera sobre cada manejador y les envía el evento para su procesamiento.
                foreach (var handler in handlers)
                {
                    await handler.Handle(@event);  // Llama al método 'Handle' del manejador de eventos.
                }
            }
        }
        catch (Exception ex)
        {
            System.Diagnostics.Debug.WriteLine(message: ex.Message);
        }
    }
}
```
