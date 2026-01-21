# MonoTouch.Dialog - .NET 9 iOS Migration Guide

## Overview

MonoTouch.Dialog is a dialog/form building library originally created by Miguel de Icaza for Xamarin.iOS. This fork has been migrated to .NET 9 iOS SDK-style project format.

## Project Structure

```
MonoTouch.Dialog/
├── MonoTouch.Dialog/
│   ├── MonoTouch.Dialog.iOS.csproj    # .NET 9 iOS library project
│   ├── Elements.cs                     # Core UI elements
│   ├── Elements/                       # Additional element types
│   │   └── Json.cs                     # EXCLUDED - System.Json not available
│   ├── Utilities/
│   │   └── LocalizationExtensions.cs   # Fixed API change
│   └── Images/
│       └── arrow.png                   # Embedded resource
└── Sample/
    ├── Sample.iOS.csproj              # .NET 9 iOS sample app
    └── Main.cs                        # Modified to remove Json demo
```

## .NET 9 iOS Project Configuration

### Library Project (MonoTouch.Dialog.iOS.csproj)

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0-ios</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>false</ImplicitUsings>
    <DefineConstants>__IOS__;__UNIFIED__</DefineConstants>
    <RootNamespace>MonoTouch.Dialog</RootNamespace>
    <AssemblyName>MonoTouch.Dialog</AssemblyName>
    <OutputType>Library</OutputType>
  </PropertyGroup>

  <!-- Exclude incompatible files -->
  <ItemGroup>
    <Compile Remove="docs\**" />
    <Compile Remove="Elements\Json.cs" />
    <None Remove="docs\**" />
    <None Remove="Info.plist" />
  </ItemGroup>

  <!-- Embedded resources -->
  <ItemGroup>
    <EmbeddedResource Include="Images\arrow.png">
      <LogicalName>arrow.png</LogicalName>
    </EmbeddedResource>
  </ItemGroup>
</Project>
```

### Sample App Project (Sample.iOS.csproj)

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0-ios</TargetFramework>
    <OutputType>Exe</OutputType>
    <Nullable>enable</Nullable>
    <ImplicitUsings>false</ImplicitUsings>
    <DefineConstants>__IOS__;__UNIFIED__</DefineConstants>
    <SupportedOSPlatformVersion>13.0</SupportedOSPlatformVersion>
    <ApplicationId>com.sample.monotouchdialog</ApplicationId>
  </PropertyGroup>

  <ItemGroup>
    <ProjectReference Include="..\MonoTouch.Dialog\MonoTouch.Dialog.iOS.csproj" />
  </ItemGroup>
</Project>
```

## API Changes from Xamarin.iOS to .NET 9 iOS

### 1. NSBundle.LocalizedString → GetLocalizedString

**File:** `Utilities/LocalizationExtensions.cs`

```csharp
// Old (Xamarin.iOS)
return NSBundle.MainBundle.LocalizedString(text, String.Empty, String.Empty);

// New (.NET 9 iOS)
return NSBundle.MainBundle.GetLocalizedString(text, String.Empty, String.Empty);
```

### 2. UIImagePickerControllerDelegate.FinishedPickingImage → FinishedPickingMedia

**File:** `Elements.cs` (ImagePickerController class)

```csharp
// Old (Xamarin.iOS)
public override void FinishedPickingImage(UIImagePickerController picker, UIImage image, NSDictionary editingInfo)
{
    container.Picked(image);
    ...
}

// New (.NET 9 iOS)
public override void FinishedPickingMedia(UIImagePickerController picker, NSDictionary info)
{
    var image = info[UIImagePickerController.OriginalImage] as UIImage;
    if (image != null)
    {
        container.Picked(image);
        ...
    }
}
```

### 3. System.Json Not Available

**Solution:** Exclude `Elements/Json.cs` from compilation

The `JsonElement` class uses `System.Json` which is not available in .NET 9. Options:
- Exclude the file (current approach)
- Rewrite using `System.Text.Json`
- Add Newtonsoft.Json dependency

## Define Constants

Required constants for compatibility:
- `__IOS__` - Indicates iOS platform
- `__UNIFIED__` - Uses unified API (not classic MonoTouch)

The `__UNIFIED__` constant controls conditional compilation for namespace imports:
```csharp
#if __UNIFIED__
using Foundation;
using UIKit;
#else
using MonoTouch.Foundation;
using MonoTouch.UIKit;
#endif
```

## Referencing from App Projects

Replace DLL reference with ProjectReference:

```xml
<!-- Old: DLL Reference -->
<Reference Include="MonoTouch.Dialog">
  <HintPath>..\libs\songtive-engine\libs\MonoTouch.Dialog.dll</HintPath>
</Reference>

<!-- New: Project Reference -->
<ProjectReference Include="..\libs\MonoTouch.Dialog\MonoTouch.Dialog\MonoTouch.Dialog.iOS.csproj" />
```

## Build Commands

```bash
# Build library only
dotnet build libs/MonoTouch.Dialog/MonoTouch.Dialog/MonoTouch.Dialog.iOS.csproj

# Build sample app
dotnet build libs/MonoTouch.Dialog/Sample/Sample.iOS.csproj
```

## Known Warnings

The library compiles with nullable reference warnings due to legacy code patterns. These are non-breaking but can be addressed by:
- Adding `#nullable disable` at file level for legacy files
- Or fixing nullable annotations throughout

## Custom Elements

If your app uses custom MonoTouch.Dialog elements not in this library (e.g., `ButtonElement`, `AppRaterElement`), you'll need to:
1. Add them to this fork
2. Or create them in your app project inheriting from `Element`

## Troubleshooting

### "LocalizedString not found"
Change `LocalizedString` to `GetLocalizedString`

### "FinishedPickingImage no suitable method to override"
Update to `FinishedPickingMedia` with new signature (see above)

### "System.Json not found"
Exclude `Elements/Json.cs` or rewrite using `System.Text.Json`

### Nullability warnings
Add `<Nullable>disable</Nullable>` to PropertyGroup if warnings are excessive
