Here you can find documentation of public APIs for [my](https://www.linkedin.com/in/laurisuoranta/) packages and SDKs.

Note: Most presentation-related attributes and enums are part of the 'Unified Presentation Model' and defined in Suoranta.BaseLib. The model is to ensure easy portability of custom business objects between platforms (Desktop and Web).
## Packages
- [Suoranta.BaseLib](Suoranta.BaseLib/): Base type library for all packages.
- [Suoranta.DocBuilder](Suoranta.DocBuilder/): Build HTML and Markdown documents flexibly with fluent syntax.
- [Suoranta.EnterpriseLib](Suoranta.EnterpriseLib/): DTO-To-LINQ (or "query objects"), MVC/WebAPI base controllers, Entity-DTO-Entity mapping, mock data generation, HTTP client hints middleware, application hosts,other functionalities related to Entity Framework and ASP.NET MVC enterprise applications. (Note: Covers three packages. Some functionalities are only for classic ASP.NET MVC as they are not needed in ASP.NET Core.) [Examples](EnterpriseApps/Examples.html) [Presentation](EnterpriseApps/EnterpriseLib.pdf)
- [Suoranta.Finance](Suoranta.Finance/): Fundamental & quantitative finance framework. Types related to financial markets, particularly different kinds of time series. 
- [Suoranta.NaturalLanguage](Suoranta.NaturalLanguage/): Advanced text pattern searching in documents and images, transliteration, other functionalities related to natural language processing.
- [Suoranta.Speech](Suoranta.Speech/): Voice commands for WinForms and other .NET apps.
- [Suoranta.Storage](Suoranta.Storage/): Virtual file system, binary formatter framework.
- [Suoranta.WebApp](Suoranta.WebApp/): Upgraded Razor infrastructure, Bootstrap & custom widgets framework, Kendo UI Razor accelerators, .NET representation of View DOM, and more. Currently targets classic ASP.NET MVC 5 but most is applicable on ASP.NET Core. [Presentation](EnterpriseApps/WebApp.pdf)
- [WebApp JavaScript APIs](Suoranta.WebAppJS/): Component-based client side scripting architecture based on the latest HTML5 standard extensions. Data bindings inspired by WinForms bindings (see [uicomponent](Suoranta.WebAppJS/module-uicomponent.html)). Multilayer async forms. SignalR client. Not required for basic WebApp development. (Documentation coverage incomplete. Unfortunately, JsDoc has a weird tendency to put fragments of class documentation into that of the containing module, which is why the docs look messy.)

See the [dependency diagram of all packages](PackageDependencies.html).

## SDKs
- [Sharpshooter.SDK & SharpTrader.SDK](Sharpshooter.SDK/): Provide extensibility features of [Sharpshooter & SharpTrader](Sharpshooter/Sharpshooter.html) (Note: Covers two packages.). [Presentation](Sharpshooter/Sharpshooter.pdf)

_Inquiries: packages at suoranta dot net_
