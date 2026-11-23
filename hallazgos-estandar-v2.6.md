# Hallazgos de la revisión de citas — Estándar v2.6

Revisión del 22 de noviembre de 2026. Base: v2.5 (`0a65ccd`). Rama `estandar-v2.6`.

Cada prohibición que se apoya en una fuente de documentación recibió, si la fuente la respalda, un bloque `quote` con texto literal en formato APA 7. La frase de entrada parafrasea el contexto. Este archivo reúne dos cosas: de dónde salen los pasajes largos y las reglas que no recibieron cita.

## 1. Pasajes largos pegados por el equipo (12)

Los copió el equipo desde las páginas y se compararon palabra por palabra con las versiones descargadas el 22 de noviembre de 2026. Todos coinciden y ya están en el `.tex`, sin iconos (✔️/❌), con el formato de código del original en `\texttt{…}` y las comillas como ` ``…'' `.

| Prop. | Regla | Archivo | Página | Pasaje |
|---|---|---|---|---|
| 3 | 2.3.1 | `secciones/02-reglas-nombrado/03-constantes.tex` | https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/identifier-names | Sección Naming conventions, elemento 12: la oración completa sobre PascalCase en las constantes |
| 5 | 2.7.2 | `secciones/02-reglas-nombrado/07-variables-no-utilizadas.tex` | https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/patterns/discards | Sección Mark unused lambda parameters, el párrafo que va después del ejemplo, completo (dos oraciones). El `_` se escribe `\texttt{\_}` |
| 8 | 2.12.2 | `secciones/02-reglas-nombrado/12-eventos.tex` | https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/event | Párrafo 4 después de la nota de Pearson: la guía de `System.EventHandler<TEventArgs>`. El tipo se escribe en `\texttt{…}` |
| 10 | 2.15.1 | `secciones/02-reglas-nombrado/15-enumeraciones.tex` | https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/names-of-classes-structs-and-interfaces | Sección Naming Enumerations, las cinco guías, una por párrafo con una línea en blanco entre ellas |
| 13 | 3.4.1 | `secciones/03-estilo-codigo/04-uso-var.tex` | https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/style-rules/ide0007-ide0008 | Sección `csharp_style_var_elsewhere`: el texto del valor `false` |
| 19 | 5.1.2 | `secciones/05-programacion-funcional/01-expresiones-lambda.tex` | https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/how-to-subscribe-to-and-unsubscribe-from-events | Sección To subscribe to events by using an anonymous function, el párrafo que va después del código: sus tres primeras oraciones |
| 26 | 9.3.1 | `secciones/09-manejo-excepciones/03-restricciones.tex` | https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca2201 | Rule description, apartado Don't throw general exceptions: sus dos párrafos, sin el título |
| 30 | 9.5.1 | `secciones/09-manejo-excepciones/05-liberacion-recursos.tex` | https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose | Sección Cascade dispose calls, recuadro Tip: la primera oración |
| 32 | 9.6.2 | `secciones/09-manejo-excepciones/06-excepciones-propias.tex` | https://learn.microsoft.com/en-us/dotnet/framework/wcf/specifying-and-handling-faults-in-contracts-and-services | Sección Error Handling Overview, párrafo 4: la primera oración |
| Rev. | 2.2.4 | `secciones/02-reglas-nombrado/02-metodos.tex` | https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/quality-rules/ca1021 | Sección How to fix violations, párrafo 1, sus dos oraciones |
| Rev. | 9.5.1 | `secciones/09-manejo-excepciones/05-liberacion-recursos.tex` | https://learn.microsoft.com/en-us/dotnet/standard/garbage-collection/implementing-dispose | Sección Cascade dispose calls, párrafo 1: la primera oración |
| Rev. | 9.7.1 | `secciones/09-manejo-excepciones/07-manejadores-async-void.tex` | https://learn.microsoft.com/en-us/archive/msdn-magazine/2013/march/async-await-best-practices-in-asynchronous-programming | Sección Avoid Async Void, párrafo 3: la tercera oración |

Si una página cambia, el pasaje puede dejar de coincidir. Antes de publicar otra versión conviene comprobarlo de nuevo.

## 2. La fuente citada contradice la regla

