```csharp
public class ProjectResourceOptions
{
    public string? LaunchProfileName { get; set; }
    public bool ExcludeLaunchProfile { get; set; }
    public bool ExcludeKestrelEndpoints { get; set; }
}

/////////////////////////////////////////////////////////////

public interface IProjectMetadata : IResourceAnnotation
{
    public string ProjectPath { get; }
    public LaunchSettings? LaunchSettings => null;
    internal IConfiguration? Configuration => null;
}

/////////////////////////////////////////////////////////////

internal sealed class DistributedApplicationResourceBuilder<T>(IDistributedApplicationBuilder applicationBuilder, T resource) : IResourceBuilder<T> where T : IResource
{
    public T Resource { get; } = resource;
    public IDistributedApplicationBuilder ApplicationBuilder { get; } = applicationBuilder;

    public IResourceBuilder<T> WithAnnotation<TAnnotation>(TAnnotation annotation, ResourceAnnotationMutationBehavior behavior = ResourceAnnotationMutationBehavior.Append) where TAnnotation : IResourceAnnotation
    {
    }
}

/////////////////////////////////////////////////////////////

public class ProjectResource(string name) : Resource(name), IResourceWithEnvironment, IResourceWithArgs, IResourceWithServiceDiscovery, IResourceWithWaitSupport
{
    internal Dictionary<EndpointAnnotation, string> KestrelEndpointAnnotationHosts
    internal bool HasKestrelEndpoints 
    internal EndpointAnnotation? DefaultHttpsEndpoint

    internal bool ShouldInjectEndpointEnvironment(EndpointReference e)
}
```

---

```
DistributedApplicationResourceBuilder : IResourceBuilder<T> where T : IResource
|  * Resource
|  * ApplicationBuilder
|___ProjectResourceBuilderExtensions
    |  * AddProject
    |___ResourceBuilderExtensions
    |   |  * WithEnvironment
    |___ProjectResourceExtensions
    |   |  *  GetProjectMetadata
    |___ResourceExtensions
    |   |  *  GetEndpoints

```
