## Order Draft

### Crear el Dominio

1. Agregar un modelo para las direccion de entrega de la orden
   
   ```csharp
    public class Address
    {
        public String Street { get; private set; }
    
        public String City { get; private set; }
    
        public String State { get; private set; }
    
        public String Country { get; private set; }
    
        public String ZipCode { get; private set; }
    
        private Address() { }
    
        public Address(string street, string city, string state, string country, string zipCode)
        {
            Street = street;
            City = city;
            State = state;
            Country = country;
            ZipCode = zipCode;
        }
    }
   ```

2. Agregar un modelo para los productos en la orden

   ```csharp
    public class OrderItem
    {
        public Int32 Id { get; private set; }
    
        public Int32 ProductId { get; private set; }
    
        public String ProductName { get; private set; }
    
        public Decimal UnitPrice { get; private set; }
    
        public String PictureUrl { get; private set; }
    
        public Decimal Discount { get; private set; }
    
        public Int32 Units { get; private set; }
    
        protected OrderItem() { }
    
        public OrderItem(int productId, string productName, decimal unitPrice, string pictureUrl, decimal discount, int units = 1)
        {
            if (units <= 0)
            {
                throw new ArgumentOutOfRangeException(message: "Invalid number of nunits", paramName: nameof(units), actualValue: units);
            }
    
            if ((unitPrice * units) < discount)
            {
                throw new ArgumentOutOfRangeException(message: "The total of order item is lower than applied discount", paramName: nameof(discount), actualValue: discount);
            }
    
            ProductId = productId;
            ProductName = productName;
            UnitPrice = unitPrice;
            Discount = discount;
            Units = units;
            PictureUrl = pictureUrl;
        }
    }
   ```

   > **Nota**
   > <br/>No crees modelos anemicos, Agrega logica relacionada.
   > ```csharp
   >     public void SetNewDiscount(decimal discount)
   >     {
   >         if (discount < 0)
   >         {
   >             throw new ArgumentOutOfRangeException(message: "Discount is not valid", paramName: nameof(discount), actualValue: discount);
   >         }
   > 
   >         Discount = discount;
   >     }
   > 
   >     public void AddUnits(int units)
   >     {
   >         if (units < 0)
   >         {
   >             throw new ArgumentOutOfRangeException(message: "Invalid units", paramName: nameof(units), actualValue: units); 
   >         }
   > 
   >         Units += units;
   >     }
   > ```

3. Define un modelo para los estados de la orden

   ```csharp
    public class OrderStatus
    {
        public Int32 Id { get; set; }
    
        public String Name { get; private set; }
    
        protected OrderStatus() { }
    
        public OrderStatus(int id, string name)
        {
            Id = id;
            Name = name;
        }
    
        public static OrderStatus Submitted = new OrderStatus(id: 1, name: nameof(Submitted).ToLowerInvariant());
    
        public static OrderStatus AwaitingValidation = new OrderStatus(id: 2, name: nameof(AwaitingValidation).ToLowerInvariant());
    
        public static OrderStatus StockConfirmed = new OrderStatus(id: 3, name: nameof(StockConfirmed).ToLowerInvariant());
    
        public static OrderStatus Paid = new OrderStatus(id: 4, name: nameof(Paid).ToLowerInvariant());
    
        public static OrderStatus Shipped = new OrderStatus(id: 5, name: nameof(Shipped).ToLowerInvariant());
    
        public static OrderStatus Cancelled = new OrderStatus(id: 6, name: nameof(Cancelled).ToLowerInvariant());
    }
   ```

