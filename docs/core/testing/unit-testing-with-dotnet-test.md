---
title: "在 .NET Core 中使用 dotnet 测试的单元测试 | Microsoft Docs"
description: "在 .NET Core 中使用 dotnet 测试的单元测试"
keywords: .NET, .NET Core
author: ardalis
ms.author: wiwagn
ms.date: 03/21/2017
ms.topic: article
ms.prod: .net-core
ms.devlang: dotnet
ms.assetid: bdcdb812-6f13-4f20-9e90-0c0977937142
translationtype: Human Translation
ms.sourcegitcommit: 4a1f0c88fb1ccd6694f8d4f5687431646adbe000
ms.openlocfilehash: 3ca312509d7ba7a7759d1ac294f79cc359419c52
ms.lasthandoff: 03/22/2017

---

# <a name="unit-testing-in-net-core-using-dotnet-test"></a>在 .NET Core 中使用 dotnet 测试的单元测试

本教程介绍分步构建示例解决方案的交互式体验，以了解单元测试概念。 如果希望使用预构建解决方案学习本教程，请在开始前[查看或下载示例代码](https://github.com/dotnet/docs/tree/master/samples/core/getting-started/unit-testing-using-dotnet-test/)。

### <a name="creating-the-source-project"></a>创建源项目

打开 shell 窗口。 创建一个名为 *unit-testing-using-dotnet-test* 的目录，以保留该解决方案。 在此新目录中，创建一个 *PrimeService* 目录。 目录结构目前如下所示：

```
/unit-testing-using-dotnet-test
    /PrimeService
```

将 *PrimeService* 作为当前目录，然后运行 [`dotnet new classlib`](../tools/dotnet-new.md) 以创建源项目。 将 *Class1.cs* 重命名为 *PrimeService.cs*。 为了使用由测试驱动的开发 (TDD)，需对 `PrimeService` 类创建故障实现：

```csharp
using System;

namespace Prime.Services
{
    public class PrimeService
    {
        public bool IsPrime(int candidate) 
        {
            throw new NotImplementedException("Please create a test first");
        } 
    }
}
```

### <a name="creating-the-test-project"></a>创建测试项目

将目录更改回 *unit-testing-using-dotnet-test* 目录，并创建 *PrimeServices.Tests* 目录。 目录结构如下所示：

```
/unit-testing-using-dotnet-test
    /PrimeService
        Source Files
        PrimeService.csproj
    /PrimeService.Tests
```

将 *PrimeService.Tests* 目录作为当前目录，并使用 [`dotnet new xunit`](../tools/dotnet-new.md) 创建一个新项目。 这会创建将 xUnit 用作测试库的测试项目。 生成的模板在 *PrimeServiceTests.csproj* 中配置了测试运行程序：

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.NET.Test.Sdk" Version="15.0.0" />
  <PackageReference Include="xunit" Version="2.2.0" />
  <PackageReference Include="xunit.runner.visualstudio" Version="2.2.0" />
</ItemGroup>
```

测试项目需要其他包创建和运行单元测试。 `dotnet new` 在以前的步骤中已添加 xUnit 和 xUnit 运行程序。 现在，将 `PrimeService` 类库作为另一个依赖项添加到项目中。 使用 [`dotnet add reference`](../tools/dotnet-add-reference.md) 命令：

```
dotnet add reference ../PrimeService/PrimeService.csproj
```

另一个选项是编辑 *PrimeService.Tests.csproj* 文件。 直接在第一个 `<ItemGroup>` 节点下，添加另一个具有库项目引用的 `<ItemGroup>`：

```xml
<ItemGroup>
  <ProjectReference Include="..\PrimeService\PrimeService.csproj" />
</ItemGroup>
```

可以在 GitHub 上的[示例存储库](https://github.com/dotnet/docs/blob/master/samples/core/getting-started/unit-testing-using-dotnet-test/PrimeService.Tests/PrimeService.Tests.csproj)中看到整个文件。

最终的解决方案布局如下所示：

```
/unit-testing-using-mstest
    /PrimeService
        Source Files
        PrimeService.csproj
    /PrimeService.Tests
        PrimeService
        PrimeServiceTests.csproj
```

## <a name="creating-the-first-test"></a>创建第一个测试

在构建库或测试前，执行 *PrimeService.Tests* 目录中的 [`dotnet restore`](../tools/dotnet-restore.md)。 此命令将为每个项目还原所有必要的 NuGet 包。

TDD 方法要求编写一个失败的测试，使其通过测试，然后重复该过程。 从 *PrimeService.Tests* 目录删除 *UnitTest1.cs*，并创建一个名为 *PrimeService_IsPrimeShould.cs* 且包含以下内容的新 C# 文件：

```csharp
using Xunit;
using Prime.Services;

namespace Prime.UnitTests.Services
{
    [TestClass]
    public class PrimeService_IsPrimeShould
    {
        private readonly PrimeService _primeService;

        public PrimeService_IsPrimeShould()
        {
            _primeService = new PrimeService();
        }

        [Fact]
        public void ReturnFalseGivenValueOf1()
        {
            var result = _primeService.IsPrime(1);

            Assert.False(result, $"1 should not be prime");
        }
    }
}
```

`[Fact]` 属性将方法表示为单个测试。 执行 [`dotnet test`](../tools/dotnet-test.md) 以构建测试和类库，然后运行测试。 xUnit 测试运行程序包含要运行测试的程序入口点。 `dotnet test` 启动测试运行程序，并向该测试运行程序提供命令行参数以指示包含测试的程序集。

测试失败。 尚未创建实现。 在 `PrimeService` 类中编写最简单的代码，使此测试通过：

```csharp
public bool IsPrime(int candidate) 
{
    if (candidate == 1) 
    { 
        return false;
    } 
    throw new NotImplementedException("Please create a test first");
} 
```

在 *PrimeService.Tests* 目录中，再次运行 `dotnet test`。 `dotnet test` 命令构建 `PrimeService` 项目，然后构建 `PrimeService.Tests` 项目。 构建这两个项目后，该命令将运行此单项测试。 测试通过。

### <a name="adding-more-features"></a>添加更多功能

你已经通过了一个测试，现在可以编写更多测试。 质数有其他几种简单情况：0，-1。 可以将其添加为具有 `[Fact]` 属性的新测试，但这很快就会变得枯燥乏味。 还有其他 xUnit 属性，可使你编写类似测试套件。  `[Theory]` 属性表示执行相同代码，但具有不同输入参数一系列测试。 可以使用 `[InlineData]` 属性来指定这些输入的值。 
 
利用这两个属性来创建测试小于 2 的几个值的单个理论（而不是创建新测试），其中，2 是最小质数：

[!code-csharp[Sample_TestCode](../../../samples/core/getting-started/unit-testing-using-dotnet-test/PrimeService.Tests/PrimeService_IsPrimeShould.cs?name=Sample_TestCode)]

运行 `dotnet test`，两项测试均失败。 若要使所有测试通过，可以更改方法开头的 `if` 子句：

```csharp
if (candidate < 2)
```

通过在主库中添加更多测试、理论和代码继续循环访问。 将以[已完成的测试版本](https://github.com/dotnet/docs/blob/master/samples/core/getting-started/unit-testing-using-dotnet-test/PrimeService.Tests/PrimeService_IsPrimeShould.cs)和[库的完整实现](https://github.com/dotnet/docs/blob/master/samples/core/getting-started/unit-testing-using-dotnet-test/PrimeService/PrimeService.cs)结束。

你已生成一个小型库和该库的一组单元测试。 已结构化解决方案，以无缝添加新包和测试，并可将大部分时间和精力投入到解决应用程序的目标中。

