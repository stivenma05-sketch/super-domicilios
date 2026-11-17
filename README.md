# 🛒 Super Domicilios — Mini proyecto 2

Aplicación de **domicilios para un supermercado**: los clientes arman su carrito y piden desde la web; los repartidores y el administrador gestionan los pedidos desde la web o desde una aplicación de escritorio. Todo comparte la misma lógica de negocio y la misma base de datos.

> Asignatura: Lenguajes de Programación · Mini proyecto 2 · 17 de noviembre de 2026
> Integrantes: _(completar)_ · Grupo / semestre / identificación: _(completar)_

## Arquitectura

```
Domicilios.sln
├── src/
│   ├── Domicilios.Core      Biblioteca de clases base: entidades, estructuras de datos,
│   │                        reglas de negocio y contratos (interfaces). Sin dependencias de EF ni de UI.
│   ├── Domicilios.Data      Entity Framework Core (SQLite) + ASP.NET Core Identity,
│   │                        repositorios, unidad de trabajo y datos de ejemplo.
│   ├── Domicilios.Web       ASP.NET Core MVC (Bootstrap 5, responsive) con login y 3 roles.
│   └── Domicilios.Desktop   Aplicación de escritorio WinForms (Admin y Repartidor).
├── tests/Domicilios.Tests   Pruebas unitarias xUnit de la lógica de negocio.
└── docs/                    Estructuras de datos, guion de la exposición y guía de GitHub.
```

Dependencias: `Web` y `Desktop` → `Data` → `Core`. La lógica solo conoce interfaces (`IProductoRepositorio`, `IPedidoRepositorio`, `IUnidadDeTrabajo`), por eso se puede probar sin base de datos.

## Cómo ejecutarlo

Requisitos: **.NET SDK 8.0** (la app de escritorio requiere Windows).

```bash
dotnet restore
dotnet test                                   # pruebas unitarias
dotnet run --project src/Domicilios.Web       # web  -> https://localhost:7080
dotnet run --project src/Domicilios.Desktop   # escritorio (Windows)
```

La primera ejecución crea la base de datos SQLite (`%LocalAppData%/Domicilios/domicilios.db`), los roles y los datos de ejemplo. Web y escritorio usan **el mismo archivo**, así un pedido hecho en la web aparece de inmediato en el escritorio. Se puede cambiar la ruta con la variable de entorno `DOMICILIOS_DB`.

### Usuarios de prueba

| Rol        | Correo                      | Contraseña   |
|------------|-----------------------------|--------------|
| Admin      | admin@domicilios.com        | Admin123     |
| Repartidor | repartidor@domicilios.com   | Reparto123   |
| Cliente    | cliente@domicilios.com      | Cliente123   |

Cupones: `BIENVENIDO10` (10 %), `FRUVER5` (5 %), `MERCADO20` ($20.000 en compras desde $150.000).

## Roles y permisos (ASP.NET Core Identity)

| Rol | Web | Escritorio |
|---|---|---|
| **Cliente** | Catálogo, carrito, crear pedido, ver y cancelar (si está pendiente) sus pedidos | — |
| **Repartidor** | Ver pedidos listos, tomarlos, marcarlos como entregados | Pedidos |
| **Admin** | Todo lo anterior + productos/inventario, reportes, cambiar estados, deshacer, comparativa de estructuras | Pedidos, productos, reportes, estructuras |

Quien se registra desde la web siempre queda como **Cliente**.

## Flujo del pedido

`Pendiente → En preparación → En camino → Entregado` (y `Cancelado` desde Pendiente / En preparación). Las transiciones válidas viven en `PedidoService` y al cancelar se devuelve el inventario.

## Requisitos del taller y dónde se cumplen

| Requisito | Dónde |
|---|---|
| Biblioteca de clase base con lógica de dominio y negocio | `src/Domicilios.Core` |
| Patrón MVC en la web | `src/Domicilios.Web` (Controllers / Views / Models) |
| Aplicación de escritorio | `src/Domicilios.Desktop` |
| Almacenamiento con Entity Framework Core | `src/Domicilios.Data` (`AppDbContext`, repositorios) |
| Estructuras de datos justificadas con ejemplos comparativos | [`docs/ESTRUCTURAS_DE_DATOS.md`](docs/ESTRUCTURAS_DE_DATOS.md) + página **Estructuras** (Admin) |
| ASP.NET Core Identity con ≥ 2 roles | `AccountController`, `DbInicializador` (3 roles) |
| Diseño responsive (tablet / smartphone) | Bootstrap 5: navbar colapsable, grillas `row-cols-*`, tablas `table-responsive` |
| Repositorio en GitHub | [`docs/GUIA_GITHUB.md`](docs/GUIA_GITHUB.md) + CI en `.github/workflows` |
| Exposición de 15 min | [`docs/GUION_PRESENTACION.md`](docs/GUION_PRESENTACION.md) |

## Características de C# y .NET usadas

| Característica | Ejemplo |
|---|---|
| Clases, herencia, clases abstractas, polimorfismo | `Descuento` → `DescuentoPorcentaje`, `DescuentoMontoFijo`, `SinDescuento` |
| Interfaces e inyección de dependencias | `IProductoRepositorio`, `IPedidoRepositorio`, DI en `Program.cs` |
| Genéricos | `Resultado<T>`, `IComparable<Pedido>`, colecciones genéricas |
| Colecciones | `Dictionary`, `HashSet`, `Queue`, `Stack`, `SortedDictionary`, `List` |
| Indexadores, `IEnumerable<T>` con `yield` | `CarritoCompras` |
| Records, record struct, sobrecarga de operadores | `ResumenPedido`, `Dinero` (`+`, `-`, `*`, conversión implícita) |
| Delegados y eventos | `PedidoService.EstadoCambiado` (`EventHandler<CambioEstadoEventArgs>`), `Func<>` en el escritorio |
| LINQ | `ReporteService` (`GroupBy`, `SelectMany`, `OrderBy`, `Sum`…) |
| async / await | Repositorios, servicios y controladores |
| Métodos de extensión | `ComoPesos()`, `Descripcion()`, `ClaseColor()` |
| Pattern matching y switch expressions | `Extensiones`, `PedidoService`, vistas |
| Tipos anulables (`?`) y `Nullable` habilitado | Todo el proyecto |
| Atributos / validación con DataAnnotations | Entidades y ViewModels |
| Entity Framework Core (Code First, relaciones, conversiones de enum) | `AppDbContext` |

## Migraciones de EF Core (opcional)

Por defecto el esquema se crea con `EnsureCreated`. Si el profesor pide migraciones:

```bash
dotnet tool install --global dotnet-ef
dotnet ef migrations add Inicial -p src/Domicilios.Data -s src/Domicilios.Web
```

Desde ese momento `DbInicializador` aplica las migraciones automáticamente (si ya existía una base de datos creada sin migraciones, bórrala primero).
