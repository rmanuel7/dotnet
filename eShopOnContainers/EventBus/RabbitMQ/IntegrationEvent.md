```csharp
/// <summary>
/// Extensión para inicializar RabbitMQ cuando se ejecuta un <see cref="IWebHost"/>.
/// </summary>
/// <param name="webHost">El host web en el que se está ejecutando la aplicación.</param>
/// <returns>El mismo <see cref="IWebHost"/> recibido como parámetro, tras inicializar RabbitMQ.</returns>
public static IWebHost RunRabbitMQ(this IWebHost webHost)
{
    // Crea un alcance de servicios para obtener instancias específicas del contenedor de dependencias.
    using (var scope = webHost.Services.CreateScope())
    {
        // Obtiene una instancia del IEventBus desde el contenedor.
        var eventBus = scope.ServiceProvider.GetRequiredService<IEventBus>();

        // Inicia el sistema de mensajería de RabbitMQ y espera a que se complete.
        eventBus.StartAsync().Wait();
    }

    // Retorna el IWebHost para permitir una cadena fluida en la configuración del host.
    return webHost;
}
```



```csharp
/// <summary>
/// Agrega y configura la infraestructura para el manejo de eventos de integración con RabbitMQ.
/// </summary>
/// <param name="services">La colección de servicios de la aplicación.</param>
/// <param name="configuration">La configuración de la aplicación, utilizada para obtener los valores de conexión.</param>
/// <returns>La colección de servicios con la configuración de eventos de integración aplicada.</returns>
/// <remarks>
/// - Configura las opciones de RabbitMQ desde la configuración de la aplicación.
/// - Registra la conexión persistente a RabbitMQ como un servicio singleton.
/// - Registra el bus de eventos como un servicio singleton utilizando <see cref="EventBusRabbitMQ"/>.
/// - Registra los servicios de eventos de integración y sus manejadores.
/// </remarks>
public static IServiceCollection AddIntegrationEventExtension(this IServiceCollection services, IConfiguration configuration)
{
    // Configurar opciones de RabbitMQ desde la configuración de la aplicación
    services.Configure<RabbitMQOptions>(config: configuration);

    // Registrar la conexión persistente a RabbitMQ
    services.AddSingleton<IRabbitMQPersisterConnection>(implementationFactory: serviceProvider =>
    {
        var logger = serviceProvider.GetRequiredService<ILogger<DefaultRabbitMQPersisterConnection>>();

        var factory = new ConnectionFactory()
        {
            HostName = configuration["EventBusConnection"]
        };

        return new DefaultRabbitMQPersisterConnection(connectionFactory: factory, logger: logger);
    });

    // Registrar el bus de eventos basado en RabbitMQ
    services.AddSingleton<IEventBus>(implementationFactory: static _serviceProvider =>
    {
        var persisterConnection = _serviceProvider.GetRequiredService<IRabbitMQPersisterConnection>();
        var subscriptionInfo = _serviceProvider.GetRequiredService<IOptions<EventBusSubscriptionInfo>>();
        var rabbitmqOptions = _serviceProvider.GetRequiredService<IOptions<RabbitMQOptions>>();
        var logger = _serviceProvider.GetRequiredService<ILogger<EventBusRabbitMQ>>();

        return new EventBusRabbitMQ(
            persisterConnection: persisterConnection,
            serviceProvider: _serviceProvider,
            subscriptionInfo: subscriptionInfo,
            logger: logger,
            rabbitmqOptions: rabbitmqOptions);
    });

    // Registrar el servicio de eventos de integración
    services.AddTransient<IOrderIntegrationEventService, OrderIntegrationEventService>();

    // Registrar los manejadores de eventos de integración
    services.AddIntegrationEventHandler<UserCheckoutAcceptedIntegrationEvent, UserCheckoutAcceptedIntegrationEventHandler>();
    services.AddIntegrationEventHandler<GracePeriodConfirmedIntegrationEvent, GracePeriodConfirmedIntegrationEventHandler>();

    return services;
}
```



