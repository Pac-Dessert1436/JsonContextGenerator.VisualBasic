# JsonContextGenerator.VisualBasic

A source generator that utilizes `JsonSerializerContext` in Visual Basic .NET, **filling a major gap in the toolchain of the language.** _The `JsonSerializerContext` source generation has been a feature officially supported by the .NET SDK only for C#._

It automatically generates strongly-typed `JsonSerializerContext` classes for any VB class or struct annotated with `GenerateJsonContextAttribute`, enabling high-performance, source-generated JSON serialization with `System.Text.Json`.

## Requirements

- [.NET 6.0](https://dotnet.microsoft.com/download/dotnet/6.0) or higher
- Visual Basic project

## Why This Project Exists

The .NET SDK's native source generators for JSON (`JsonSerializerContext`) are **C# only**. VB developers are left without an easy way to use source-generated serialization - until now.

This generator brings **full JSON context support** to VB projects, with:
- Attribute-driven configuration
- Proper `JsonSerializerOptions` setup (Default / Strict / Web modes)
- Compile-time type-safe property mapping
- Runtime-friendly `JsonTypeInfo` resolution

## Features

- Works with VB **classes** (not structs, because structs are immutable)
- Generates a dedicated `{TypeName}JsonContext` class
- Supports `JsonContextOption`: `Default`, `Strict`, `Web`
- Handles public properties automatically
- Provides primitive type fallback in `GetTypeInfo`
- Produces clean, readable, auto‑generated code
- **Fully incremental for fast builds**

## Installation

Add this generator to your VB.NET project via NuGet:

```bash
dotnet add package JsonContextGenerator.VisualBasic
```

Or reference the project directly:

```xml
<ProjectReference Include="..\JsonContextGenerator.VisualBasic\JsonContextGenerator.VisualBasic.vbproj"
    OutputItemType="Analyzer" ReferenceOutputAssembly="false" />
```

## Usage

### 1. Annotate your type

```vb
Imports System.Text.Json.Serialization

<GenerateJsonContext(JsonContextOption.Web)>
Public Class Product
    Public Property Name As String
    Public Property Quantity As Integer
    Public Property Price As Decimal
End Class
```

### 2. Use the generated context

```vb
Dim apple As New Product With {.Name = "Apple", .Quantity = 10, .Price = 1.5D}
Dim productTypeInfo = ProductJsonContext.Default.ProductTypeInfo
Dim json = JsonSerializer.Serialize(apple, productTypeInfo)
Dim product = JsonSerializer.Deserialize(json, productTypeInfo)
```

## Generated Code Example

For a class `Product`, the generator produces:

```vb
Partial Public Class ProductJsonContext
    Inherits JsonSerializerContext

    Public Shared Shadows ReadOnly Property [Default] As ProductJsonContext
        Get
            Return _defaultInstance.Value
        End Get
    End Property

    Public ReadOnly Property PersonTypeInfo As JsonTypeInfo(Of Person)
        Get
            If _person_TypeInfo Is Nothing Then _person_TypeInfo = CreatePersonTypeInfo(Options)
            Return _person_TypeInfo
        End Get
    End Property

    ' Full property mapping, options loading, and type resolution...
End Class
```

## Configuration Options

| Option      | Effect                                             |
|-------------|----------------------------------------------------|
| `Default`   | `WriteIndented = True`                             |
| `Strict`    | Case‑sensitive, ignore read‑only properties        |
| `Web`       | Case‑insensitive + ignore `null` values in output  |

## Current Limitations

- Only **public properties** are included in the JSON contract
- No support for structs due to their immutability
- No support for custom naming policies per property so far
- Nested generic types are supported, but the context must be generated for the exact closed type

## Contributing

Issues and pull requests are welcome. This generator aims to keep VB on par with C# for modern `System.Text.Json` features.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.