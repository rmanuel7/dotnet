```markdown
info: Aspire.Dashboard.Startup[0]
      Aspire version: 1.0.0
info: Aspire.Dashboard.Startup[0]
      Now listening on: https://localhost:7173
info: Aspire.Dashboard.Startup[0]
      OTLP/gRPC listening on: https://localhost:18889
warn: Aspire.Dashboard.Startup[0]
      OTLP server is unsecured. Untrusted apps can send telemetry to the dashboard. For more information, visit https://go.microsoft.com/fwlink/?linkid=2267030
info: Aspire.Dashboard.Startup[0]
      Login to the dashboard at https://localhost:7173/login?t=f8c79cd1537022cbc1234543b4643d2b
dbug: Aspire.Dashboard.Otlp.OtlpLogsService[0]
      Processed logs export. Failure count: 0
dbug: Aspire.Dashboard.Services.DashboardClient[0]
      Dashboard configured to connect to: https://localhost:18891/
warn: Aspire.Dashboard.Components.BrowserStorage.SessionBrowserStorage[0]
      Error when reading 'Aspire_ConsoleLog_Filters' as ConsoleLogsFilters.
      System.InvalidOperationException: JavaScript interop calls cannot be issued at this time. This is because the component is being statically rendered. When prerendering is enabled, JavaScript interop calls can only be performed during the OnAfterRenderAsync lifecycle method.
         at Microsoft.AspNetCore.Components.Server.Circuits.RemoteJSRuntime.BeginInvokeJS(Int64 asyncHandle, String identifier, String argsJson, JSCallResultType resultType, Int64 targetInstanceId)
         at Microsoft.JSInterop.JSRuntime.InvokeAsync[TValue](Int64 targetInstanceId, String identifier, CancellationToken cancellationToken, Object[] args)
         at Microsoft.JSInterop.JSRuntime.InvokeAsync[TValue](Int64 targetInstanceId, String identifier, Object[] args)
         at Microsoft.AspNetCore.Components.Server.ProtectedBrowserStorage.ProtectedBrowserStorage.GetAsync[TValue](String purpose, String key)
         at Aspire.Dashboard.Components.BrowserStorage.BrowserStorageBase.GetAsync[TValue](String key) in C:\Users\ADMIN\Documents\X-GitHub\Aspire\Aspire.Dashboard\src\Aspire.Dashboard\Components\BrowserStorage\BrowserStorageBase.cs:line 28
dbug: Aspire.Dashboard.Components.LogViewer[0]
      Log entries changed.
info: Grpc.Net.Client.Internal.GrpcCall[3]
      Call failed with gRPC error status. Status code: 'Unavailable', Message: 'Error connecting to subchannel.'.
      System.Net.Sockets.SocketException (10061): No se puede establecer una conexión ya que el equipo de destino denegó expresamente dicha conexión.
         at System.Net.Sockets.Socket.AwaitableSocketAsyncEventArgs.ThrowException(SocketError error, CancellationToken cancellationToken)
         at System.Net.Sockets.Socket.AwaitableSocketAsyncEventArgs.System.Threading.Tasks.Sources.IValueTaskSource.GetResult(Int16 token)
         at System.Net.Sockets.Socket.<ConnectAsync>g__WaitForConnectWithCancellation|285_0(AwaitableSocketAsyncEventArgs saea, ValueTask connectTask, CancellationToken cancellationToken)
         at Grpc.Net.Client.Balancer.Internal.SocketConnectivitySubchannelTransport.TryConnectAsync(ConnectContext context, Int32 attempt)
info: Grpc.Net.Client.Internal.GrpcCall[3]
      Call failed with gRPC error status. Status code: 'Unavailable', Message: 'Error connecting to subchannel.'.
      System.Net.Sockets.SocketException (10061): No se puede establecer una conexión ya que el equipo de destino denegó expresamente dicha conexión.
         at System.Net.Sockets.Socket.AwaitableSocketAsyncEventArgs.ThrowException(SocketError error, CancellationToken cancellationToken)
         at System.Net.Sockets.Socket.AwaitableSocketAsyncEventArgs.System.Threading.Tasks.Sources.IValueTaskSource.GetResult(Int16 token)
         at System.Net.Sockets.Socket.<ConnectAsync>g__WaitForConnectWithCancellation|285_0(AwaitableSocketAsyncEventArgs saea, ValueTask connectTask, CancellationToken cancellationToken)
         at Grpc.Net.Client.Balancer.Internal.SocketConnectivitySubchannelTransport.TryConnectAsync(ConnectContext context, Int32 attempt)
info: Grpc.Net.Client.Internal.GrpcCall[3]
      Call failed with gRPC error status. Status code: 'Unavailable', Message: 'Error connecting to subchannel.'.
      System.Net.Sockets.SocketException (10061): No se puede establecer una conexión ya que el equipo de destino denegó expresamente dicha conexión.
         at System.Net.Sockets.Socket.AwaitableSocketAsyncEventArgs.ThrowException(SocketError error, CancellationToken cancellationToken)
         at System.Net.Sockets.Socket.AwaitableSocketAsyncEventArgs.System.Threading.Tasks.Sources.IValueTaskSource.GetResult(Int16 token)
         at System.Net.Sockets.Socket.<ConnectAsync>g__WaitForConnectWithCancellation|285_0(AwaitableSocketAsyncEventArgs saea, ValueTask connectTask, CancellationToken cancellationToken)
         at Grpc.Net.Client.Balancer.Internal.SocketConnectivitySubchannelTransport.TryConnectAsync(ConnectContext context, Int32 attempt)
```