```csharp
/// <summary>
/// Registra un manejador de eventos de integración en el contenedor de dependencias.
/// </summary>
/// <typeparam name="TIntegrationEvent">El tipo del evento de integración que se manejará.</typeparam>
/// <typeparam name="TIntegrationEventHandler">El tipo del manejador del evento de integración, que debe implementar <see cref="IIntegrationEventHandler{TIntegrationEvent}"/>.</typeparam>
/// <param name="services">La colección de servicios de la aplicación.</param>
/// <returns>La colección de servicios con el manejador de eventos registrado.</returns>
/// <remarks>
/// - Registra el manejador de eventos como un servicio transitorio utilizando una clave basada en el tipo del evento.
/// - Configura la suscripción al evento en <see cref="EventBusSubscriptionInfo"/>.
/// </remarks>
public static IServiceCollection AddIntegrationEventHandler<TIntegrationEvent, TIntegrationEventHandler>(this IServiceCollection services)
    where TIntegrationEvent : IntegrationEvent
    where TIntegrationEventHandler : class, IIntegrationEventHandler<TIntegrationEvent>
{
    Type EventType = typeof(TIntegrationEvent);

    services.AddKeyedTransient<IIntegrationEventHandler, TIntegrationEventHandler>(serviceKey: EventType);

    services.Configure<EventBusSubscriptionInfo>(configureOptions: subInfo =>
    {
        subInfo.EventTypes[key: EventType.Name] = EventType;
    });

    return services;
}
```



```csharp
/// <summary>
/// Define una interfaz para un sistema de mensajería basado en eventos, 
/// que permite la suscripción, desuscripción y publicación de eventos de integración.
/// </summary>
public interface IEventBus
{
    /// <summary>
    /// Publica un evento de integración para que sea manejado por los suscriptores correspondientes.
    /// </summary>
    /// <param name="event">El evento de integración que se publicará.</param>
    void Publish(IntegrationEvent @event);

    /// <summary>
    /// Inicia la conexión con el sistema de mensajería y enlaza las colas a los eventos configurados.
    /// </summary>
    Task StartAsync();
}
```