4. Define un modelo que presente la orden

   ```csharp
    public class CostumerOrder
    {
        private readonly List<OrderItem> _orderItems;
    
        public Int32 Id { get; private set; }
    
        public DateTime OrderDate { get; private set; }
    
        public Address Address { get; private set; }
    
        public Int32? BuyerId { get; private set; }
    
        public Int32 OrderStatusId { get; private set; }
    
        public String Description { get; private set; }
    
        public Boolean IsDraft { get; private set; }
    
        public Int32? PaymentMethodId { get; private set; }
    
        public IReadOnlyCollection<OrderItem> OrderItems => _orderItems;
    
        public OrderStatus OrderStatus { get; private set; }
    
    
        protected CostumerOrder()
        {
            _orderItems = new List<OrderItem>();
            IsDraft = false;
        }
    
        public CostumerOrder(Address address, int? buyerId, int? paymentMethodId)
        {
            OrderDate = DateTime.UtcNow;
            Address = address;
            BuyerId = buyerId;
            OrderStatusId = OrderStatus.Submitted.Id;
            PaymentMethodId = paymentMethodId;
        }
    }
   ```

   > **NOTA**
   > <br/>No construyas modelos anemicos, agrega logica relacionada
   > ```csharp
   >     public static CostumerOrder NewDraft()
   >     {
   >         var order = new CostumerOrder();
   > 
   >         order.IsDraft = true;
   > 
   >         return order;
   >     }
   > 
   >     public void AddOrderItem(int productId, string productName, decimal unitPrice, decimal discount, string pictureUrl, int units = 1)
   >     {
   >         var existingOrderForProduct = _orderItems.SingleOrDefault(predicate: o => o.ProductId == productId);
   > 
   >         if (existingOrderForProduct != null)
   >         {
   >             // If previous line exist modify it with higher discount and units
   >             
   >             if (discount > existingOrderForProduct.Discount)
   >             {
   >                 existingOrderForProduct.SetNewDiscount(discount: discount);
   >             }
   > 
   >             existingOrderForProduct.AddUnits(units: units);
   >         }
   >         else
   >         {
   >             var orderItem = new OrderItem(productId: productId, productName: productName, unitPrice: unitPrice, pictureUrl: pictureUrl, discount: discount, units: units);
   > 
   >             _orderItems.Add(orderItem);
   >         }
   >     }
   > ```

<br/>
<br/>
<br/>

### Crear la Infrastructura: Si es necesaria la persistencia


<br/>
<br/>
<br/>

### Crear la capa de aplicación

1. Agregar los modelos para los datos que se van a utilizar en el controlador.
   
   - Modelo para recibir la cesta `CustomerBasket` del comprador
     
     - Crear un modelo para representar los productos en la cesta `BasketItem`
       
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
       
     - Crear un modelo para representar la cesta

       ```csharp
        public class OrderDraftViewModel
        {
            public string BuyerId { get; private set; }
        
            public IEnumerable<BasketItem> Items { get; private set; }
        
            public OrderDraftViewModel(string buyerId, IEnumerable<BasketItem> items)
            {
                BuyerId = buyerId;
                Items = items;
            }
        }
       ```
2. Crear el controlador que va ha representar las operaciones de en las ordenes

   ```csharp
    [Route(template: "api/v1/[controller]")]
    [Authorize]
    [ApiController]
    public class OrdersController : Controller
    {
        [Route(template: "draft")]
        [HttpPost]
        [ProducesResponseType(type: typeof(CostumerOrder), statusCode: (int)HttpStatusCode.OK)]
        public async Task<IActionResult> GetOrderDraftFromBasketAsync([FromBody] OrderDraftViewModel orderDraft)
        {
            var order = await Task.Run(function: () =>
            {
                var oDraft = CostumerOrder.NewDraft();
    
                foreach (var item in orderDraft.Items)
                {
                    oDraft.AddOrderItem(
                        productId: Convert.ToInt32(value: item.ProductId),
                        productName: item.ProductName,
                        unitPrice: item.UnitPrice,
                        discount: 0,
                        pictureUrl: item.PictureUrl,
                        units: item.Quantity);
                }
    
                return oDraft;
            });
    
            return Ok(value: order);
        }
    }
   ```

3. Configurar los servicios en el injector de dependencias

   - Agrega el controlador

     ```csharp
      services.AddMvc(setupAction: o => o.EnableEndpointRouting = false)
          .AddControllersAsServices();
     ```

   - Agrega seguridad a la aplicación

     ```csharp
        services.AddAuthentication(configureOptions: o =>
        {
            o.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
            o.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;

        }).AddJwtBearer(configureOptions: o =>
        {
            o.Audience = "orders";
            o.Authority = Configuration["IdentityUrl"];
            o.RequireHttpsMetadata = false;
            o.MapInboundClaims = false;
            o.IncludeErrorDetails = true;
            o.Events = new JwtBearerEvents
            {
                OnAuthenticationFailed = authContext =>
                {
                    Console.WriteLine(format: "Authentication failed: {0}", arg: [authContext.Exception.Message]);
                    return Task.CompletedTask;
                }
            };
        });
     ```

<br/>
<br/>
<br/>

## Configura el cliente que va a consumitr la API

