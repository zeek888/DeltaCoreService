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
Una vez que configuramos la Web Api, creamos el archivo JSON de configuración, donde vamos a ir agregando los servicios que vamos a exponer:

![ServiceStructure](https://res.cloudinary.com/jcerino/image/upload/v1619551982/ServiceFolder_pvnyri.png)
Como vemos en la imagen, el archivo JSON debe llamarse `ServicesDG.json`, y debe estar alojado en la carpeta `/Configuration`.

En mi base de datos, tengo la siguiente tabla:
![ServiceStructure](https://res.cloudinary.com/jcerino/image/upload/v1619552495/TablaTicket_tuf4aw.png)
Vamos a agregar un nuevo servicio para obtener todos los registros de nuestra tabla:

```json
{
  "services": [
    {
      "name": "tickets",
      "method": "GET",
      "info": {
        "table": "dbo.Ticket",
        "operation": "SELECT",
        "columns": [
          {
            "name": "TicketId"
          },
          {
            "name": "Clave"
          },
          {
            "name": "Folio"
          }
        ]
      }
    }
  ]
}

```

Como vemos, solo devolverá las columnas `TicketId`, `Clave` y `Folio`.

4.- Consultar el servicio
--------
Para consultar el servicio, hacemos un request a nuestra api al endpoint `/api/service/[nombreServicio]` para nuestro ejemplo es : `/api/service/tickets` y listo, se obtiene los resultados de la consulta a la tabla.

NOTA: Es importante mencionar que para la búsqueda del servicio se toma en cuenta el nombre y el metodo, en este caso es un GET, pero si tratamos de ejecutar un metodo POST al servicio, nos devolverá error, ya que no existe un metodo POST para el servicio, solo GET.


Para mas información de la estructura del archivo JSON consulte la WIKI.