```csharp
/// <summary>
/// Implementación del bus de eventos basada en RabbitMQ. Permite la suscripción, 
/// desuscripción y publicación de eventos a través de un broker de mensajería RabbitMQ.
/// </summary>
public class EventBusRabbitMQ : IEventBus, IDisposable
{
    /// <summary>
    /// Nombre del broker utilizado por RabbitMQ para el intercambio de mensajes.
    /// </summary>
    private const string BROKER_NAME = "eshop_event_bus";

    /// <summary>
    /// Conexión persistente a RabbitMQ.
    /// </summary>
    private readonly IRabbitMQPersisterConnection _persisterConnection;

    /// <summary>
    /// Registro de logs para monitorear el comportamiento del bus de eventos.
    /// </summary>
    private readonly ILogger<EventBusRabbitMQ> _logger;

    private readonly IServiceProvider _serviceProvider;

    private readonly EventBusSubscriptionInfo _subscriptionInfo;

    /// <summary>
    /// Canal de comunicación utilizado para consumir mensajes de RabbitMQ.
    /// </summary>
    private IChannel _consumerChannel;

    /// <summary>
    /// Nombre de la cola de mensajes utilizada para procesar eventos.
    /// </summary>
    private string? _queueName;

    /// <summary>
    /// Inicializa una nueva instancia de la clase <see cref="EventBusRabbitMQ"/>.
    /// </summary>
    /// <param name="persisterConnection">La conexión persistente a RabbitMQ.</param>
    /// <param name="logger">El registro de logs para la instancia.</param>
    /// <exception cref="ArgumentNullException">Se lanza si <paramref name="persisterConnection"/> o <paramref name="logger"/> son nulos.</exception>
    /// <param name="queueName">Nombre de la cola asociada. Si no se especifica, se utiliza una cola predeterminada.</param>
    public EventBusRabbitMQ(IRabbitMQPersisterConnection persisterConnection, IServiceProvider serviceProvider, IOptions<EventBusSubscriptionInfo> subscriptionInfo, ILogger<EventBusRabbitMQ> logger, IOptions<RabbitMQOptions> rabbitmqOptions)
    {
        _persisterConnection = persisterConnection ?? throw new ArgumentNullException(nameof(persisterConnection));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
        _queueName = rabbitmqOptions.Value.EventBusClientName;
        _serviceProvider = serviceProvider;
        _subscriptionInfo = subscriptionInfo.Value;

        // Crea el canal para consumir mensajes.
        _consumerChannel = CreateConsumerChannel();
    }

    /// <summary>
    /// Publica un evento de integración en el bus de eventos RabbitMQ.
    /// </summary>
    /// <param name="event">El evento de integración que se publicará.</param>
    /// <exception cref="BrokerUnreachableException">
    /// Se lanza si no se puede conectar al broker de RabbitMQ después de múltiples intentos.
    /// </exception>
    /// <exception cref="SocketException">
    /// Se lanza si ocurre un problema relacionado con la red durante la publicación del evento.
    /// </exception>
    public void Publish(IntegrationEvent @event)
    {
        // Verifica si la conexión con RabbitMQ está activa; si no, intenta reconectar.
        if (!_persisterConnection.IsConnected)
        {
            _persisterConnection.TryConnect();
        }

        // Define una política de reintento en caso de errores de conexión.
        var policy = RetryPolicy.Handle<BrokerUnreachableException>()
            .Or<SocketException>()
            .WaitAndRetry(
                retryCount: 5, // Número de reintentos.
                sleepDurationProvider: retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)), // Tiempo de espera exponencial.
                onRetry: (ex, time) =>
                {
                    _logger.LogWarning(ex.ToString()); // Registra la excepción en cada intento fallido.
                });

        // Crea un canal para la publicación del mensaje.
        using (var channel = _persisterConnection.CreateModel())
        {
            // Obtiene el nombre del evento basado en su tipo.
            var eventName = @event.GetType().Name;

            // Declara un intercambio de tipo "direct".
            channel.ExchangeDeclareAsync(exchange: BROKER_NAME, type: "direct").Wait();

            // Serializa el evento a JSON y lo convierte a un arreglo de bytes.
            // var message = JsonSerializer.Serialize(value: @event);
            var message = JsonConvert.SerializeObject(value: @event);
            var body = Encoding.UTF8.GetBytes(message);

            // Ejecuta la publicación dentro de la política de reintento.
            policy.Execute(() =>
            {
                // Crear una nueva instancia de las propiedades básicas para el mensaje.
                var properties = new BasicProperties();

                // Establecer el modo de entrega como persistente, lo que asegura que el mensaje no se pierda
                // en caso de una caída del servidor.
                properties.DeliveryMode = DeliveryModes.Persistent;

                // Publicar el mensaje de manera asincrónica en el canal especificado.
                channel.BasicPublishAsync(
                    exchange: BROKER_NAME,  // Nombre del intercambio donde se publicará el mensaje.
                    routingKey: eventName,  // Clave de enrutamiento basada en el nombre del evento.
                    mandatory: true,         // Indica si el mensaje debe ser entregado a al menos una cola.
                    basicProperties: properties, // Propiedades adicionales del mensaje (en este caso, solo la persistencia).
                    body: body              // Cuerpo del mensaje, usualmente el contenido del evento serializado.
                ).AsTask().Wait();
            });
        }
    }

    /// <summary>
    /// Libera los recursos utilizados por la instancia de <see cref="EventBusRabbitMQ"/>.
    /// Cierra el canal de consumo y limpia los manejadores registrados.
    /// </summary>
    /// <remarks>
    /// Este método es parte de la implementación de la interfaz <see cref="IDisposable"/> y debe llamarse 
    /// para liberar los recursos no administrados como el canal de comunicación con RabbitMQ y las colecciones de manejadores.
    /// </remarks>
    public void Dispose()
    {
        // Verifica si el canal de consumo existe antes de intentar cerrarlo.
        if (_consumerChannel != null)
        {
            _consumerChannel.Dispose();
        }
    }

    /// <summary>
    /// Crea y configura un canal de consumidor para RabbitMQ, que se encarga de 
    /// recibir mensajes y reenviarlos al procesador de eventos correspondiente.
    /// </summary>
    /// <returns>El canal de comunicación configurado.</returns>
    /// <exception cref="Exception">Se lanza si no se puede establecer una conexión con RabbitMQ.</exception>
    private IChannel CreateConsumerChannel()
    {
        // Verifica si la conexión con RabbitMQ está activa; si no, intenta reconectar.
        if (!_persisterConnection.IsConnected)
        {
            _persisterConnection.TryConnect();
        }

        // Crea un canal de comunicación con RabbitMQ.
        var channel = _persisterConnection.CreateModel();

        // Declara un intercambio en RabbitMQ con el tipo "direct".
        channel.ExchangeDeclareAsync(exchange: BROKER_NAME, type: "direct")
            .Wait();

        // Declara una cola con el nombre especificado para recibir los mensajes. La cola es durable, no exclusiva y no se eliminará automáticamente.
        channel.QueueDeclareAsync(queue: _queueName, durable: true, exclusive: false, autoDelete: false, arguments: null)
            .Wait();

        // Crea un consumidor para escuchar mensajes de la cola.
        var consumer = new AsyncEventingBasicConsumer(channel);

        // Define el manejador para los mensajes recibidos.
        consumer.ReceivedAsync += async (model, ea) =>
        {
            var eventName = ea.RoutingKey; // Obtiene el nombre del evento del enrutamiento.
            var message = Encoding.UTF8.GetString(ea.Body.ToArray()); // Decodifica el mensaje recibido.

            // Procesa el evento de manera asincrónica.
            await ProcessEvent(eventName, message);

            // El mensaje ha sido procesado correctamente y puede ser descartado del sistema de mensajería.
            // multiple: false - asegura que solo se confirme el mensaje actual
            await channel.BasicAckAsync(deliveryTag: ea.DeliveryTag, multiple: false);
        };

        // Configura el canal para comenzar a consumir mensajes de la cola especificada.
        // autoAck: Desactiva la confirmación automática para un control manual.
        channel.BasicConsumeAsync(queue: _queueName, autoAck: false, consumer: consumer).Wait();

        // Maneja excepciones en el canal y recrea el canal si es necesario.
        channel.CallbackExceptionAsync += async (sender, ea) =>
        {
            _consumerChannel.Dispose();
            _consumerChannel = CreateConsumerChannel();
        };

        return channel;
    }

    /// <summary>
    /// Procesa un evento recibido identificando sus manejadores registrados 
    /// y ejecutando la lógica correspondiente.
    /// </summary>
    /// <param name="eventName">El nombre del evento recibido.</param>
    /// <param name="message">El mensaje del evento en formato JSON.</param>
    /// <returns>Una tarea asincrónica que representa el proceso de manejo del evento.</returns>
    private async Task ProcessEvent(string eventName, string message)
    {
        await using (var scope = _serviceProvider.CreateAsyncScope())
        {
            // Verifica si hay manejadores registrados para el evento.
            if (_subscriptionInfo.EventTypes.TryGetValue(eventName, value: out var eventType))
            {
                // Deserializa el mensaje JSON al tipo de evento correspondiente.
                // var integrationEvent = JsonSerializer.Deserialize(json: message, returnType: eventType);
                var integrationEvent = JsonConvert.DeserializeObject(value: message, type: eventType) as IntegrationEvent;

                // Obtiene la lista de manejadores registrados para este evento.
                var handlers = scope.ServiceProvider.GetKeyedServices<IIntegrationEventHandler>(serviceKey: eventType);

                // Invoca el método "Handle" de cada manejador de manera asincrónica.
                foreach (var handler in handlers)
                {
                    await handler.Handle(@event: integrationEvent);
                }
            }
        }
    }

    /// <summary>
    /// Inicia la conexión con el sistema de mensajería y enlaza las colas a los eventos configurados.
    /// </summary>
    public Task StartAsync()
    {
        // Inicia una tarea en segundo plano para manejar la configuración del sistema de mensajería.
        _ = Task.Run(() =>
        {
            // Verifica si la conexión con el sistema de mensajería está activa.
            if (!_persisterConnection.IsConnected)
            {
                // Si no está conectada, intenta establecer la conexión.
                _persisterConnection.TryConnect();
            }

            // Crea un canal de comunicación con el sistema de mensajería.
            using (var channel = _persisterConnection.CreateModel())
            {
                // Itera sobre los tipos de eventos configurados en _subscriptionInfo.EventTypes.
                foreach (var (eventName, _) in _subscriptionInfo.EventTypes)
                {
                    // Vincula la cola especificada (_queueName) al intercambio (BROKER_NAME)
                    // y la clave de enrutamiento correspondiente (eventName) de manera asíncrona.
                    // Se utiliza Wait() para bloquear hasta que la operación se complete.
                    channel.QueueBindAsync(
                        queue: _queueName,
                        exchange: BROKER_NAME,
                        routingKey: eventName)
                    .Wait();
                }
            }
        });

        // Retorna una tarea completada inmediatamente, indicando que no hay más acciones a realizar de manera síncrona.
        return Task.CompletedTask;
    }

}
```