- **2.6.1 y 2.6.2:** prohíben el prefijo `s_`. La guía de nombres (2025c) dice que los campos estáticos privados o internos empiezan con `s_`.
- **7.1.1:** prohíbe documentar los tipos que no son servicios. La página de documentación XML (2026c) dice que el compilador avisa con CS1591 en todo miembro visible.
- **9.6.1:** «solo los constructores que reciben datos». CA1032 y la guía de buenas prácticas (2025a) piden los tres constructores estándar.
- **9.4.4:** prohíbe `?? throw`. La página del operador `??` (2026h) lo recomienda para validar argumentos.
- **4.5.1:** prohíbe el ternario anidado. La página del operador (2026e) lo muestra como válido.
- **5.1.2 (asignación sobre el estado capturado):** la página de lambdas (2026g) muestra una lambda que asigna una variable capturada.
- **3.3.1 (inicializadores):** el ejemplo de la sección *new operator* de 2025e abre la llave en la misma línea que `new`.
- **3.4.1, casos 3 y 4:** 2025e dice que no se deduzca el tipo del nombre del método. IDE0007 (2022c), que es lo que cita la regla, sí los acepta.

## 3. La fuente citada no respalda la prohibición (decisión del equipo)

- **1.3.1:** la guía de nombres (2025c) no habla del idioma.
- **2.7.1:** 2026a solo dice que el nombre de la variable se puede omitir. El aviso CS0168 sería el respaldo real.
- **2.7.2:** la frase «evita que el analizador de estilo señale un parámetro sin usar» ya no está en la página. IDE0060 la respaldaría.
- **2.11.1, 2.11.3 y 2.11.4:** la página de propiedades (2025f) describe las opciones sin prohibir ninguna.
- **2.14.3:** la guía dice CONSIDER y la regla lo vuelve obligatorio.
- **3.4.2:** que `var` no pueda deducir el tipo no está en 2026l; es el error CS9176. Con C# 15, `with(...)` volverá obsoleta la excepción de capacidad y comparador. Según IDE0028, `[]` en lugar de `new T[] { }` cambia un poco la semántica.
- **3.5.1:** las convenciones 2 a 4 y la opción `dotnet_style_operator_placement_when_wrapping` no aparecen en 2025e.
- **3.6.1:** 2025e no habla de atributos.
- **4.6.1:** CA1502 usa 25 por defecto; el 10 lo fija el equipo.
- **5.1.1:** Martin (2008) es anterior a las lambdas de Java. 2026g dice que una lambda de bloque suele tener dos o tres instrucciones.
- **5.1.2 (lambda dentro de lambda):** no tiene fuente propia.
- **5.2.1:** la página de delegados (2024) no lo dice. IDE0200 lo respaldaría.
- **5.3.1:** la página de operadores de consulta (2025h) no prohíbe encadenar `ToList` ni escribir la cadena en una línea.
- **6.1:** CA1034 no detecta los tipos anidados `internal` ni `protected`, y además admite los `protected` para las subclases. El texto de la v2.6 ya lo dice y declara que el estándar se aparta de CA1034.
- **7.1.2:** `inheritdoc` está en la página *Recommended XML tags*, no en 2026c. Las reglas 7.2.1 y 7.2.2 tampoco tienen respaldo en 2026c.
- **8.2.1:** la prohibición del código deshabilitado no aparece en 2025e.
- **9.4.2:** la prohibición de `?:` o `if` no está en 2026h. La de usar `??` sobre una propiedad no anulable no tiene fuente.
- **9.6.2:** «`void`, nunca `bool`» y «el rechazo no viaja como código» son decisiones del equipo.
- **10.3.1:** la guía de inyección de dependencias (2026p) llama variante del localizador a inyectar una fábrica, pero recomienda `IServiceScopeFactory.CreateScope`, que es lo que usa la Excepción.
- **11.1.1** (excepción según el nivel), **11.2.1** y **11.3.1:** las páginas de Apache no lo dicen; McConnell es un libro.
- **2.14.4 y 9.7.2:** ya declaran que se apartan de .NET.

## 4. Sin fuente

- 2.1.6
- 2.3.2: CA1802 respaldaría la parte de `const`.
- 2.6.2
- 2.14.2
- 7.2.2
- 8.1.2
- 8.2.2
- 8.3.1

