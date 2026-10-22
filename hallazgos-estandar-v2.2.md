# Hallazgos sobre el Estándar de Codificación C# v2.2

**Fecha:** 22 de octubre de 2026
**Documento revisado:** `estandar v2.2/` (Estándar de Codificación C# para Desarrollo de Videojuegos, Versión 2.2)
**Contra qué se comparó:** el proyecto final `bastion_Final_Finalv2/bastion` y las revisiones de reglas hechas durante la construcción de la v2.2.

---

## Cómo se obtuvieron estos hallazgos

- Se leyeron los 112 archivos `.cs` del proyecto final: unas 5,160 líneas, de las cuales 402 son `Strings.Designer.cs`, que es código generado. También se leyó la configuración (`.editorconfig`, `Directory.Build.props`, `CodeMetricsConfig.txt` y los `.csproj`) y los encabezados de los scripts SQL.
- Cada construcción del código se comparó con cada regla de la v2.2 y se clasificó así:
  1. la regla no se puede cumplir tal como está redactada;
  2. la regla se incumple en el proyecto;
  3. la regla es ambigua;
  4. el estándar no cubre el tema.
- Lo que se podía medir se midió con herramientas en lugar de a ojo:
  - un script para el orden de los `using`;
  - la longitud de las líneas;
  - una búsqueda de `=>` en propiedades;
  - compilaciones de prueba con .NET 10 (SDK 10.0.400) para comprobar CA1502, IDE0029, IDE0030, IDE0270 y el error CS0428.
- Las rutas de código (`src/...`, `tests/...`, `database/...`) son relativas a la raíz del proyecto `bastion_Final_Finalv2/bastion`. Los números de regla son los de la v2.2.

---

## Estado actual del estándar v2.2 (cambios respecto a la v2.1)

| Cambio | Detalle |
|---|---|
| Título | Dice "Versión 2.2". Los autores son los mismos de la v2.1. |
| 2.11.4 (nueva) | "Cuerpo de expresión solo en catálogos de diccionario". Su ejemplo devuelve la clave cuando falta una entrada (`?? key`), igual que la biblioteca de localización de Microsoft (Microsoft, 2025d, 2026e) y que `ResourceManager.GetString` cuando no encuentra el recurso (Microsoft, 2025e). La alternativa de lanzar una excepción se descartó porque no se recomienda lanzarlas desde el `get` de una propiedad (Cwalina y Abrams, 2008c). |
| 9.4.2 (nueva) | "Valor alternativo con `??` cuando una expresión puede ser nula" (Microsoft, 2026b; Microsoft, s. f.). Complementa la prohibición de usar `!`, que no tiene efecto en tiempo de ejecución (Microsoft, 2026a). |
| 3.4.1 (eliminada) | Se quitó el límite de 120 caracteres por línea. "Uso de `var`" pasó de 3.5 a 3.4. |
| Referencias | Se agregó Microsoft (2026h) sobre el operador `??`. Se quitaron IDE0025 y "Retrieve resources in .NET apps", que ya no se citaban. |

---

## 1. Reglas que el código no puede cumplir tal como están redactadas

### 1.1 Regla 2.11.4: cuerpo de expresión solo en catálogos de diccionario

El proyecto tiene 15 propiedades con `=>` fuera de un catálogo. En tres casos la propiedad es obligatoria y no se puede convertir en método:

| Caso | Dónde | Por qué tiene que ser propiedad |
|---|---|---|
| Enlace de datos (MVVM) | `LoginViewModel.cs:84, 105, 108, 117` | La vista enlaza por nombre de propiedad: `SetBinding(nameof(IsVisible), nameof(LoginViewModel.IsPasswordHidden))` en `LoginView.cs:144`. |
| Implementa una interfaz | `FakeUnitOfWork.cs:18-22` | `IUnitOfWork` declara `IAccountRepository Accounts { get; }`. |
| `DbSet` de EF Core | `BastionDbContext.cs:16-20` | La alternativa habitual, `{ get; set; } = null!`, usa el `!` que prohíbe la 9.4.1. |

```csharp
// src/Bastion.Client/Screens/Login/LoginViewModel.cs:104
[DependsOn(nameof(IsBusy))]
public bool IsIdle => !IsBusy;
```

Además:

- La regla no aclara si "cuerpo de expresión en una propiedad" incluye los accesores `get =>` y `set =>`. Hay 37 en el proyecto.
- La regla no define qué es un "catálogo de diccionario". `SupportedLanguages.cs:9` guarda entradas de recursos en propiedades autoimplementadas:

  ```csharp
  public static SupportedLanguage Spanish { get; } = new SupportedLanguage("es-MX", Strings.LanguageNameSpanish);
  ```

  Aquí no hay error porque "Español (MX)" solo existe en el `.resx` neutro y no cambia con el idioma. Aun así, la regla no deja claro si esta clase cuenta como catálogo.

**Sugerencia:** agregar una excepción para las propiedades que exigen una interfaz, un enlace de datos o un framework, y definir "catálogo de diccionario".

### 1.2 Regla 5.2.1: grupo de método en lugar de lambda, imposible con árboles de expresión

```csharp
// src/Bastion.Server.Adapters/Persistence/Conversions/UpperSnakeCaseEnumConverter.cs:13
: base(value => ToCode(value), code => FromCode(code))
```

La regla pide escribir `base(ToCode, FromCode)`, pero el constructor de `ValueConverter` en EF Core recibe árboles de expresión (`Expression<Func<...>>`). Se comprobó con .NET 10 que eso no compila: `error CS0428: Cannot convert method group 'ToCode' to non-delegate type 'Expression<Func<int, string>>'`.

**Sugerencia:** agregar la excepción "salvo cuando el parámetro es un árbol de expresión".

### 1.3 Regla 9.4.1: "punto de entrada" sin definir y datos que llegan por red

El punto de entrada real del servidor no usa `ThrowIfNull`, sino que responde con un fallo de WCF, porque una `ArgumentNullException` no le sirve al cliente:

```csharp
// src/Bastion.Server.Host/Accounts/AccountService.cs:33
if (credentials is null || device is null)
{
    throw LoginTranslator.ToFault(new InvalidCredentialsException());
}
```

En cambio, clases internas como `SignedInPlayer.cs:9-11` sí repiten `ThrowIfNull`, aunque la regla dice que no se repita en métodos internos.

También hay `??` sobre propiedades declaradas como no anulables:

- `LoginTranslator.cs:19`: `device.DeviceFingerprint ?? string.Empty`, donde `DeviceFingerprint` es `string`.
- `AccountClient.cs:75-77` y `PasswordHasher.cs:23` hacen lo mismo.

La razón es que un DTO deserializado puede traer `null` aunque su tipo diga que no.

**Sugerencia:** definir qué es un "punto de entrada" (servicio de red, archivo, interfaz de usuario) y qué hacer con los datos deserializados.

### 1.4 Regla 9.2.1: conservar la excepción original, en la frontera de red

```csharp
// src/Bastion.Server.Host/Accounts/AccountService.cs:55
catch (DbUpdateException ex)
{
    _logger.Error($"The sign-in could not be saved. IpAddress={connection.IpAddress}", ex);
    throw new FaultException(ServerFailureReason);
}
```

La excepción original se pierde a propósito para no enviar detalles internos al cliente, y antes se registra en la bitácora. Es correcto por seguridad, pero la regla no prevé esta excepción.

---

## 2. Reglas que se incumplen en el proyecto

### 2.1 Regla 6.3: orden de las directivas `using` (15 archivos)

```csharp
// src/Bastion.Client/Controls/LanguagePicker.cs:1
using System;
using Bastion.Client.Localization;
using Bastion.Client.Theme;
using Gum;
```

Los usings del proyecto (`Bastion`) van antes que los de terceros (`Gum`). Pasa en los 11 archivos de `Controls` y `Screens` y en las 4 clases de prueba, donde van antes que `Xunit`.

**Por qué ocurre:** el orden de la 6.3 no es alfabético, y la herramienta "ordenar usings" del editor ordena alfabéticamente. Ninguna opción de `.editorconfig` puede exigir "terceros antes que proyecto", así que la regla solo se puede revisar a mano.

### 2.2 Regla 11.1: dónde se registra una excepción

- `AccountClient.cs:35-64` registra en `Warn` cada vez que **traduce** y relanza una excepción (6 `catch`).
- `LoginViewModel.cs:200-226` **maneja** las excepciones sin registrarlas (5 `catch`) y no tiene logger.

La regla dice exactamente lo contrario: el registro va donde la excepción se maneja finalmente, no en cada traducción intermedia.

**Sugerencia:** corregir el código o aceptar en la regla el registro en la capa de red.

---

## 3. Reglas ambiguas: el proyecto las cumple solo porque no son específicas

| Regla | Qué falta definir | Ejemplo en el proyecto |
|---|---|---|
| 2.10 Valores mágicos | Si 0, 1 y 2 cuentan como valores mágicos. Si aplica a infraestructura, interfaz, pruebas y expresiones regulares. Si se permiten literales dentro de una definición con nombre. | `AccountRules.cs:8-15` define las constantes 3 y 30, pero la expresión regular repite `{3,30}`. Otros casos: `Account.cs:93` `= 0`; `LoginService.cs:88` `Pending, 0`; `TextStyles.cs:5` tamaño de letra `34`. |
| 4.4 No repetir código | Solo menciona bloques de código, no datos ni constantes. | El valor 254 está en `AccountRules.cs:10`, `AccountConfiguration.cs:13` y `AccessLogConfiguration.cs:10`. Los valores 64 y 32 están en `PasswordHasher.cs:9-10` y en `AccountConfiguration.cs:14-15`. `ScreenMarginX = 24` aparece en 3 vistas. |
| 10.1 Inyección de dependencias | Qué es un "servicio externo". Si una clase concreta o abstracta del framework vale como "interfaz". | `LoginService.cs:14` obtiene el logger de forma estática. `AccountRepository.cs:12` recibe un `DbContext` concreto. `UnitOfWork.cs:19` crea repositorios con `new`. `AccountService.cs:42` pide el servicio al contenedor dentro del método. |
| DTO (2.11.3, 2.8.1 y 2.2.2) | Tres reglas dicen cosas distintas: un DTO lleva `get; set;`, un valor inmutable va en un `record`, y el ejemplo de 2.2.2 usa `readonly record struct`. | `SignedInPlayer`, `GeneratedToken`, `SignedInSession` y `LoginCredentials` son clases inmutables, no records. Los contratos de red usan `get; set;`. |
| 2.14.2 Sufijos | Si la lista de sufijos permitidos es cerrada o abierta. | Se usan Translator, Registration, Converter, Generator, Hasher, Store, Navigator, Initializer, Skin, Handler, Client, Channel, ViewModel, View y Configuration. `StateHandler.cs:3` es una clase, aunque la 2.14.1 reserva el sufijo EventHandler para delegados. |
| 2.2.1 Nombres de método con verbo | Los métodos de conversión `To…` y `From…`. | `LoginTranslator.cs:12-52` y `UpperSnakeCaseEnumConverter.cs:17, 34`. |
| 4.3.2 Rama `default` | Exige la rama, pero no dice qué debe hacer: lanzar una excepción o devolver un valor. | `LoginTranslator.cs:74` y `AccountClient.cs:90` devuelven `Banned` cuando el valor es desconocido. |
| 2.12 Eventos | La regla solo nombra el método que eleva el evento (`On…`). `Handle…` aparece solo en ejemplos. No prevé eventos que define una biblioteca externa sin `sender` ni `e`. | Manejadores llamados `On…` en `BastionGame.cs:75`, `LoginView.cs:248` y `PickerSkin.cs:82`. `LanguagePicker.cs:23` recibe un `Action` de Gum. |
| 3.4 Uso de `var` | Qué es un "tipo evidente": métodos de fábrica estáticos, arreglos de tipos integrados. | `Server.Host/Program.cs:33` usa `var certificate = ServerCertificate.Load(...)`. `LanguageService.cs:81` usa `string[] … = new string[...]` y la línea 64 usa `var … = new List<string>()`. |
| 2.2.2 Máximo de parámetros | Si cuenta a los constructores. Cómo cuenta `params`. Si los records posicionales están exentos del camelCase de la 2.1.1. | `LoginService.cs:20` tiene justo 3 parámetros. `Shapes.cs:35` usa `params` y `Skins.cs:40` lo llama con 4 argumentos. `AccessAttempt.cs:6` usa parámetros en PascalCase. |
| 12.3.1 Patrón AAA | Cómo separar preparación, ejecución y verificación cuando la prueba espera una excepción. | Hay tres estilos: `Assert.Throws` en una línea (`TestLoginService.cs:34`), `Record.Exception` (`TestLoginService.cs:78`) y un `Action` como paso de ejecución (`TestAccount.cs:39`). |
| 2.3.2 y 2.6 | La 2.3.2 dice `static readonly` (campo), pero el proyecto usa propiedades estáticas. Tampoco se define cómo nombrar campos `public static readonly`. | `Palette.cs:7`, `TextStyles.cs`, `ButtonLooks.cs` y `LockoutPolicy.cs:15`. |
| 11.1 Niveles de bitácora | En qué nivel van los eventos de seguridad, como un inicio de sesión fallido. | `LoginService.cs:124` registra todo intento en `Info`. |
| 4.6.1 Complejidad ciclomática | Solo menciona `CodeMetricsConfig.txt`. Faltan dos pasos: declarar el archivo como `AdditionalFiles` y activar CA1502, que viene desactivada en .NET 10. Además, la fecha de la referencia no coincide (ver nota 1). | Se comprobó con .NET 10 que sin esos dos pasos no aparece ningún aviso. El proyecto sí los configuró (`Directory.Build.props` y `.editorconfig`). |
| 6.1 Un tipo por archivo | No dice nada de tipos anidados, `struct` ni delegados. | No hay casos en el proyecto, pero la regla los deja abiertos. |
| 9.4.1 Referencias anulables | Su ejemplo solo muestra un valor de retorno anulable. No hay ejemplo de parámetro ni de propiedad anulable. | `PlayerProfile` en la 9.4.2 usa `string? nickname`, que la 9.4.1 cubre por su texto pero no por su ejemplo. |
| 9.4.2 Operador `??` | IDE0029 e IDE0030 solo detectan el ternario escrito con `!= null`, no con `is null` ni `is not null`, que es lo que obliga la 9.4.1 (Microsoft, 2023b). IDE0270 detecta un `if` que asigna, pero no un `if` con `return` anticipado. | Se comprobó con .NET 10. La regla depende de la revisión de código. |

1. **Nota sobre la fecha de la referencia de la 4.6.1:** el estándar cita "Microsoft (2023d, 22 de abril)", pero la página de CA1502 muestra hoy el 9 de noviembre de 2022 (Microsoft, 2022a).

---

## 4. Temas que el estándar no cubre

| # | Tema | Ejemplo en el proyecto |
|---|---|---|
| 1 | **Código generado** | `Strings.Designer.cs` incumple seis reglas (llaves, espacio de nombres, `using`, campos, documentación XML), pero se regenera solo. No hay una regla que lo exente. |
| 2 | Programación asíncrona | `LoginViewModel.cs:155` tiene `async Task LogIn()` sin sufijo `Async`. `:197` usa `Task.Run`. `LoginDialogView.cs:104` usa `async void`. |
| 3 | Liberación de recursos | `ServerCertificate.cs:15` guarda un `X509Certificate2` sin implementar `IDisposable`. `ServiceChannel.cs:68` llama a `Close()`, que puede lanzar, dentro de `Dispose`. |
| 4 | Manejo del tiempo | `AccountService.cs:39` usa `DateTime.UtcNow`. El dominio usa `DateTime` en vez de `DateTimeOffset`. Solo el script SQL exige UTC. |
| 5 | Cultura y comparación de texto | `ServerAddress.cs:34` usa `InvariantCulture` y `AccountRepository.cs:23` usa `ToLowerInvariant()`. El código lo hace bien, pero ninguna regla lo pide. |
| 6 | Diseño de excepciones (ver nota 1) | Una contraseña incorrecta se maneja como excepción (`LoginService.cs:37`). Las excepciones propias tienen un solo constructor. |
| 7 | Atributos | `[DoesNotReturn]`, `[ServiceBehavior]`, `[DependsOn]`, `[DataContract]`: no hay reglas de formato ni de uso. |
| 8 | Coincidencia de patrones | `Account.cs:76` usa `is DateTime lockedUntil`. `ButtonSkin.cs:97` usa un `switch` por tipo. |
| 9 | Enumeraciones | `InteractionFlags.cs` usa `[Flags]`, nombre en plural y `None = 0`. Algunos enums tienen líneas en blanco entre valores y otros no. |
| 10 | Propiedades con efectos | En `ModalLayer.cs:17` el `set` agrega o quita la capa. En `LoginViewModel.cs:37` el `set` cambia el idioma. La 2.11.2 solo habla del estado del dominio. |
| 11 | Cómo partir líneas | Al eliminar la 3.4.1 no queda ninguna regla. Hay estilos mezclados en `AccessLogConfiguration.cs:19-23` y en `CallerAddress.cs:13`. |
| 12 | Argumentos booleanos sin nombre | `LoginViewModel.cs:214` pasa `ShowDialog(..., true)`. `LanguageService.cs:94` pasa `(..., true, true)`. En otro lugar se escribe `ignoreCase: true`. |
| 13 | `catch` repetidos o con filtro `when` | `LanguagePreferenceStore.cs:36-45` tiene dos `catch` con el mismo cuerpo. |
| 14 | Tamaño de clases, métodos y archivos | No hay límite. `LoginViewModel` tiene 297 líneas y `MainMenuView` tiene 247 líneas con 35 constantes. |
| 15 | Pruebas más allá de nombres y AAA | Pruebas con parámetros (`[Theory]`, `[InlineData]`), dobles de prueba llamados `Fake*`, literales en las pruebas y preparación en el constructor. |
| 16 | Convenciones SQL | Tablas en singular y PascalCase, prefijos `PK_`, `FK_`, `UQ_`, `CK_`. Solo están en un comentario de `02_create_tables.sql`. |
| 17 | Seguridad | Hash de contraseñas, comparación en tiempo constante, verificación señuelo y manejo de secretos. La v2.0 tenía reglas sobre esto y la v2.1 las quitó. |
| 18 | Colecciones expuestas | `SupportedLanguages.cs:15` expone un arreglo como `IReadOnlyList`, que se puede convertir de vuelta y modificar. |

1. **Nota sobre el diseño de excepciones:** el estándar tampoco define el formato de los mensajes de excepción. Además, cliente y servidor tienen tipos con el mismo nombre: `AccountRestrictedException`, `InvalidCredentialsException` y `AccountRestriction`.

**Otro hallazgo de configuración:** el `.editorconfig` del proyecto sigue con la numeración de la v2.1 (`3.4.1`, `3.5.1`). Además, su `max_line_length = 120` nunca se aplicó al compilar: el proyecto tiene dos líneas de más de 120 caracteres (`AccountClient.cs:14` con 131 y `TestLoginService.cs:79` con 121).

---

## 5. Propuestas de reglas investigadas, pendientes de decisión

Ninguna de estas reglas se agregó al estándar.

| # | Regla propuesta | Dónde iría | Conflictos detectados | Fuentes |
|---|---|---|---|---|
| 1 | Sufijo `Async` y `await` en toda la cadena; prohibido `async void` fuera de manejadores y prohibido bloquear con `.Result` o `.Wait()` | 2.2.4 | No está claro si el `CancellationToken` cuenta para el límite de 3 parámetros de la 2.2.2. | Cleary (2013); Microsoft (2020c, 2021, 2024, 2026f); Okur et al. (2014) |
| 2 | Declaración `using` para todo objeto `IDisposable`; la clase dueña de un recurso lo libera | 9.5 | Una dependencia recibida por constructor (sección 10) no se libera, porque la clase no es su dueña. | Cwalina y Abrams (2008a); Ghanavati et al. (2020); Microsoft (2017, 2020a, 2022b, 2025a, 2025c, 2026g) |
| 3 | Cultura invariante para datos que se transmiten o guardan; comparación ordinal de cadenas | 1.3.2 | Las bitácoras (11.3) formatean con la cultura actual. La comparación ordinal distingue acentos en los apodos. | Eghbali y Pradel (2020); Microsoft (2020b, 2023a, 2026c) |
| 4 | Obtener el tiempo de un `TimeProvider` inyectado | 10.2 | La 10.1 dice "interfaz", pero `TimeProvider` es una clase abstracta. | Microsoft (2025b, 2025f, 2026h); Nystrom (2014); Tiwari et al. (2025) |
| 5 | Exponer colecciones como solo lectura con `AsReadOnly()` | 2.11.5 | CA1002 marcaría 5 ejemplos del estándar que reciben `List<Player>` por constructor. | Coblenz et al. (2017); Cwalina y Abrams (2008b); Microsoft (2016, 2026d) |
| 6 | Evitar parámetros booleanos que cambian el comportamiento de un método | Sin definir | Falta investigarla. | Martin (2008), que ya está en el estándar |

**Hechos comprobados con .NET 10 para estas propuestas:**

- Con culturas `es-ES`, `de-DE` y `tr-TR`, `double.Parse("1.5")` falla.
- En `es-MX` la fecha se escribe con el día primero (`04/03/2026`).
- `"login".ToUpper()` en cultura turca da `LOGİN`.
- Una lista expuesta como `IReadOnlyList` se puede convertir de vuelta y modificar; con `AsReadOnly()` no.
- Los ejemplos de estas propuestas son bocetos y no se compilaron. Solo se compilaron los ejemplos de las reglas que ya están en el estándar (2.11.4 y 9.4.2).

---

## 6. Prioridades sugeridas

1. **2.11.4:** agregar la excepción para enlace de datos, interfaces y frameworks, y definir qué es un "catálogo de diccionario".
2. **5.2.1 y 9.4.1:** corregir su redacción, porque hoy no se pueden cumplir en EF Core ni en un servicio WCF.
3. **6.3 y 11.1:** decidir si se corrige el código o se ajustan las reglas.
4. **4.6.1:** completar los pasos de configuración y corregir la fecha de la referencia.
5. **Código generado:** agregar una regla que lo exente.
6. **Propuestas 1 a 6 (sección 5):** decidir cuáles se adoptan.

---

## Referencias (APA 7)

> Las letras de año corresponden a esta lista. Si una fuente pasa al estándar, sus letras se ajustan a la lista de referencias del estándar.

- Cleary, S. (2013, marzo). Async/await: Best practices in asynchronous programming. *MSDN Magazine, 28*(3). https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming
- Coblenz, M., Nelson, W., Aldrich, J., Myers, B., & Sunshine, J. (2017). Glacier: Transitive class immutability for Java. En *2017 IEEE/ACM 39th International Conference on Software Engineering (ICSE)* (pp. 496–506). IEEE. https://doi.org/10.1109/ICSE.2017.52
- Cwalina, K., & Abrams, B. (2008a, 22 de octubre). *Dispose pattern* [Reimpresión de *Framework design guidelines* (2.ª ed.), Addison-Wesley Professional]. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/dispose-pattern
- Cwalina, K., & Abrams, B. (2008b, 22 de octubre). *Guidelines for collections* [Reimpresión de *Framework design guidelines* (2.ª ed.), Addison-Wesley Professional]. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/guidelines-for-collections
- Cwalina, K., & Abrams, B. (2008c, 22 de octubre). *Property design* [Reimpresión de *Framework design guidelines* (2.ª ed.), Addison-Wesley Professional]. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/property
- Eghbali, A., & Pradel, M. (2020). No strings attached: An empirical study of string-related software bugs. En *Proceedings of the 35th IEEE/ACM International Conference on Automated Software Engineering* (pp. 956–967). Association for Computing Machinery. https://doi.org/10.1145/3324884.3416576
- Ghanavati, M., Costa, D. E., Seboek, J., Lo, D., & Andrzejak, A. (2020). Memory and resource leak defects and their repairs in Java projects. *Empirical Software Engineering, 25*(1), 678–718. https://doi.org/10.1007/s10664-019-09731-8
- Martin, R. C. (2008). *Clean code: A handbook of agile software craftsmanship*. Prentice Hall.
- Microsoft. (2016, 4 de noviembre). *CA1002: Do not expose generic lists*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1002
- Microsoft. (2017, 30 de marzo). *SQL Server connection pooling (ADO.NET)*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/sql-server-connection-pooling
- Microsoft. (2020a, 18 de diciembre). *CA1001: Types that own disposable fields should be disposable*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1001
- Microsoft. (2020b, 26 de agosto). *CA1310: Specify StringComparison for correctness*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1310
- Microsoft. (2020c, 18 de junio). *CA2016: Forward the CancellationToken parameter to methods that take one*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2016
- Microsoft. (2021, 27 de agosto). *CA1849: Call async methods when in an async method*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1849
- Microsoft. (2022a, 9 de noviembre). *CA1502: Avoid excessive complexity*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1502
- Microsoft. (2022b, 18 de marzo). *IDE0063: Use simple 'using' statement*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0063
- Microsoft. (2023a, 17 de julio). *CA1305: Specify IFormatProvider*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1305
- Microsoft. (2023b, 25 de julio). *Null check can be simplified (IDE0029, IDE0030, and IDE0270)*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0029-ide0030-ide0270
- Microsoft. (2024, 22 de noviembre). *Async return types (C#)*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/async-return-types
- Microsoft. (2025a, 1 de febrero). *CA2000: Dispose objects before losing scope*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2000
- Microsoft. (2025b, 1 de julio). *DateTime.Now property*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/api/system.datetime.now
- Microsoft. (2025c, 22 de octubre). *Implement a Dispose method*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose
- Microsoft. (2025d, 1 de julio). *LocalizedString.ResourceNotFound property*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.localization.localizedstring.resourcenotfound
- Microsoft. (2025e, 1 de julio). *ResourceManager.GetString method*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/api/system.resources.resourcemanager.getstring
- Microsoft. (2025f, 1 de julio). *Stopwatch class*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.stopwatch
- Microsoft. (2026a, 20 de enero). *! (null-forgiving) operator (C# reference)*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-forgiving
- Microsoft. (2026b, 20 de enero). *?? and ??= operators: Null-coalescing operators (C# reference)*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-coalescing-operator
- Microsoft. (2026c, 23 de febrero). *Best practices for comparing strings in .NET*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/standard/base-types/best-practices-strings
- Microsoft. (2026d, 30 de marzo). *CA2227: Collection properties should be read only*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2227
- Microsoft. (2026e, 22 de septiembre). *Make an ASP.NET Core app's content localizable*. Microsoft Learn. https://learn.microsoft.com/en-us/aspnet/core/fundamentals/localization/make-content-localizable
- Microsoft. (2026f, 17 de abril). *Task-based asynchronous pattern (TAP): Introduction and overview*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap
- Microsoft. (2026g, 16 de enero). *using statement: Ensure the correct use of disposable objects (C# reference)*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using
- Microsoft. (2026h, 16 de enero). *What is the TimeProvider class*. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/standard/datetime/timeprovider-overview
- Microsoft. (s. f.). *Expressions: C# language specification* (§12.18, The null coalescing operator) [Texto de trabajo del estándar ECMA-334]. Microsoft Learn. https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions
- Nystrom, R. (2014). Game loop. En *Game programming patterns*. Genever Benning. https://gameprogrammingpatterns.com/game-loop.html
- Okur, S., Hartveld, D. L., Dig, D., & van Deursen, A. (2014). A study and toolkit for asynchronous programming in C#. En *Proceedings of the 36th International Conference on Software Engineering* (pp. 1117–1127). Association for Computing Machinery. https://doi.org/10.1145/2568225.2568309
- Tiwari, S., Chen, S., Joukov, A., Vandervelde, P., Li, A., & Padhye, R. (2025). It's about time: An empirical study of date and time bugs in open-source Python software. En *2025 IEEE/ACM 22nd International Conference on Mining Software Repositories (MSR)* (pp. 39–51). IEEE. https://doi.org/10.1109/MSR66628.2025.00020
