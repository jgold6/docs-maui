---
title: "Localization"
description: "Learn how to localize .NET MAUI apps using .NET resource files."
ms.date: 08/07/2023
---

# Localization

Localization is the process of adapting an app to meet the specific language or cultural requirements of a target market. To accomplish localization, the text and images in an app may need to be translated into multiple languages. A localized app automatically displays translated text based on the culture settings of the device.

.NET includes a mechanism for localizing apps using [resource files](/dotnet/core/extensions/create-resource-files). A resource file stores text and other content as name/value pairs that allow the app to retrieve content for a provided key. Resource files allow localized content to be separated from app code.

To localize a .NET Multi-platform App UI (.NET MAUI) app using resource files you should:

1. Create resource files. For more information, see [Create resource files](#create-resource-files).
1. Specify the app's default culture. For more information, see [Specify the app's default culture](#specify-the-apps-default-culture).
1. Perform platform setup. For more information, see [Perform platform setup](#perform-platform-setup).
1. Localize text. For more information, see [Localize text](#localize-text).
1. Localize images. For more information, see [Localize images](#localize-images).
1. Localize the app name. For more information, see [Localize the app name](#localize-the-app-name).
1. Test localization. For more information, see [Test localization](#test-localization).

In addition, the layout direction of an app can be overridden. For more information, see [Right to left localization](#right-to-left-localization).

## Create resource files

Resource files are XML files with a *.resx* extension that are compiled into binary resource (*.resources*) files during the build process. A localized app typically contains a default resource file with all strings used in the app, as well as resource files for each supported language.

Resource files contain the following information for each item:

- **Name** specifies the key used to access the text in code.
- **Value** specifies the translated text.
- **Comment** is an optional field containing additional information.

A resource file can be added with the **Add New Item** dialog in Visual Studio:

:::image type="content" source="media/localization/add-resource-file.png" alt-text="Screenshot of adding a resource file in Visual Studio.":::

Once the file is added, rows can be added for each text resource:

:::image type="content" source="media/localization/default-strings.png" alt-text="Screenshot of the default strings for the app.":::

The **Access Modifier** drop-down determines how Visual Studio generates the class used to access resources. Setting the Access Modifier to **Public** or **Internal** results in a generated class with the specified accessibility level. Setting the Access Modifier to **No code generation** doesn't generate a class file. The default resource file should be configured to generate a class file, which results in a file with the *.Designer.cs* extension being added to the project.

Once the default resource file is created, additional files can be created for each locale the app supports. Each additional resource file should have the same root filename as the default resource file, but should also include the culture in the filename. For example, if you add a resource file named *AppResources.resx*, you might also create resource files named *AppResources.en-US.resx* and *AppResources.fr-FR.resx* to hold localized resources for the English (United States) and French (France) cultures, respectively. In addition, you should set the **Access Modifier** for each additional resource file to **No code generation**.

At runtime, your app attempts to resolve a resource request in order of specificity. For example, if the device culture is **en-US** the application looks for resource files in this order:

1. AppResources.en-US.resx
1. AppResources.en.resx
1. AppResources.resx (default)

The following screenshot shows a Spanish translation file named *AppResources.es.resx*:

:::image type="content" source="media/localization/spanish-strings.png" alt-text="Screenshot of the Spanish strings for the app.":::

The localized resource file uses the same **Name** values specified in the default file but contains Spanish language strings in the **Value** column. Additionally, the **Access Modifier** is set to **No code generation**.

## Specify the app's default culture

For resource files to work correctly, the app must have a default culture specified. This is the culture whose resources are used if no localized resources for a particular culture can be found. To specify the default culture:

1. In Solution Explorer in Visual Studio, right-click your .NET MAUI app project and select **Properties**.
1. Select the **Package** tab.
1. In the **General** area, select the appropriate language/culture from the **Assembly neutral language** control:

    :::image type="content" source="media/localization/neutral-language.png" alt-text="Screenshot of setting the neutral language for the assembly.":::

1. Save your changes.

Alternatively, add the `<NeutralLanguage>` node to the first `<PropertyGroup>` in your .csproj, and specify your locale as its value:

```xml
<NeutralLanguage>en-US</NeutralLanguage>
```

> [!WARNING]
> If you don'y specify a default culture, the <xref:System.Resources.ResourceManager> class returns `null` values for any cultures without a specific resource file. When a default culture is specified, the <xref:System.Resources.ResourceManager> class returns results from the default resource file for unsupported cultures. Therefore, it's recommended that you always specify a default culture so that text is displayed for unsupported cultures.

## Perform platform setup

Some platforms require additional setup so that all .NET MAUI controls are localized.

### iOS and Mac Catalyst

On iOS and Mac Catalyst, you must declare all supported languages in the *Info.plist* file for your .NET MAUI app project. To do this, open the *Info.plist* file in an XML editor and set an array for the `CFBundleLocalizations` key and provide values that correspond to the resource files. In addition, ensure you set an expected language via the `CFBundleDevelopmentRegion` key:

```xml
<key>CFBundleLocalizations</key>
<array>
    <string>de</string>
    <string>es</string>
    <string>fr</string>
    <string>ja</string>
    <string>pt</string> <!-- Brazil -->
    <string>pt-PT</string> <!-- Portugal -->
    <string>ru</string>
    <string>zh-Hans</string>
    <string>zh-Hant</string>
</array>
<key>CFBundleDevelopmentRegion</key>
<string>en</string>
```

Alternatively, in Solution Explorer in Visual Studio, open the *Info.plist* file for your chosen platform in the **Generic PList Editor**. Then, set an array for the `CFBundleLocalizations` key and provide values that correspond to the resource files. In addition, ensure you set an expected language via the `CFBundleDevelopmentRegion` key:

:::image type="content" source="media/localization/info-plist.png" alt-text="Screenshot of the supported locales for the app in the generic Info.plist editor.":::

For more information about *Info.plist*, see [Information property list](~/macios/info-plist.md).

### Windows

To support multiple languages in a .NET MAUI app on Windows you must declare each supported language in the *Resources\Windows\Package.appxmanifest* file of your .NET MAUI app project:

1. Open the *Package.appxmanifest* file in a text editor and locate the following section:

    ```xml
    <Resources>
        <Resource Language="x-generate"/>
    </Resources>
    ```

1. Replace `<Resource Language="x-generate">` with `<Resource />` elements for each of your supported languages:

    ```xml
    <Resources>
        <Resource Language="en-US"/>      
        <Resource Language="de-DE"/>
        <Resource Language="es-ES"/>
        <Resource Language="fr-FR"/>
        <Resource Language="ja-JP"/>
        <Resource Language="pt-BR"/>
        <Resource Language="pt-PT"/>
        <Resource Language="ru-RU"/>
        <Resource Language="zh-CN"/>
        <Resource Language="zh-TW"/>        
    </Resources>
    ```

## Localize text

Text is localized using a class generated from your default resources file. The class is named based on the default resource filename. Given a default resource filename of *AppResources.resx*, Visual Studio generates a matching class named `AppResources` containing static properties for each entry in the resource file.

In XAML, localized text can be retrieved by using the `x:Static` markup extension to access the generated static properties:

```xaml
<ContentPage ...
             xmlns:strings="clr-namespace:LocalizationDemo.Resources.Strings">
    <VerticalStackLayout>
        <Label Text="{x:Static strings:AppResources.NotesLabel}" />
        <Entry Placeholder="{x:Static strings:AppResources.NotesPlaceholder}" />
        <Button Text="{x:Static strings:AppResources.AddButton}" />
    </VerticalStackLayout>
</ContentPage>
```

For more information about the `x:Static` markup extension, see [x:Static markup extension](~/xaml/fundamentals/markup-extensions.md#xstatic-markup-extension).

Localized text can also be retrieved in code:

```csharp
Label notesLabel = new Label();
notesLabel.Text = AppResources.NotesLabel,

Entry notesEntry = new Entry();
notesEntry.Placeholder = AppResources.NotesPlaceholder,

Button addButton = new Button();
addButton.Text = AppResources.AddButton,
```

The properties in the `AppResources` class use the <xref:System.Globalization.CultureInfo.CurrentUICulture> property value to determine which culture resource file to retrieve values from.

## Localize images

In addition to storing text, resource files can also store images and binary data. However, devices have a range of screen sizes and densities and each platform has functionality for displaying density-dependent images. Therefore, platform image localization functionality should be used instead of storing images in resource files.

### Android

On Android, localized images, known as drawables, are stored using a folder-based naming convention in the *Platforms\\Android\\Resources* folder. Folders are named *drawable* with a suffix for the language and culture. For example, the Spanish-language folder is named *drawable-es*. The folder name *drawable* should contain the images for your default language and culture. The build action of each image should be set to **AndroidResource**.

Only two characters are required when specifying a top-level language, such as *es*. However, when specifying a full locale, the folder name format requires a dash and lowercase *r* to separate the language from the culture. For example, the Mexico locale (es-MX) folder should be named *drawable-es-rMX*. The image file names in each locale folder should be identical:

:::image type="content" source="media/localization/images-folder-structure-android.png" alt-text="Screenshot of the localized folder structure in Visual Studio for images on Android.":::

### iOS

On iOS, localized images are stored using a folder-based naming convention in the *Platforms\\iOS\\Resources* folder. Folders are named with the language, and optional culture, followed by *.lproj*. For example, the Spanish-language folder is named *es.lproj*. The build action of each image should be set to **BundleResource**.

Only two characters are required when specifying a top-level language, such as *es*. However, when specifying a full locale, the folder name format requires a dash to separate the language from the culture. For example, the Mexico locale (es-MX) folder should be named *es-MX.lproj*. The image file names in each locale folder should be identical:

:::image type="content" source="media/localization/images-folder-structure-ios.png" alt-text="Screenshot of the localized folder structure in Visual Studio for images on iOS.":::

<!-- This is required on .NET 7 due to a bug in .NET MAUI, which might be fixed in .NET 8. -->
In addition, in your project file you must set the `IPhoneResourcePrefix` build property to the folder that contains the localized image folders:

```xml
<PropertyGroup Condition="$(TargetFramework.Contains('-ios'))">
  <IPhoneResourcePrefix>Platforms/iOS/Resources</IPhoneResourcePrefix>
</PropertyGroup>
```

If an image is not present for a particular language, iOS will fall back to the default native language folder and load the image from there.

### Mac Catalyst

On Mac Catalyst, localized images are stored using a folder-based naming convention in the *Platforms\\MacCatalyst\\Resources* folder. Folders are named with the language, and optional culture, followed by *.lproj*. For example, the Spanish-language folder is named *es.lproj*. The build action of each image should be set to **BundleResource**.

Only two characters are required when specifying a top-level language, such as *es*. However, when specifying a full locale, the folder name format requires a dash to separate the language from the culture. For example, the Mexico locale (es-MX) folder should be named *es-MX.lproj*. The image file names in each locale folder should be identical:

:::image type="content" source="media/localization/images-folder-structure-maccatalyst.png" alt-text="Screenshot of the localized folder structure in Visual Studio for images on MacCatalyst.":::

<!-- This is required on .NET 7 due to a bug in .NET MAUI, which might be fixed in .NET 8. -->
In addition, in your project file you must set the `IPhoneResourcePrefix` build property to the folder that contains the localized image folders:

```xml
<PropertyGroup Condition="$(TargetFramework.Contains('-maccatalyst'))">
  <IPhoneResourcePrefix>Platforms/MacCatalyst/Resources</IPhoneResourcePrefix>
</PropertyGroup>
```

If an image is not present for a particular language, MacCatalyst will fall back to the default native language folder and load the image from there.

### Windows

On Windows, localized images are stored using a folder-based naming convention in the *Platforms\\Windows\\Assets\\Images* folder. Folders are named with the language, and optional culture. For example, the Spanish-language folder is named *es* and the Mexico locale folder should be named *es-MX*. The image file names in each locale folder should be identical:

:::image type="content" source="media/localization/images-folder-structure-windows.png" alt-text="Screenshot of the localized folder structure in Visual Studio for images on Windows.":::

### Consume localized images

On Android, iOS, and MacCatalyst localized images can be consumed by setting the <xref:Microsoft.Maui.Controls.Image.Source> property of an <xref:Microsoft.Maui.Controls.Image> to the image filename. However, on Windows `Image.Source` must be set to the path and filename of the image. This can be accomplished in XAML by using the [`OnPlatform`](xref:Microsoft.Maui.Controls.Xaml.OnPlatformExtension) markup extension:

```xaml
<Image Source="{OnPlatform flag.png, WinUI=Platforms/Windows/Assets/Images/flag.png}" />
```

For more information about the [`OnPlatform`](xref:Microsoft.Maui.Controls.Xaml.OnPlatformExtension) markup extension, see [Customize UI appearance based on the platform](~/platform-integration/customize-ui-appearance.md#customize-ui-appearance-based-on-the-platform).

The equivalent C# code uses the <xref:Microsoft.Maui.Devices.DeviceInfo> API to obtain the current platform:

```csharp
FileImageSource imageSource = new FileImageSource();
if (DeviceInfo.Current.Platform == DevicePlatform.WinUI)
      imageSource.File = "Platforms/Windows/Assets/Images/flag.png";
else
      imageSource.File = "flag.png";

Image image = new Image { Source = imageSource };
```

## Localize the app name

A localized app name is specified per-platform, and doesn't use resource files.

### Android

On Android, the localized app name can be stored using a folder-based naming convention in the *Platforms\\Android\\Resources* folder. Folders are named *values* with a suffix for the language and culture. For example, the Spanish-language folder is named *values-es*. A *Strings.xml* file should be added to each folder, with a build action of **AndroidResource**, that sets a string to the localized app name.

Only two characters are required when specifying a top-level language, such as *es*. However, when specifying a full locale, the folder name format requires a dash and lowercase *r* to separate the language from the culture. For example, the Mexico locale (es-MX) folder should be named *values-es-rMX*.

Each translatable string is an XML element with the resource ID specified as the `name` attribute and the translated string as the value. You need to escape according to normal XML rules, and the `name` must be a valid Android resource ID (no spaces or dashes).

Therefore, to localize the app name add `<string>` element as the child of a `<resources>` element, set it's `name` attribute to a suitable ID with the translated string as the value:

```xml
<resources>
    <!-- French -->
    <string name="app_name">Maison</string>    
</resources>
```

Then, to use the localized app name in your app add the [`Label`](xref:Android.App.ActivityAttribute.Label) property to the [`Activity`](xref:Android.App.ActivityAttribute) in your app's `MainActivity` class, and set its value to `@string/id`:

```csharp
[Activity(Label = "@string/app_name", Theme = "@style/Maui.SplashTheme", MainLauncher = true, ConfigurationChanges = ConfigChanges.ScreenSize | ConfigChanges.Orientation | ConfigChanges.UiMode | ConfigChanges.ScreenLayout | ConfigChanges.SmallestScreenSize | ConfigChanges.Density)]
public class MainActivity : MauiAppCompatActivity
{
    protected override void OnCreate(Bundle savedInstanceState)
    {
        base.OnCreate(savedInstanceState);
    }
}
```

### iOS and Mac Catalyst

On iOS, the localized app name is stored using a folder-based naming convention in the *Platforms\\iOS\\Resources* folder. Folders are named with the language, and optional culture, followed by *.lproj*. For example, the Spanish-language folder is named *es.lproj*. A *InfoPlist.strings* file should be added to each folder, with a build action of **BundleResource**, that sets the `CFBundleDisplayName` key and value.

The syntax for localized string values is:

```text
/* comment */
"key"="localized-value";
```

You should escape the following characters in strings:

- `\"` quote
- `\\` backslash
- `\n` newline

Therefore, to localize the app name add a value for the `CFBundleDisplayName` key to the *InfoPlist.strings* file:

```text
/* French */
CFBundleDisplayName="Maisons";
```

Other keys that you can use to localize app-specific strings are:

- `CFBundleName` - specifics the short name of the app bundle, which may be displayed to users in situations such as the absence of a value for `CFBundleDisplayName`.
- `CFBundleShortVersionString` - specifies the release version number of the app bundle.
- `NSHumanReadableCopyright` - the copyright notice for the app bundle.

<!-- This is required on .NET 7 due to a bug in .NET MAUI, which might be fixed in .NET 8. -->
In addition, in your project file you must set the `IPhoneResourcePrefix` build property to the folder that contains the localized folders:

```xml
<PropertyGroup Condition="$(TargetFramework.Contains('-ios'))">
  <IPhoneResourcePrefix>Platforms/iOS/Resources</IPhoneResourcePrefix>
</PropertyGroup>
```

### Mac Catalyst

On MacCatalyst, the localized app name is stored using a folder-based naming convention in the *Platforms\\MacCatalyst\\Resources* folder. Folders are named with the language, and optional culture, followed by *.lproj*. For example, the Spanish-language folder is named *es.lproj*. A *InfoPlist.strings* file should be added to each folder, with a build action of **BundleResource** that sets the `CFBundleDisplayName` key and value.

The syntax for localized string values is:

```text
/* comment */
"key"="localized-value";
```

You should escape the following characters in strings:

- `\"` quote
- `\\` backslash
- `\n` newline

Therefore, to localize the app name add a value for the `CFBundleDisplayName` key to the *InfoPlist.strings* file:

```text
/* French */
CFBundleDisplayName="Maisons";
```

Other keys that you can use to localize app-specific strings are:

- `CFBundleName` - specifics the short name of the app bundle, which may be displayed to users in situations such as the absence of a value for `CFBundleDisplayName`.
- `CFBundleShortVersionString` - specifies the release version number of the app bundle.
- `NSHumanReadableCopyright` - the copyright notice for the app bundle.

<!-- This is required on .NET 7 due to a bug in .NET MAUI, which might be fixed in .NET 8. -->
In addition, in your project file you must set the `IPhoneResourcePrefix` build property to the folder that contains the localized folders:

```xml
<PropertyGroup Condition="$(TargetFramework.Contains('-maccatalyst'))">
  <IPhoneResourcePrefix>Platforms/MacCatalyst/Resources</IPhoneResourcePrefix>
</PropertyGroup>
```

### Windows

On Windows, your app name is defined in your app package manifest. Localizing this app name requires you to first specify the default language for the app, and then create a string resource file for each locale you intend to support. The string resource that represents the localized app name can then be consumed in your app package manifest by using the `ms-resource` URI scheme.

For more information about localizing strings in your app package manifest, see [Localize strings in your UI and app package manifest](/windows/uwp/app-resources/localize-strings-ui-manifest).

#### Specify the default language

For Windows resource files to work correctly, the Windows app must have a default language specified. This is the language whose resources are used if no localized resources for a particular language can be found. To specify the default culture:

1. In Solution Explorer, open the *Packageappxmanifest* file in the package manifest editor.
1. In the manifest editor, on the **Application** tab, set the **Default language** field to your chosen default language:

    :::image type="content" source="media/localization/windows-default-language.png" alt-text="Screenshot of setting the default language of a Windows app in the package manifest.":::

1. Save your changes.

At a minimum, you'll need to provide a string resource for the app name for this default language. This is the resource that will be loaded if no better match can be found for the user's preferred language or display language settings.

#### Create Windows resource files

On Windows, the localized app name can be stored in a Windows resource file for each locale. A Windows resource file is an XML file with a *.resw* extension that's compiled into a binary format and stored in a *.pri* file. The *.resw* file for each locale should be named *Resources.resx* and stored using a folder-based naming convention in the *Platforms\\Windows\\Strings\\* folder. Folders are named with the language, and optional culture. For example, the Spanish-language folder is named *es* and the Mexico locale folder should be named *es-MX*.

There's currently no Visual Studio item template for creating a Windows resource file in a .NET MAUI app. Therefore, to create a Windows resource file for each locale:

1. In the *Platforms\\Windows\\* folder of your .NET MAUI app project, create a *Strings* folder.
1. In the *Strings* folder, create a folder for each locale.
1. In the folder for each locale, create a new file named *Resources.resw* that contains the following XML:

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <root>
    <!--
      Microsoft ResX Schema

      Version 2.0

      The primary goals of this format is to allow a simple XML format
      that is mostly human readable. The generation and parsing of the
      various data types are done through the TypeConverter classes
      associated with the data types.

      Example:

      ... ado.net/XML headers & schema ...
      <resheader name="resmimetype">text/microsoft-resx</resheader>
      <resheader name="version">2.0</resheader>
      <resheader name="reader">System.Resources.ResXResourceReader, System.Windows.Forms, ...</resheader>
      <resheader name="writer">System.Resources.ResXResourceWriter, System.Windows.Forms, ...</resheader>
      <data name="Name1"><value>this is my long string</value><comment>this is a comment</comment></data>
      <data name="Color1" type="System.Drawing.Color, System.Drawing">Blue</data>
      <data name="Bitmap1" mimetype="application/x-microsoft.net.object.binary.base64">
          <value>[base64 mime encoded serialized .NET Framework object]</value>
      </data>
      <data name="Icon1" type="System.Drawing.Icon, System.Drawing" mimetype="application/x-microsoft.net.object.bytearray.base64">
          <value>[base64 mime encoded string representing a byte array form of the .NET Framework object]</value>
          <comment>This is a comment</comment>
      </data>

      There are any number of "resheader" rows that contain simple
      name/value pairs.

      Each data row contains a name, and value. The row also contains a
      type or mimetype. Type corresponds to a .NET class that support
      text/value conversion through the TypeConverter architecture.
      Classes that don't support this are serialized and stored with the
      mimetype set.

      The mimetype is used for serialized objects, and tells the
      ResXResourceReader how to depersist the object. This is currently not
      extensible. For a given mimetype the value must be set accordingly:

      Note - application/x-microsoft.net.object.binary.base64 is the format
      that the ResXResourceWriter will generate, however the reader can
      read any of the formats listed below.

      mimetype: application/x-microsoft.net.object.binary.base64
      value   : The object must be serialized with
              : System.Runtime.Serialization.Formatters.Binary.BinaryFormatter
              : and then encoded with base64 encoding.

      mimetype: application/x-microsoft.net.object.soap.base64
      value   : The object must be serialized with
              : System.Runtime.Serialization.Formatters.Soap.SoapFormatter
              : and then encoded with base64 encoding.

      mimetype: application/x-microsoft.net.object.bytearray.base64
      value   : The object must be serialized into a byte array
              : using a System.ComponentModel.TypeConverter
              : and then encoded with base64 encoding.
      -->
    <xsd:schema id="root" xmlns="" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:msdata="urn:schemas-microsoft-com:xml-msdata">
      <xsd:import namespace="http://www.w3.org/XML/1998/namespace" />
      <xsd:element name="root" msdata:IsDataSet="true">
        <xsd:complexType>
          <xsd:choice maxOccurs="unbounded">
            <xsd:element name="metadata">
              <xsd:complexType>
                <xsd:sequence>
                  <xsd:element name="value" type="xsd:string" minOccurs="0" />
                </xsd:sequence>
                <xsd:attribute name="name" use="required" type="xsd:string" />
                <xsd:attribute name="type" type="xsd:string" />
                <xsd:attribute name="mimetype" type="xsd:string" />
                <xsd:attribute ref="xml:space" />
              </xsd:complexType>
            </xsd:element>
            <xsd:element name="assembly">
              <xsd:complexType>
                <xsd:attribute name="alias" type="xsd:string" />
                <xsd:attribute name="name" type="xsd:string" />
              </xsd:complexType>
            </xsd:element>
            <xsd:element name="data">
              <xsd:complexType>
                <xsd:sequence>
                  <xsd:element name="value" type="xsd:string" minOccurs="0" msdata:Ordinal="1" />
                  <xsd:element name="comment" type="xsd:string" minOccurs="0" msdata:Ordinal="2" />
                </xsd:sequence>
                <xsd:attribute name="name" type="xsd:string" use="required" msdata:Ordinal="1" />
                <xsd:attribute name="type" type="xsd:string" msdata:Ordinal="3" />
                <xsd:attribute name="mimetype" type="xsd:string" msdata:Ordinal="4" />
                <xsd:attribute ref="xml:space" />
              </xsd:complexType>
            </xsd:element>
            <xsd:element name="resheader">
              <xsd:complexType>
                <xsd:sequence>
                  <xsd:element name="value" type="xsd:string" minOccurs="0" msdata:Ordinal="1" />
                </xsd:sequence>
                <xsd:attribute name="name" type="xsd:string" use="required" />
              </xsd:complexType>
            </xsd:element>
          </xsd:choice>
        </xsd:complexType>
      </xsd:element>
    </xsd:schema>
    <resheader name="resmimetype">
      <value>text/microsoft-resx</value>
    </resheader>
    <resheader name="version">
      <value>2.0</value>
    </resheader>
    <resheader name="reader">
      <value>System.Resources.ResXResourceReader, System.Windows.Forms, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089</value>
    </resheader>
    <resheader name="writer">
      <value>System.Resources.ResXResourceWriter, System.Windows.Forms, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089</value>
    </resheader>
  </root>
  ```

1. Open each *Resources.resw* file and add a string resource that represents the app's name:

    :::image type="content" source="media/localization/resw-editor.png" alt-text="Screenshot of the resw file editor in Visual Studio on Windows.":::

    > [!NOTE]
    > Resource identifiers are case insensitive, and must be unique per resource file.

1. Save the Windows resource files.

> [!NOTE]
> Windows resource files use a build action of `PRIResource`. This build action doesn't need setting on each *.resw* file in a .NET MAUI app, because it's implicitly applied.

An example of the folder and file structure is shown in the following screenshot:

:::image type="content" source="media/localization/windows-strings-folders.png" alt-text="Screenshot of the localized folder structure in Visual Studio for strings on Windows.":::

#### Consume the localized app name

The string resource that represents the app name can be consumed by using the `ms-resource` URI scheme:

1. In Solution Explorer, open the *Packageappxmanifest* file in the package manifest editor.
1. In the manifest editor, on the **Application** tab, set the **Display name** field to `ms-resource:` followed by the name of the string resource that identifies your app name:

    :::image type="content" source="media/localization/windows-localize-app-name.png" alt-text="Screenshot of setting the localized app name in the package manifest on Windows.":::

1. Save your changes.

## Test localization

At runtime, an app loads the appropriate localized resources on a per-thread basis, based on the culture specified by the <xref:System.Globalization.CultureInfo.CurrentUICulture> property.

Testing localization is best accomplished by changing your device language:

- On Android, this can be accomplished in the Settings app. In the Settings app, you can also set the language for each app without changing your device language.
- On iOS, this can be accomplished in the Settings app. In the Settings app, you can also set the language for each app without changing your device language.
- On Mac, this can be accomplished in System Settings. In System Settings, you can also set the language for each app without changing your device language.
- On Windows, this can be accomplished in Settings.

> [!WARNING]
> While it's possible to set the value of <xref:System.Globalization.CultureInfo.CurrentUICulture> in code, the resulting behavior is inconsistent across platforms so this isn't recommended for testing.

## Right-to-left localization

Flow direction, or layout direction, is the direction in which the UI elements on the page are scanned by the eye. Some languages, such as Arabic and Hebrew, require that UI elements are laid out in a right-to-left flow direction. .NET MAUI apps automatically respect the device's flow direction based on the selected language and region. For information about how to retrieve the flow direction of the device, based on its locale, see [Get the layout direction](~/platform-integration/appmodel/app-information.md#get-the-layout-direction).

To override the flow direction of an app, set the <xref:Microsoft.Maui.Controls.VisualElement.FlowDirection?displayProperty=nameWithType> property. This property gets or sets the direction in which UI elements flow within any parent element that controls their layout, and should be set to one of the <xref:Microsoft.Maui.FlowDirection> enumeration values:

- `LeftToRight`
- `RightToLeft`
- `MatchParent`

<!-- Setting the <xref:Microsoft.Maui.Controls.VisualElement.FlowDirection> property to `RightToLeft` on an element sets the alignment to the right, the reading order to right-to-left, and the layout of the control to flow from right-to-left. -->

> [!WARNING]
> Changing the <xref:Microsoft.Maui.Controls.VisualElement.FlowDirection> property at runtime causes an expensive layout process that will affect performance.

The default <xref:Microsoft.Maui.Controls.VisualElement.FlowDirection> property value for an element is `MatchParent`. Therefore, an element inherits the `FlowDirection` property value from its parent in the visual tree, and any element can override the value it gets from its parent.

> [!TIP]
> If you do need to change the flow direction, set the <xref:Microsoft.Maui.Controls.VisualElement.FlowDirection> property on a window, page or root layout. This causes all of the elements contained within the page, or root layout, to respond appropriately to the flow direction.

### Platform setup

Specific platform setup is required to enable right-to-left locales.

#### Android

App's created using the .NET MAUI app project template will automatically include support for right-to-left locales. This support is enabled by the `android:supportsRtl` attribute being set to `true` on the `<application>` node in the app's *AndroidManifest.xml* file:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application ... android:supportsRtl="true" />
    ...
</manifest>
```

Right-to-left localization can then be tested by changing the device or emulator to use the right-to-left language. Alternatively, provided that you've activated developer options in the Settings app, you can enable **Force RTL layout direction** in **Settings > Developer Options**. For information on configuring developer options, see [Configure on-device developer options](https://developer.android.com/studio/debug/dev-options) on developer.android.com.

#### iOS and Mac Catalyst

The required right-to-left locale should be added as a supported language to the array items for the `CFBundleLocalizations` key in *Info.plist*. The following example shows Arabic having been added to the array for the `CFBundleLocalizations` key:

```xml
<key>CFBundleLocalizations</key>
<array>
    <string>en</string>
    <string>ar</string>
</array>
```

Right-to-left localization can then be tested by changing the language and region on the device or simulator to a right-to-left locale that was specified in *Info.plist*.

#### Windows

The required language resources should be specified in the `<Resources>` node of the *Package.appxmanifest* file. Replace `<Resource Language="x-generate">` with `<Resource />` elemnts for each of your supported languages. For example, the following markup specifies that "en" and "ar" localized resources are available:

```xml
<Resources>
    <Resource Language="en" />
    <Resource Language="ar" />
</Resources>
```

Right-to-left localization can then be tested by changing the language and region on the device to the appropriate right-to-left locale.