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

2. Crear un servicio que acceda al microservicio de `basket`
   
   - Define una interfaz para el servicio
     
     - Define un metodo para acceder a la `Basket`
     - Define un metodo para agregar un `BasketItem` a la `Basket`
     - Define un metodo para actualizar la `Basket`
        
     ```csharp
      /// <summary>
      /// Proporciona una abstracción para gestionar la cesta de compra de un usuario en la tienda.
      /// </summary>
      public interface IBasketServices
      {
          /// <summary>
          /// Agrega un producto con el identificador <paramref name="productId"/> a la cesta del <paramref name="user"/>.
          /// </summary>
          /// <param name="user">El usuario al que pertenece la cesta.</param>
          /// <param name="productId">El identificador del producto que se desea agregar a la cesta.</param>
          /// <param name="quantity">La cantidad del producto que se va a agregar.</param>
          /// <returns>
          /// Una <see cref="Task"/> que representa la operación asincrónica y contiene la <see cref="Basket"/> actualizada del usuario.
          /// </returns>
          Task<Basket> AddItemToBasketAsync(ApplicationUser user, int productId, int quantity = 1);
      
          /// <summary>
          /// Recupera la <see cref="Basket"/> asociada al <paramref name="user"/> especificado.
          /// </summary>
          /// <param name="user">El usuario cuya cesta de compras se desea obtener.</param>
          /// <returns>
          /// Una <see cref="Task"/> que representa la operación asincrónica y contiene la <see cref="Basket"/> asociada al usuario.
          /// </returns>
          Task<Basket> GetBasketAsync(ApplicationUser user);
      
          /// <summary>
          /// Actualiza las cantidades de productos en la <see cref="Basket"/> del <paramref name="user"/> especificado.
          /// </summary>
          /// <param name="user">El usuario cuya cesta se desea actualizar.</param>
          /// <param name="quantities">Las nuevas cantidades de los productos.</param>
          /// <returns>
          /// Una <see cref="Task"/> que representa la operación asincrónica y contiene la <see cref="Basket"/> actualizada del usuario.
          /// </returns>
          Task<Basket> SetQuantitiesAsync(ApplicationUser user, Dictionary<string, int> quantities);
      }
     ```
     
