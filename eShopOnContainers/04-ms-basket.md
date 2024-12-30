# [Designing and Developing Multi-Container and Microservice-Based .NET Applications](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/multi-container-microservice-net-applications/)

The Basket microservice manages temporal data about product items that users are adding to their 
shopping baskets, which includes the price of the items at the time they were added to the basket. 
When a product’s price is updated in the catalog, that price should also be updated in the active 
baskets that hold that same product, plus the system should probably warn the user saying that a 
particular item’s price has changed since they added it to their basket.

> [!NOTE]
> The Catalog microservice shouldn’t update the Basket table directly, because the Basket table is 
> owned by the Basket microservice. To make an update to the Basket microservice, the Catalog 
> microservice should use eventual consistency probably based on asynchronous communication such 
> as integration events (message and event-based communication). This is how the eShopOnContainers
> reference application performs this type of consistency across microservices


### Microservicio Basket: Creación de la capa de Dominio

1. Crear el modelo `BasketItem` que represente el producto que el cliente ha agregado a la cesta o carrito `Basket`.
   
   ```csharp
   public class BasketItem : IValidatableObject
   {
       public string Id { get; set; }
       public string ProductId { get; set; }
       public string ProductName { get; set; }
       public decimal UnitPrice { get; set; }
       public decimal OldUnitPrice { get; set; }
       public int Quantity { get; set; }
       public string PictureUrl { get; set; }
   }
    ```

2. Creacion del modelo `CostumerBasket` que representa la cesta/carrito del cliente `Costumer`.

   ```csharp
   public class CustomerBasket
   {
       public string BuyerId { get; set; }

       public List<BasketItem> Items { get; set; }

       public CustomerBasket(string buyerId)
       {
           BuyerId = buyerId;
           Items = new List<BasketItem>();
       }
   }
   ```
   


### Microservicio Basket: Creacion de la capa infrastructura

El microservicio de carrito de compras (`Basket`) maneja datos que son **transitorios** y generalmente no requieren almacenamiento a largo plazo. Un carrito de compras **es un recurso temporal** que los usuarios llenan y vacían rápidamente durante una sesión.

`Redis`, al ser una base de datos en memoria, es ideal para manejar este tipo de datos volátiles, ya que permite una **lectura y escritura extremadamente rápidas**.

1. Instalar el paquete
   
   ```powershell
   dotnet add package StackExchange.Redis --version 2.8.24
   ```

2. Cree una abstracion para administrar la persistencia de los datos
   
   ```csharp
   public interface IBasketRepository
   {
       Task<CustomerBasket> GetBasketAsync(string customerId);
       Task<CustomerBasket> UpdateBasketAsync(CustomerBasket basket);
   }
   ```
   
4. Implemente la interfaz para interactuar con Redis

   ```csharp
    public class BasketOnRedisRepository : IBasketRepository
    {
        private readonly ILogger<BasketOnRedisRepository> _logger;
        private readonly ConnectionMultiplexer _redis;
        private readonly IDatabase _database;
    
        public BasketOnRedisRepository(ILogger<BasketOnRedisRepository> logger, ConnectionMultiplexer redis)
        {
            _logger = logger;
            _redis = redis;
            _database = _redis.GetDatabase();
        }
    
        public async Task<CustomerBasket> GetBasketAsync(string customerId)
        {
            var data = await _database.StringGetAsync(key: customerId);
    
            if (data.IsNullOrEmpty)
            {
                return null;
            }
    
            return JsonSerializer.Deserialize<CustomerBasket>(json: data)!;
        }
    
        public async Task<CustomerBasket> UpdateBasketAsync(CustomerBasket basket)
        {
            var created = await _database.StringSetAsync(
                key: basket.BuyerId,
                value: JsonSerializer.Serialize(value: basket));
    
            if (!created)
            {
                _logger.LogInformation(message: "Problem occur persisting the item.");
    
                return null;
            }
    
            _logger.LogInformation(message: "Basket item persisted succesfully.");
    
            return await GetBasketAsync(customerId: basket.BuyerId);
        }
    }
   ```


### Microservicio Basket: Creación de la capa de Application

1. Cree el controlador con el cual se va acceder a la informacion de la cesta `Basket`

   ```csharp
    [Route(template: "api/v1/[controller]")]
    public class BasketController : Controller
    {
        private readonly ILogger<BasketController> _logger;
        private readonly IBasketRepository _repository;
    
        public BasketController(IBasketRepository basketRepository, ILogger<BasketController> logger)
        {
            _repository = basketRepository;
            _logger = logger;
        }
    
        // GET /id
        [HttpGet(template: "{id}")]
        [ProducesResponseType(typeof(CustomerBasket), (int)HttpStatusCode.OK)]
        public async Task<IActionResult> Get(string id)
        {
            var basket = (await _repository.GetBasketAsync(customerId: id))
                ?? new CustomerBasket(buyerId: id);
    
            return Ok(value: basket);
        }
    
        // POST /value
        [HttpPost]
        [ProducesResponseType(typeof(CustomerBasket), (int)HttpStatusCode.OK)]
        public async Task<IActionResult> Post([FromBody] CustomerBasket value)
        {
            var basket = await _repository.UpdateBasketAsync(basket: value);
            
            return Ok(value: basket);
        }
    }
   ```
   
