# ASP.NET Core MVC

## Célkitűzés

Egyszerű szerver oldalon renderelt felületek készítésének alapszintű elsajátítása ASP.NET Core technológia segítségével.

## Előfeltételek

A labor elvégzéséhez szükséges eszközök:

- Microsoft SQL Server (LocalDB vagy Express edition, Visual Studio telepítővel telepíthető)
- Visual Studio 2022 .NET 6 SDK-val telepítve

Amit érdemes átnézned:

- EF Core előadás anyaga
- ASP.NET Core Web API (kliens renderelt) előadás anyaga
- ASP.NET Core MVC, Razor Pages (szerver rendererelt) előadás anyaga
- A használt adatbázis [sémája](https://BMEVIAUBB04.github.io/gyakorlat-mssql/sema.html)

## Feladat 0: Kiinduló projekt letöltése, indítása

Az előző laborokon megszokott adatmodellt fogjuk használni MS SQL LocalDB segítségével. Az adatbázis sémájában néhány mező a .NET-ben ismeretes konvencióknak megfelelően átnevezésre került, felépítése viszont megegyezik a korábban megismertekkel.

1. Töltsük le a GitHub repository-t a reposiory főoldaláról (https://github.com/BMEVIAUBB04/gyakorlat-rest-web-api > *Code* gomb, majd *Download ZIP*) vagy a közvetlen [letöltő link](https://github.com/BMEVIAUBB04/gyakorlat-rest-web-api/archive/refs/heads/master.zip) segítségével. 
2. Csomagoljuk ki
3. Nyissuk meg a kicsomagolt mappa AcmeShop alkönyvtárban lévő solution fájlt.

A kiinduló solution egyelőre egy projektből áll:`AcmeShop.Data`: EF modellt, a hozzá tartozó kontextust (`AcmeShopContext`) tartalmazza. Hasonló az EF Core gyakorlaton generált kódhoz, de ez Code-First migrációt is tartalmaz (`Migrations` almappa).

## Feladat 1: Webes projekt elkészítése

1. Adjunk a solutionhöz egy új web projektet
    - Típusa: ASP.NET Core Web App (Model-View-Controller) (**nem Web Api!, nem sima Web App, fontos a zárójeles rész!**)
    - Neve: *AcmeShop.Mvc*
    - Framework: .NET 6.0
    - Authentication type: *None*
    - HTTPS, Docker: kikapcsolni

1. Függőségek felvétele az új projekthez
    - adjuk meg projektfüggőségként az `AcmeShop.Data`-t
    - adjuk hozzá a *Microsoft.EntityFrameworkCore.Design* NuGet csomagot

1. Adatbáziskapcsolat, EF beállítása
    - connection string beállítása a konfigurációs fájlban (appsettings.json). A nyitó `{` jel után
    ```javascript
     "ConnectionStrings": {
       "AcmeShopContext": "Data Source=(localdb)\\MSSQLLocalDB;Initial Catalog=AcmeShop"
     },
    ```
   - connection string kiolvasása a konfigurációból, `AcmeShopContext` példány konfigurálása ezen connection string alapján, `AcmeShopContext` példány regisztrálása DI konténerbe. Program.cs-be, a `builder.Build()` sor elé:
    ```csharp
    builder.Services.AddDbContext<AcmeShopContext>(
        options => options.UseSqlServer(
            builder.Configuration.GetConnectionString(nameof(AcmeShopContext))));
    ```

1. Ha van már adatbázis _AcmeShop_ néven, töröljük le.
1. Fordítsuk a solutiont.
1. Adatbázis inicializálása Package Manager Console (PMC)-ban
   - Indítandó projekt az `AcmeShop.Api` projekt legyen (jobbklikk az AcmeShop.Api-n > *Set as Startup Project*)
   - A PMC-ben a Defult projekt viszont az `AcmeShop.Data` legyen
   - PMC-ből generáltassuk az adatbázist az alábbi paranccsal
    ```powershell
    Update-Database
    ```

1. Projekt indítása

## Feladat 2: Generált webalkalmazás vizsgálata


## Feladat 3: Adatbázis objektumok lekérdezése és megjelenítése

Az eddig legenerált MVC oldalak nem használták az adatbázisunkat. Vegyünk fel új kontrollereket és nézeteket, melyek segítségével le tudjuk kérdezni az adatbázist (a kontroller feladata) és az eredményt HTML-be tudjuk formázni (ez a nézetek feladata)! A leggyorsabb módja ennek a kódgenerálás (scaffolding).

1. Adjunk hozzá az MVC projekthez a *Microsoft.VisualStudio.Web.CodeGeneration.Design* NuGet csomagot.
1. Az AcmeShopContext.cs alján (_Data projekt_) kommentezzük vissza az `AcmeShopContextFactory` osztályt. (Erre nem kellene szükség legyen, valószínűleg a generátorban lévő bug miatt kell mégis.)
1. Fordítsuk az MVC projektet.
1. PMC-ben telepítsük az ASP.NET Core kódgeneráló eszközt, ha még korábban nem telepítettük az adott gépen
    ```powershell
    dotnet tool install -g dotnet-aspnet-codegenerator
    ```
1. Lépjünk be a projekt könyvtárába
    ```powershell
    cd .\AcmeShop.Mvc
    ```
1. Generáljunk a kódgenerálóval kontrollert és kapcsolódó nézeteket a `Termek` entitáshoz (`-m`), mely a `AcmeShopContext` kontextushoz  (`-dc`) tartozik. A generált kontroller neve legyen `TermekController` (`-name`), az `AcmeShop.Mvc.Controllers` névtérbe  (`-namespace`) kerüljön. A generált fájl a *Controllers* mappába (`-outDir`) kerüljön. A generált nézetek használják az alapértelmezett (projektben már meglévő) layoutot (`-udl`).
    ```powershell
    dotnet aspnet-codegenerator controller -m AcmeShop.Data.Termek -dc AcmeShop.Data.AcmeShopContext -outDir Controllers -name TermekekController -namespace AcmeShop.Mvc.Controllers -udl
    ```
1. Kommentezzük ki az `AcmeShopContextFactory` osztályt.
1. Vegyünk fel egy új navigációs lehetőséget a Views\Shared\_Layout.cshtml-be. Másoljuk le a _Home_ menüpontot reprezentáló `<li>` címkét saját maga alá és írjuk át az `asp-controller` értékét az `<a>` gyerekcímkében `Termekek`-re és az `<a>` címkék közötti szöveget is a kívánt menüpont névre, pl. _Termékek_. A teljes `<li>` valami az alábbihoz hasonló lesz:
``` HTML
<li class="nav-item">
    <a class="nav-link text-dark" asp-area="" asp-controller="Termekek" asp-action="Index">Termékek</a>
</li>
```

Vegyük észre, hogy nem állítottuk be az `<a>` elem `href` tulajdonságát, helyette az [`asp-page` TagHelper](https://docs.microsoft.com/en-US/aspnet/core/mvc/views/tag-helpers/intro)t használtuk, ami a kontroller és a kontrolleren belüli művelet neve alapján az az URL-t fogja nekünk generálni az elkészülő HTML-be, amit böngészőben navigálva a megadott action hívódik meg.

Teszteljük az így létrejött alkalmazást, teszteljük a termékekkel kapcsolatos funkciókat! A tesztelt oldalak forráskódját is tekintsük át (a kontrollert és a nézeteket és is)!

![Scaffolding eredménye](assets/1-scaffolding-5.png)

A legtöbb funkció alapvetően működőképes, de lehet találni hibákat, kényelmetlenségeket. Például:
- A törlés általában csak akkor működik, ha általunk felvett termékről van szó (például a _Create New_ link segítségével). A legtöbb alapból felvett terméknek ugyanis van már valamilyen érintettsége idegen kulcs kényszerben.
- A kapcsolódó elemeknek (kategória, áfakulcs) csak az azonosítójukkal szerepelnek, ez elég kényelmetlenné tesz minden funkciót ahol megjelennek - pl. nem tudjuk melyik kategóriát jelentik az egyes azonosítók.

## Feladat 4: Listázó nézet szépítése

A _Views/Termekek/Index.cshtml_ felelős a termékek listájának megjelenítéséért. A `TermekekController.Index` ugyan nem adja meg, hogy ez a nézet legyen a szerencsés, de a nézet felderítés algoritmusa (_Views/[Kontroller név]/[Függvény név]_) ezt jelöli ki.

Vizsgáljuk meg ezen nézet kódját és végezzük el az alábbi módosításokat.

1. Írjuk át az oldal címét (HTML `<title>` címke). Ez a böngésző címsorában vagy a böngészőfülön megjelenő felirat. Ezt MVC-ben általában nem a nézet írja ki közvetlenül (`<title>` tag-et használva), hanem valamelyik layout nézet. Esetünkben konkrétan a Views/Shared/_Layout.cshtml. Figyeljük meg ez utóbbi nézet kódjában a `<title>@ViewData["Title"] - AcmeShop.Mvc</title>` sort. Ebből látható, hogy a ViewData szótárban a `Title` kulcshoz tartozó érték kerül a `<title>` címkébe. Szerencsére ezt az értéket az _Index.cshtml_-ben is állít(hat)juk. A nézet tetején írjuk át a vonatkozó C# blokkban a beállított értéket.

    ```razor
    @{
        ViewData["Title"] = "Terméklista";
    }
    ```

1. Rögtön ezalatt egy `<h1>` címke található, ami az oldal címsorát adja. Ezt is írjuk át, pl.

    ```html
    <h1>Termékek listája</h1>
    ```

1. Ez a nézet ne jelenítse meg a képet és a leírást. Töröljük/kommentezzük ki (`@*...*@`) HTML táblázat fejlécéből és törzséből is.

    ```razor
    @*<th>
        @Html.DisplayNameFor(model => model.Leiras)
    </th>
    <th>
        @Html.DisplayNameFor(model => model.Kep)
    </th>*@
    ```

    ```razor
    @*<td>
        @Html.DisplayFor(modelItem => item.Leiras)
      </td>
      <td>
        @Html.DisplayFor(modelItem => item.Kep)
      </td>*@
    ```
    
1. A táblázatban a fejléc feliratok egy az egyben a property-k nevei, ami felhasználói szemmel elég ronda. Ezen feliratokat a `DisplayNameFor` HTML helper generálja. Ez a helper alapvetően csak utolsó lehetőségként használja a property nevét, előnyben részesíti a property-re tett `Display` attribútumot (a `System.ComponentModel.DataAnnotations` névtérből). Tegyük fel ezen attribútumot első körben a `Kategoria` property-re (a _Data_ projekt `Termek` osztályában).

    ```csharp
    [Display(Name = "Kategória")]
    public Kategoria? Kategoria { get; set; }
    ```

1. A kapcsolódó elemekből (áfakulcs, kategória) a táblázatban csak az azonosítók szerepelnek. A megjelenítendő propertyt a `DisplayFor` HTML helper jelöli ki. Írjuk át a kategória megjelenítéséért felelős `DisplayFor` hívást, hogy az azonosító helyett a nevet használja.

    ```razor
    @*@Html.DisplayFor(modelItem => item.Kategoria.Id)*@
    @Html.DisplayFor(modelItem => item.Kategoria.Nev)
    ```

1. Próbáljuk ki a változásokat, ellenőrizzük, hogy a következők megváltoztak-e:
- oldal címe
- oldal címsora
- leírás, kép oszlopok
- kategória oszlop fejléce
- kategória oszlop értéke

![Scaffolding eredménye](assets/index_view_rev1.png)

Bónuszként láthatjuk, hogy a termék részletes oldalon (_Details_ link) is a kategória neve megváltozott, mert ugyanazon `Display` attribútumot használja a HTML helper (`@Html.DisplayNameFor(model => model.Kategoria)`).

## Feladat 5 

A termékek szerkesztő és létrehozás (Edit, Create) oldalain láthatjuk, hogy a generált kódunk felismerte a kapcsolódó entitásokat is, amiket legördülő listából választhatunk ki. Ez nagyon jó, de az azonosító alapján nehezen tudjuk megmondani a helyes termékkategóriát és ÁFA-kulcsot, így helyesebb volna a nevüket megjeleníteni a felületen.

1. Vizsgáljuk meg az új termék létrehozásához tartozó műveletet. Igazából két műveletről van szó, a `Create()` művelet a **/Termekek/Create** címre navigáláskor hívódik meg, míg a másik `HttpPost` attribútummal ellátott változat ugyanezen címre egy űrlapot visszaküldenek. (HTML űrlapok visszaküldése tipikusan HTTP POST üzenettel történik). Az előbbi művelet felelős a létrehozó űrlap megjelenítéséért: feltölt néhány `ViewData` adatot, a többi a nézet (_Views/Termekek/Create.cshtml_) feladata. A `ViewData`-ba minden kapcsolódó elemhez egy-egy `SelectList` kerül. A `SelectList` a legördülő menükhöz használható modellként, összefogja a legördülő menühöz kapcsolódó listát, hogy a listaelemtípus melyik propertyjét kell használni feliratként, illetve elemazonosítóként. Pont a feliratot szeretnénk állítani, ezért módosítsuk a kategóriához létrehozott `SelectList`-et:

    ```csharp
    ViewData["KategoriaId"] = new SelectList(_context.Kategoria, nameof(Kategoria.Id), nameof(Kategoria.Nev));
    ```

1. A kapcsolódó nézetből (_Views/Termekek/Create.cshtml_) töröljük/kommentezzük ki a leírás és a kép propertykhez generált részeket.

    ```razor
    @*<div class="form-group">
        <label asp-for="Leirasclass="control-label"></label>
        <input asp-for="Leirasclass="form-control" />
        <span asp-validation-for="Leirasclass="text-danger"></span>
    </div>
    <div class="form-group">
        <label asp-for="Kepclass="control-label"></label>
        <input asp-for="Kepclass="form-control" />
        <span asp-validation-for="Kepclass="text-danger"></span>
    </div>*@
    ```

1. A másik (POST-os) `Create` műveletben a paraméter `Bind` attribútummal van ellátva. Ebben azt a lehető legszűkebb tulajdonsághalmazt érdemes megadni, ami a művelet elvégzéséhez kell. Ez amiatt kell, hogy vicces kedvű vagy támadó szándékú kolléga ne tudjon olyan HTTP POST kérést készíteni, amiben olyan tulajdonságokat is kitölt, amiket nem akarunk a HTTP kérésben érkező adatok alapján tölteni. Innen is érdemes kivenni a kép és leírás tulajdonságokat, hiszen ezeket már eleve meg se lehet adni az űrlapon.

    ```csharp
    public async Task<IActionResult> Create([Bind("Id,Nev,  NettoAr,Raktarkeszlet,AfaId,KategoriaId")] Termek termek)
    ```

1. Tegyünk töréspontot mindkét `Create` műveletre.

1. Debug módban indítva a projektet új termék felvételével próbáljuk ki, hogy a kategória legördülő menü jól működik-e. Kövessük végig a folyamatot debuggerrel ([**F10**] gombbal léptetve).

---

Az itt található oktatási segédanyagok a BMEVIAUBB04 tárgy hallgatóinak készültek. Az anyagok oly módú felhasználása, amely a tárgy oktatásához nem szorosan kapcsolódik, csak a szerző(k) és a forrás megjelölésével történhet.

Az anyagok a tárgy keretében oktatott kontextusban értelmezhetőek. Az anyagokért egyéb felhasználás esetén a szerző(k) felelősséget nem vállalnak.