3. Crea la clase de implementación del servicio
   
   ```csharp
      /// <summary>
      /// Proporciona el servicio para administrar el <see cref="Basket"/> de un usuario
      /// </summary>
      public class BasketServices : IBasketServices
      {
          private readonly IOptions<AppSettingsJson> _settings;
          private readonly ILogger<BasketServices> _logger;
          private readonly ICatalogServices _catalogServices;
          private readonly HttpClient _httpClient;
          private readonly string _basketUrlApi;
      
          /// <summary>
          /// Inicializa una nueva instancia de la clase <see cref="BasketServices"/>.
          /// </summary>
          /// <param name="settings">Las opciones de configuración, representadas por un objeto <see cref="AppSettingsJson"/>.</param>
          /// <param name="httpClient">El cliente HTTP utilizado para realizar solicitudes.</param>
          /// <param name="catalogServices">El servicio que proporciona acceso a los productos y el catálogo de la tienda.</param>
          /// <param name="logger">El logger utilizado para registrar mensajes y errores durante la ejecución de los métodos en esta clase.</param>
          public BasketServices(IOptions<AppSettingsJson> settings, HttpClient httpClient, ICatalogServices catalogServices, ILogger<BasketServices> logger)
          {
              _logger = logger;
              _httpClient = httpClient;
              _catalogServices = catalogServices;
              _settings = settings;
              _basketUrlApi = String.Format(
                  format: "{0}/api/v1/basket",
                  args: [_settings.Value.BasketUrl]);
          }
      
          /// <summary>
          /// Agrega un producto con el identificador <paramref name="productId"/> a la cesta del <paramref name="user"/>.
          /// </summary>
          /// <param name="user">El usuario al que pertenece la cesta.</param>
          /// <param name="productId">El identificador del producto que se desea agregar a la cesta.</param>
          /// <param name="quantity">La cantidad del producto que se va a agregar.</param>
          /// <returns>
          /// Una <see cref="Task"/> que representa la operación asincrónica y contiene la <see cref="Basket"/> actualizada del usuario.
          /// </returns>
          public async Task<Basket> AddItemToBasketAsync(ApplicationUser user, int productId, int quantity = 1)
          {
              // WebMvc le manda un POST a ApiGw
              // [+] newItem
              // var newItem = new { CatalogItemId = productId, BasketId = user.Id, Quantity = quantity };
      
              // Web.BFF envia un solicitud GET al catalog para obtener el product que se desea agregar a la cesta.
              // Step 1: Get the item from catalog
              var item = await _catalogServices.GetByIdCatalogItemAsync(id: productId);
      
              // Web.BFF envia una solictud GET al basket para obtener la cesta del usuario, si existe.
              // Step 2: Get current basket status
              var basket = await GetBasketAsync(user: user);
      
              // Step 3: Merge current status with new product
              basket.AddBasketItem(item: item, productId: productId, quantity: quantity);
      
              // Web.BFF envia una solicitud POST al basket para actualizar la cesta del usuario.
              // Step 4: Update basket
              var updateUri = URI.Basket.UpdateBasketUri(baseUri: _basketUrlApi);
      
              var basketContent = new StringContent(
                  content: JsonSerializer.Serialize(value: basket),
                  encoding: System.Text.Encoding.UTF8,
                  mediaType: "application/json");
              
              var data = await _httpClient.PostAsync(requestUri: updateUri, content: basketContent);
              
              return await DeserializeBasketAsync(content: data.Content);
          }
      
          /// <summary>
          /// Recupera la <see cref="Basket"/> asociada al <paramref name="user"/> especificado.
          /// </summary>
          /// <param name="user">El usuario cuya cesta de compras se desea obtener.</param>
          /// <returns>
          /// Una <see cref="Task"/> que representa la operación asincrónica y contiene la <see cref="Basket"/> asociada al usuario.
          /// </returns>
          public async Task<Basket> GetBasketAsync(ApplicationUser user)
          {
              var uri = URI.Basket.GetBasketUri(baseUri: _basketUrlApi, id: user.Id);
      
              var responseString = await _httpClient.GetStringAsync(requestUri: uri);
      
              return String.IsNullOrEmpty(value: responseString)
                  ? new Basket(buyerId: user.Id)
                  : DeserializeBasket(responseJson: responseString);
          }
      
          /// <summary>
          /// Actualiza las cantidades de productos en la <see cref="Basket"/> del <paramref name="user"/> especificado.
          /// </summary>
          /// <param name="user">El usuario cuya cesta se desea actualizar.</param>
          /// <param name="quantities">Las nuevas cantidades de los productos.</param>
          /// <returns>
          /// Una <see cref="Task"/> que representa la operación asincrónica y contiene la <see cref="Basket"/> actualizada del usuario.
          /// </returns>
          public async Task<Basket> SetQuantitiesAsync(ApplicationUser user, Dictionary<string, int> quantities)
          {
              // Step 1: Retrieve the current basket
              var curBasket = await GetBasketAsync(user: user);
              
              if (curBasket == null)
              {
                  return null;
              }
      
              // Step 2: Update with new quantities
              foreach(var update  in quantities)
              {
                  var item = curBasket.Items.Find(match: x => x.Id == update.Key);
                  item.Quantity = update.Value;
              }
      
              // Web.BFF envia una solicitud POST al basket para actualizar la cesta del usuario.
              // Save the update basket
              var updateUri = URI.Basket.UpdateBasketUri(baseUri: _basketUrlApi);
      
              var basketContent = new StringContent(
                  content: JsonSerializer.Serialize(value: curBasket),
                  encoding: System.Text.Encoding.UTF8,
                  mediaType: "application/json");
      
              var data = await _httpClient.PostAsync(requestUri: updateUri, content: basketContent);
      
              return await DeserializeBasketAsync(content: data.Content);
          }
      
          /// <summary>
          /// Deserializa el contenido HTTP en un objeto de tipo <see cref="Basket"/>.
          /// </summary>
          /// <param name="content">El contenido HTTP que contiene los datos JSON para deserializar.</param>
          /// <returns>
          /// Una <see cref="Task"/> que representa el proceso asincrónico y contiene <see cref="Basket"/> deserializado.
          /// </returns>
          public async Task<Basket> DeserializeBasketAsync(HttpContent content)
          {
              return DeserializeBasket(
                  responseJson: await content.ReadAsStringAsync());
          }
      
          /// <summary>
          /// Deserializa una cadena JSON en un objeto de tipo <see cref="Basket"/>.
          /// </summary>
          /// <param name="responseJson">La cadena JSON que representa un objeto <see cref="Basket"/>.</param>
          /// <returns>Un objeto deserializado de tipo <see cref="Basket"/>.</returns>
          public Basket DeserializeBasket(string responseJson)
          {
              JsonNode basketJson = JsonNode.Parse(json: responseJson)!;
              JsonArray jsonItems = basketJson["items"]!.AsArray();
      
              return new Basket(buyerId: basketJson["buyerId"]!.GetValue<String>())
              {
                  Items = jsonItems.Select(selector: static jsonNode =>
                  {
                      return new BasketItem
                      {
                          Id = jsonNode["id"]!.GetValue<String>(),
                          ProductId = jsonNode["productId"]!.GetValue<String>(),
                          ProductName = jsonNode["productName"]!.GetValue<String>(),
                          UnitPrice = jsonNode["unitPrice"]!.GetValue<Decimal>(),
                          OldUnitPrice = jsonNode["oldUnitPrice"]!.GetValue<Decimal>(),
                          Quantity = jsonNode["quantity"]!.GetValue<Int32>(),
                          PictureUrl = jsonNode["pictureUrl"]!.GetValue<String>()
                      };
                  })
                  .ToList()
              };
          }
      }
   ```
   
   > **NOTA**
   > <br/>Esta clase usará `HttpClient` para comunicarse con el microservicio.
   > <br/>Recuerda pasar el `access_token` en la cabecera de la solicitud, en este caso se utiliza
   > ```csharp
   >  services.AddHttpClient<IBasketServices, BasketServices>()
   >       .AddHttpMessageHandler<HttpClientAuthorizationDelegatingHandler>();
   > ```
   