2. Configurar los servicios y el flujo de procesamiento de solicitudes utilizando una clase `Startup`.
   
   - Agregue los servicios
     
     - Agregue el controlador
       
       ```csharp
       services.AddMvc(setupAction: o => o.EnableEndpointRouting = false)
           .AddControllersAsServices();
       ```
       
     - Agregue las opciones para proporcionar [acceso fuertemente tipado](https://learn.microsoft.com/es-es/dotnet/core/extensions/options) a grupos de configuraciones

       ```csharp
       services.AddOptions();
       services.Configure<AppSettingsJson>(config: Configuration);
       ```

     - Configure y Registre Redis

       ```csharp
         services.AddSingleton<ConnectionMultiplexer>(implementationFactory: serviceProvider =>
         {
             var option = serviceProvider.GetRequiredService<IOptions<AppSettingsJson>>();
             var appSettingsJson = option.Value;
         
             var configuration = ConfigurationOptions.Parse(
                 configuration: appSettingsJson.ConnectionString,
                 ignoreUnknown: true);
         
             configuration.ResolveDns = true;
         
             return ConnectionMultiplexer.Connect(configuration: configuration);
         });
         
         services.AddTransient<IBasketRepository, BasketOnRedisRepository>();
       ```
       
       > **NOTA**
       > <br/>En Redis, el puerto predeterminado es `6379`, que se utiliza automáticamente si no se especifica explícitamente en la cadena de conexión.

     - Agrega seguridad a la API
       
       ```csharp
         /// ASP.NET Core adds default namespaces to some known claims, which might not be required in the app.
         /// Optionally, disable these added namespaces and use the exact claims that the OpenID Connect server created.
         // JwtSecurityTokenHandler.DefaultInboundClaimTypeMap.Clear();
         // JwtSecurityTokenHandler.DefaultOutboundClaimTypeMap.Clear();
         
         services
             .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
             .AddJwtBearer(options => {
         
                 options.Authority = Configuration["IdentityUrl"];
         
                 options.Audience = "basket";
         
                 options.RequireHttpsMetadata = false;
         
                 options.IncludeErrorDetails = true;
         
                 options.MapInboundClaims = false;
         
                 options.Events = new JwtBearerEvents
                 {
                     OnAuthenticationFailed = context =>
                     {
                         Console.WriteLine($"Error en la autenticación: {context.Exception.Message}");
                         return Task.CompletedTask;
                     }
                 };
             });
       ```

       > **NOTA**
       > <br/>[Disabling JWT access token encryption](https://documentation.openiddict.com/configuration/token-formats#disabling-jwt-access-token-encryption)
       > <br/>By default, the OpenIddict server enforces encryption for all the token types it supports.
       > ```csharp
       > services.AddOpenIddict()
       >     .AddServer(options =>
       >      {
       >         // Error en la autenticación: IDX10618:
       >         //   Key unwrap failed using decryption Keys:
       >         //     '[PII of type 'System.String' is hidden. For more details, see https://aka.ms/IdentityModel/PII.]'.
       >         options.DisableAccessTokenEncryption();
       >
       >         options.Configure(options =>
       >         {
       >             // Emitir "typ": "JWT" en lugar de "at+jwt" para Access Tokens.
       >             options.DisableAccessTokenEncryption = true;
       >         });
       >     });
       > ```

<br/>
<br/>
<br/>

## Web MVC Application: Modificar el cliente
Como se mencionó anteriormente, los datos que posee cada microservicio son privados para ese microservicio y solo se puede acceder a ellos mediante su **API** de microservicio. Por lo tanto, uno de los desafíos que se presenta es cómo implementar procesos de comunicación de extremo a extremo (`end-to-end business processes`) y, al mismo tiempo, mantener la coherencia en varios microservicios.

1. Modificar el dominio de la aplicación Web Mvc
   
   - Agregar el modelo que represente los producto `BasketItem` en la cesta `Basket`
     
     ```csharp
     public class BasketItem
     {
          public string Id { get; set; }
          public string ProductId { get; set; }
          public string ProductName { get; set; }
          public decimal UnitPrice { get; set; }
          public decimal OldUnitPrice { get; set; }
          public int Quantity { get; set; }
          public string PictureUrl { get; set; }
     }
     ```
     
   - Agregar el modelo que represente la cesta `Basket` de productos
       
       ```csharp
         public class Basket
         {
             public string BuyerId { get; set; }
         
             public List<BasketItem> Items { get; set; }
         
             public Basket(string buyerId)
             {
                 BuyerId = buyerId;
                 Items = new List<BasketItem>();
             }
         }
       ```
     
     > **NOTA**
     > <br/>No construyas modelos anemicos, introduce logica dentro de ellos
     > 1. Agrega un metodo para añadir productos a la cesta
     >    
     >    ```csharp
     >    public void AddBasketItem(CatalogItem item, int productId, int quantity)
     >    {
     >        var curItem = Items.Find(match: itm => itm.ProductId.Equals(value: productId.ToString()));
     >    
     >        if (curItem == null)
     >        {
     >            Items.Add(item: new BasketItem
     >            {
     >                Id = Guid.NewGuid().ToString(),
     >                ProductId = item.Id.ToString(),
     >                ProductName = item.Name,
     >                UnitPrice = item.Price,
     >                Quantity = quantity,
     >                PictureUrl = item.PictureUri,
     >            });
     >        }
     >        else
     >        {
     >            curItem.Quantity = quantity;
     >        }
     >    }
     >    ```
     >    
     > 2. Calcula el precio total de la cesta
     >    
     >    ```csharp
     >    public decimal Total()
     >        {
     >            return Math.Round(
     >                d: Items.Sum(
     >                    selector: s => s.UnitPrice *  s.Quantity), 
     >                decimals: 2);
     >        }
     >    ```
   - -




