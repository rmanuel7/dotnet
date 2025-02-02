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