4. Crea un controlador para consumir el servico
   
   ```csharp
      /// <summary>
      /// Controlador que maneja las operaciones relacionadas con <see cref="IBasketServices"/>.
      /// </summary>
      [Authorize]
      public class BasketController : Controller
      {
          private readonly IIdentityParser<ApplicationUser> _identityParser;
          private readonly IBasketServices _basketSvc;
      
          /// <summary>
          /// Inicializa una nueva instancia de la clase <see cref="BasketController"/>.
          /// </summary>
          /// <param name="identityParser">El extrae la información del usuario en un objeto <see cref="ApplicationUser"/>.</param>
          /// <param name="basketSvc">El servicio que maneja la lógica de negocio relacionada con <see cref="Basket"/>.</param>
          public BasketController(IIdentityParser<ApplicationUser> identityParser, IBasketServices basketSvc)
          {
              _identityParser = identityParser;
              _basketSvc = basketSvc;
          }
      
          /// <summary>
          /// Muestra la cesta de compras del usuario.
          /// </summary>
          /// <returns>Una vista que muestra el contenido de la cesta de compras del usuario.</returns>
          public async Task<IActionResult> Index()
          {
              var user = _identityParser.Parse(principal: HttpContext.User);
              var vm = await _basketSvc.GetBasketAsync(user:  user);
      
              return View(model: vm);
          }
      
          /// <summary>
          /// Actualiza las cantidades de los artículos en la <see cref="Basket"/>.
          /// </summary>
          /// <param name="quantities">La lista de los productos y sus cantidades.</param>
          /// <param name="action">La acción que se desea realizar.</param>
          /// <returns>Una vista actualizada con el contenido de la cesta.</returns>
          [HttpPost]
          public async Task<IActionResult> Index(Dictionary<string, int> quantities, string action)
          {
              if (!quantities.Any())
              {
                  return BadRequest(error: "No updates sent");
              }
      
              var user = _identityParser.Parse(principal: HttpContext.User);
              var basket = await _basketSvc.SetQuantitiesAsync(user: user, quantities: quantities);
      
              if (basket == null)
              {
                  return BadRequest(error: $"Basket with ID {user.Id} not found.");
              }
      
              if (action == "[ CHECKOUT ]")
              {
                  return RedirectToAction(actionName: "Create", controllerName: "Order");
              }
      
              return View (model: basket);
          }
      
          /// <summary>
          /// Agrega un artículo a la cesta de compras del usuario.
          /// </summary>
          /// <param name="productDetails">El objeto que contiene los detalles del producto que se va a agregar a la cesta.</param>
          /// <param name="quantity">La cantidad del producto a agregar a la cesta (valor por defecto es 1).</param>
          /// <returns>Una vista con la cesta de compras actualizada.</returns>
          public async Task<IActionResult> AddToBasket(CatalogItem productDetails, int quantity = 1)
          {
              if (productDetails?.Id != null)
              {
                  var user = _identityParser.Parse(principal: HttpContext.User);
      
                  var basket = await _basketSvc.AddItemToBasketAsync(user: user, productId: productDetails.Id, quantity: quantity);
      
                  var curItem = basket.Items.Find(match: itm => itm.ProductId.Equals(value: productDetails.Id.ToString()));
      
                  ViewData["COUNT_ITEM"] = curItem.Quantity;
      
                  return View(model:  productDetails);
              }
      
              return RedirectToAction(
                  actionName: "Index",
                  controllerName: "Catalog");
          }
      }
   ```
   