## 5. Citan libros que no se pudieron abrir

Martin, McConnell, Cwalina y Abrams, Hunt y Thomas, Fowler, van Deursen y Seemann, y Osherove. Las reglas afectadas son 2.1.2 a 2.1.5, 2.2.2, 2.2.3, 2.4.1, 2.5.x, 2.6.3, 2.9.1, 2.10.1, 4.1.1, 4.2.1, 4.3.2, 4.4.x, 6.1 (un tipo por archivo), 6.2, 8.1.1, 9.1.1, 9.4.3, 10.2.1 y 12.x.

Dos datos que vienen de memoria y hay que comprobar en los libros:
- McConnell propone limitar los parámetros a unos siete, no a tres (2.2.2).
- Osherove nombra las clases de prueba con el sufijo `Tests`, no con el prefijo `Test` (12.1.1).

## 6. Referencias por revisar

- **IDE0065 (2022b):** la referencia dice 25 de junio de 2022. La página dice 30 de septiembre de 2020, actualizada el 29 de septiembre de 2023.
- **Event design (2023b):** actualizada el 30 de marzo de 2026.
- **Names of classes (2023c):** actualizada el 29 de mayo de 2025.
- **CA1032 (2016), CA2225 (2019), CA1062 (2020) y CA1502 (2022d):** las páginas tienen ahora fecha del 13 de noviembre de 2026.
- **IDE0130 e IDE0161 (s. f.):** las páginas sí tienen fecha.
- **Logging in C# (2025d):** está en la lista, pero no se cita en ningún lugar.
- **Letras sin usar:** 2023d y 2025b.
- **Fecha de consulta:** APA 7 la pide para las páginas que cambian con el tiempo («Recuperado el 22 de noviembre de 2026, de …»). Es opcional y no se aplicó.

## 7. Revisión final de las citas

Una segunda lectura independiente revisó si cada cita establece o justifica la prohibición de su regla. Antes de corregir, se comprobaron en las páginas los puntos principales.

**Corregido en esta versión:**
- **2.1.1:** se añadió la paráfrasis de los nombres significativos, para respaldar el caso del contador con significado en el dominio.
- **2.2.4:** el bloque es ahora la corrección de CA1021. La v2.6 declara que CA1021 solo revisa la API pública y exime el patrón `Try`.
- **2.7.2:** el bloque pasó a la Descripción, que es la parte que respalda.
- **3.4.2:** el bloque viene ahora de las convenciones de código (2025e), que piden expresiones de colección para inicializar todos los tipos de colección.
- **3.7.1:** el bloque viene ahora de 2025e, que pide inicializadores de objeto junto a un contraejemplo.
- **6.1:** se declara que el estándar se aparta de CA1034 en los anidados `protected`.
- **9.3.1:** el bloque viene ahora de 2025e, que dice que no se capturen excepciones generales. CA1031 acepta como corrección relanzar la excepción, así que el código incorrecto de la regla solo se detecta en la revisión de código; la v2.6 ya lo dice.
- **9.5.1:** se añadió el bloque que establece que la clase dueña implementa `IDisposable`. Se conservó la cursiva del original.
- **9.5.2:** se añadió el bloque que explica por qué falla un `Close` directo.
- **9.7.1:** se añadió el bloque que justifica el `try`. Se quitó «el proceso termina», que la fuente no dice.
- **Formato APA:** se corrigieron los localizadores de CWE-209 (por número de mitigación) y las citas de varias obras del mismo autor, que ahora van separadas por coma.

**Sigue siendo parcial (no se cambió):**
- **2.7.2:** según la fuente, un `_` único es un nombre de parámetro común, no un descarte. Eso afecta al ejemplo correcto `_ => …`.
- **3.4.1:** la cita describe la opción de configuración, no una razón.
- **9.6.2:** `void`, nunca `bool`, y la prohibición de códigos en la respuesta son decisiones del equipo.
- **9.4.4:** CA1854 corrige con `TryGetValue`; la regla pide `GetValueOrDefault` para los valores de tipo referencia.
- **9.4.1:** la parte de `is null` frente a `== null` no tiene cita.
- **APA:** el bloque se usa también para las citas de menos de 40 palabras. Es una decisión del equipo.