```csharp
public class EventBusSubscriptionInfo
{
    public Dictionary<string, Type> EventTypes { get; } = [];
}
```



```csharp
public class RabbitMQOptions
{
    public String EventBusClientName { get; set; }
}
```



```csharp
/// <summary>
/// Define un contrato para manejar la conexión persistente con RabbitMQ.
/// </summary>
public interface IRabbitMQPersisterConnection : IDisposable
{
    /// <summary>
    /// Indica si la conexión con RabbitMQ está actualmente activa.
    /// </summary>
    bool IsConnected { get; }

    /// <summary>
    /// Intenta establecer una conexión con RabbitMQ. 
    /// Si la conexión es exitosa, devuelve true; de lo contrario, devuelve false.
    /// </summary>
    /// <returns>True si la conexión se establece correctamente, false en caso contrario.</returns>
    bool TryConnect();

    /// <summary>
    /// Crea un nuevo modelo de canal (IChannel) para la comunicación con RabbitMQ.
    /// Nota: IModel ha sido renombrado a IChannel en versiones recientes de RabbitMQ.
    /// </summary>
    /// <returns>Un objeto IChannel que representa un canal para la comunicación con RabbitMQ.</returns>
    IChannel CreateModel();
}
```

```csharp
/// <summary>
/// Implementación predeterminada de la conexión persistente con RabbitMQ.
/// Administra la creación, conexión, y el manejo de eventos de desconexión o fallos en la conexión con RabbitMQ.
/// </summary>
public class DefaultRabbitMQPersisterConnection : IRabbitMQPersisterConnection
{
    private readonly IConnectionFactory _connectionFactory;
    private readonly ILogger<DefaultRabbitMQPersisterConnection> _logger;

    IConnection _connection;
    bool _disposed;

    object sync_root = new object();

    /// <summary>
    /// Crea una instancia de <see cref="DefaultRabbitMQPersisterConnection"/>.
    /// </summary>
    /// <param name="connectionFactory">Factory para la creación de conexiones RabbitMQ.</param>
    /// <param name="logger">Logger para registrar información sobre las operaciones de conexión.</param>
    /// <exception cref="ArgumentNullException">Lanzado si <paramref name="connectionFactory"/> o <paramref name="logger"/> son nulos.</exception>
    public DefaultRabbitMQPersisterConnection(IConnectionFactory connectionFactory, ILogger<DefaultRabbitMQPersisterConnection> logger)
    {
        _connectionFactory = connectionFactory ?? throw new ArgumentNullException(nameof(connectionFactory));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }

    /// <summary>
    /// Obtiene el estado de la conexión a RabbitMQ.
    /// </summary>
    public bool IsConnected
    {
        get
        {
            return _connection != null && _connection.IsOpen && !_disposed;
        }
    }

    /// <summary>
    /// Crea un canal de comunicación con RabbitMQ.
    /// Nota: IModel ha sido renombrado a IChannel en versiones recientes de RabbitMQ.
    /// </summary>
    /// <returns>Un objeto de tipo <see cref="IChannel"/> para la interacción con RabbitMQ.</returns>
    /// <exception cref="InvalidOperationException">Lanzado si no hay conexión activa a RabbitMQ.</exception>
    public IChannel CreateModel()
    {
        if (!IsConnected)
        {
            throw new InvalidOperationException("No RabbitMQ connections are available to perform this action");
        }

        return _connection.CreateChannelAsync()
            .GetAwaiter()
            .GetResult();
    }

    /// <summary>
    /// Libera los recursos utilizados por la conexión a RabbitMQ.
    /// </summary>
    public void Dispose()
    {
        if (_disposed) return;

        _disposed = true;

        try
        {
            _connection.Dispose();
        }
        catch (IOException ex)
        {
            _logger.LogCritical(ex.ToString());
        }
    }

    /// <summary>
    /// Intenta establecer una conexión con RabbitMQ.
    /// Utiliza un patrón de reintentos para manejar excepciones comunes como problemas de red o RabbitMQ no disponible.
    /// </summary>
    /// <returns>Devuelve <c>true</c> si la conexión fue exitosa; de lo contrario, <c>false</c>.</returns>
    public bool TryConnect()
    {
        _logger.LogInformation("RabbitMQ Client is trying to connect");

        lock (sync_root)
        {
            var policy = RetryPolicy.Handle<SocketException>()
                .Or<BrokerUnreachableException>()
                .WaitAndRetry(
                retryCount: 5,
                sleepDurationProvider: retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),
                onRetry: (ex, time) =>
                {
                    _logger.LogWarning(ex.ToString());
                });

            policy.Execute(() =>
            {
                _connection = _connectionFactory.CreateConnectionAsync()
                    .GetAwaiter()
                    .GetResult();
            });

            if (IsConnected)
            {
                _connection.ConnectionShutdownAsync += OnConnectionShutdown;
                _connection.CallbackExceptionAsync += OnCallbackException;
                _connection.ConnectionBlockedAsync += OnConnectionBlocked;

                _logger.LogInformation($"RabbitMQ persister connection acquire a connection {_connection.Endpoint.HostName} and is subscribed to failure events");

                return true;
            }
            else
            {
                _logger.LogCritical("FATAL ERROR: RabbitMQ connections can't be created and opened");

                return false;
            }
        }
    }

    /// <summary>
    /// Maneja el evento cuando la conexión a RabbitMQ es bloqueada.
    /// Intenta reconectar automáticamente.
    /// </summary>
    private Task OnConnectionBlocked(object sender, ConnectionBlockedEventArgs e)
    {
        if (_disposed) return Task.CompletedTask;

        _logger.LogWarning("A RabbitMQ connection is shutdown. Trying to re-connect...");

        TryConnect();

        return Task.CompletedTask;
    }

    /// <summary>
    /// Maneja el evento cuando ocurre una excepción en la conexión de RabbitMQ.
    /// Intenta reconectar automáticamente.
    /// </summary>
    Task OnCallbackException(object sender, CallbackExceptionEventArgs e)
    {
        if (_disposed) return Task.CompletedTask;

        _logger.LogWarning("A RabbitMQ connection throw exception. Trying to re-connect...");

        TryConnect();

        return Task.CompletedTask;
    }

    /// <summary>
    /// Maneja el evento cuando la conexión a RabbitMQ se cierra.
    /// Intenta reconectar automáticamente.
    /// </summary>
    Task OnConnectionShutdown(object sender, ShutdownEventArgs reason)
    {
        if (_disposed) return Task.CompletedTask;

        _logger.LogWarning("A RabbitMQ connection is on shutdown. Trying to re-connect...");

        TryConnect();

        return Task.CompletedTask;
    }
}
```