5. Agrega un `ViewComponent` para representar los articulos en `Basket`

   ```csharp
      /// <summary>
      /// Componente de vista que muestra el número de artículos en la cesta de compras del usuario.
      /// </summary>
      public class BasketCounter : ViewComponent
      {
          private readonly IBasketServices _basketServices;
      
          /// <summary>
          /// Inicializa una nueva instancia de la clase <see cref="BasketCounter"/>.
          /// </summary>
          /// <param name="basketServices">Servicio que maneja la lógica de la cesta de compras.</param>
          public BasketCounter(IBasketServices basketServices)
          {
              _basketServices = basketServices;
          }
      
          /// <summary>
          /// Ejecuta la lógica del componente para recuperar la cantidad de artículos en la cesta del usuario.
          /// </summary>
          /// <param name="User">Usuario cuya cesta se va acontar</param>
          /// <returns>Un resultado de vista con el número de artículos en la cesta.</returns>
          public async Task<IViewComponentResult> InvokeAsync(ApplicationUser User)
          {
              var vcm = new BasketCounterViewComponentModel();
      
              var basket = await _basketServices.GetBasketAsync(user: User);
      
              vcm.ItemsCount = basket.Items.Sum(selector: x => x.Quantity);
      
              return View(model: vcm);
          }
      }
   ```
   
6. Registrar los servicios en el injector de dependencias
   
   ```csharp
      services.AddTransient<HttpClientAuthorizationDelegatingHandler>();
      services.AddSingleton<IHttpContextAccessor, HttpContextAccessor>();
      
      services.AddHttpClient<IBasketServices, BasketServices>()
          .AddHttpMessageHandler<HttpClientAuthorizationDelegatingHandler>();
      services.AddTransient<IIdentityParser<ApplicationUser>, IdentityParser>();
   ```

   > **NOTA**
   > <br/>No olvides agregar el `options.Scope.Add("basket")`
