# DeltaCoreService
El componente DeltaCoreService, ayuda a exponer servicio de acceso a datos de una forma rapida y sencilla, podemos exponer tablas o procedimientos almacenados con filtros, paginación y campos a devolver.

¿Que es?
--------
En pocas palabras, el componente permite convertir una Web Api, en un servicio de datos, esto, sin definir modelos, solo configurando los servicios en un archivo JSON, el cual, al modificarse, se reflejan los cambios en la aplicación, evitando recompilar nuestro proyecto.


1.- Instalar el nuget
--------
Primero debemos instalar el paquete [Nuget del componente](https://www.nuget.org/packages/DeltaGrid.ServicesApi.Core)  en nuestra Web API

2.- Configurar la Web Api
--------
Una vez que instalamos el paquete, debemos configurar nuestra web api para que pueda soportar la ejecución de los servicios.

En el `Startup`, sección `ConfigureServices`, debemos registrar los servicios necesarios ejecutando el metodo de extensión 'RegisterCoreServices'

```csharp
public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllers().AddNewtonsoftJson(options => options.SerializerSettings.ContractResolver = new DefaultContractResolver());
            services.RegisterCoreServices();
        }
```

En la sección de `Configure`, debemos configurar algunas operaciones necesarias:

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
        ....
        DeltaGrid.ServicesApi.Core.Extensions.ConfiguracionApp.Configurar(el =>
            {
                el.CadenaConexion = (str) => Configuration.GetConnectionString(str);   //Obtener la cadena de conexión a la base de datos
                //Resolver la inyección de dependencia
                el.DgResolverType = (tipo, requerido) => requerido ? app.ApplicationServices.GetRequiredService(tipo) : app.ApplicationServices.GetService(tipo); 
            });
        ....
        }
```
Agregar la cadena de conexión en el appsettings:


```json
  "ConnectionStrings": {
    "ConexionLocal": "Persist Security Info=True;user id=admin;password=test;Initial Catalog=MyBd;Data Source=localhost;MultipleActiveResultSets=True;",
  }
```
Es importante mencionar que por defecto, se usa la cadena de conexión `ConexionLocal` asi que es importante dejar el mismo nombre a la cadena de conexión.


3.- Crear y configurar el archivo JSON de configuración de servicios
--------
Una vez que configuramos la Web Api, creamos el archivo JSON de configuración, donde vamos a ir agregando los servicios que vamos a exponer.
