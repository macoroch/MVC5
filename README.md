# MVC5 with Ninject (DI container)
This project is not about beauty but rather a reference on how to use MVC5 alltogether with NInject.

Solution
---------
It is divided in 3 parts: 
Domain: here we define the model
UI: is where all the MVC framework is working
UnitTests: for our solution

NuGet Packages
--------------
Install-Package Ninject -version 3.3.4 -projectname SportsStore. WebUi_MVC5
Install-Package Ninject -version 3.3.4 -projectname SportsStore.UnitTests

Install-Package Ninject.Web.Common -version 3.3.1 -projectname SportsStore. WebUi_MVC5
Install-Package Ninject.Web.Common -version 3.3.1 -projectname SportsStore.UnitTests

Install-Package Ninject.MVC5 -Version 3.3.0 -projectname SportsStore. WebUi_MVC5
Install-Package Ninject.MVC5 -Version 3.3.0 -projectname SportsStore.UnitTests

Install-Package Moq -version 4.10.0 -projectname SportsStore. WebUi_MVC5
Install-Package Moq -version 4.10.0 -projectname SportsStore.UnitTests

Install-Package Microsoft.Aspnet.Mvc -version 5.2.6 -projectname SportsStore.Domain
Install-Package Microsoft.Aspnet.Mvc -version 5.2.6 -projectname SportsStore.UnitTests

Install-Package EntityFramework -projectname SportsStore.WebUi_MVC5
Install-Package EntityFramework -projectname SportsStore.Domain

Install-Package -version 4.1.3 bootstrap –projectname SportsStore.WebUI


Ninject DI Container
--------------------
The version for MVC5 differs from the one in MVC3
It is implemented in the global.asax
In RegisterServices we define which is the implementation of our interface that we are using.

Creating a Fake data repository
-------------------------------
Before creating any database sometimes is good to have data to see if this can be rendered in our View.
In Domain create a interface called IproductRepositiory.cs
Implement this interface as HardCodedProductsRepository
In the global .asax tell inject to use the HardCodedProductsRepository implementation of Products.

kernel.Bind<IProductRepository>().To<HardCodedProductsRepository>();

When define the controller Products we request the Products NInject will create an instance for use in the constructor of the HardCodedProductsRepository and when we invoke Products it will get the implementation HardCodedProductsRepository.
 private IProductRepository repository;
 public ProductController(IProductRepository productRepository)
 {
            this.repository = productRepository;
 }

public ViewResult List()
{
	return View(repository.Products);
	...
}

Modification of the default route
---------------------------------
Instead of loading the Home controller we would like to load by default the Product controller and the List action.
With the version the routing mechanism is defined in global.asax and not in App_Start RouteConfig.
The reason is that we derive the global.asax from NinjectHttpApplication.

Database
--------
Download SQL Server Express> Open it> And take note of the server name. Could be something like this: JOHN\SQLEXPRESS
From the Server explorer create a new connection string to the database server that we just created. From here we can work as if we were in the SQL Management Studio.

CREATE TABLE Products
(
[ProductID] INT NOT NULL PRIMARY KEY IDENTITY,
[Name] NVARCHAR(100) NOT NULL,
[Description] NVARCHAR(500) NOT NULL,
[Category] NVARCHAR(50) NOT NULL,
[Price] DECIMAL(16, 2) NOT NULL
)

Populate the table with some values.


EF
-------------------------
Added the connection string on the Web.config under the WebUI:MVC5 project.
Create the DBContext class: EFDbContext

Use EF to retrieve the products
--------------------------------
Retrive the products from the database instead of the repostory:
Create in the Domain a class called EFProductRepository which implements IProductRepository.
Get the products from the context.
Change in the global.asax the implementation that we are using from IProductRepository:

kernel.Bind<IProductRepository>().To<EFProductRepository>();

Adding Pagination
-----------------
The default page is 1.
public class ProductController : Controller {
...
public int PageSize = 4;
...

public ViewResult List(int page = 1) {
return View(repository.Products
.OrderBy(p => p.ProductID)
.Skip((page - 1) * PageSize)
.Take(PageSize));
}

}

We can always test the pagination from the address bar: http://localhost:50530/Product/List?page=2

ViewModel
---------
In the appliaction we have entities that are an abstraction of the business and they are defined in the Domain.
Then we have classes that are needed for the application to pass data between the controller and the View. For instance the pagination information: total number of pages, current page, etc. For that we create the PageInfo class under models in the WebUi_MVC5.

The ViewModel class will contain all the information that needs to be provided exchanged between Controller and View. This will contain buiness data (Products) as well as for instance the PageInfo. So we group all this data under the ViewModel-

In this case we have defined the ProductsListViewModel class and thefore our model in the View needs to use the it.


Displaying the Pages on the UI
------------------------------
In order to do this we create an extension of HtmlHelper under the HtmlHelper folder called PagingHelpers.
Add this the HTML Helper Method Namespace to the Views/web.config File, where we have all the namespaces that we are referencing in the View with the keyword @using:
        <add namespace="SportsStore.WebUI_MVC5.HtmlHelpers"/>

We invoke the implemetation from the View.

Improving the URL
-----------------
If instead of http://localhost/?page=2 we want something like http://localhost/Page2 add the following route into the gloabl.asax (remeber that this is where we have defined the routes for our case as this class inherits from Nijectable)

Added Bootstrap
---------------
The  _Layout.cshtml and the List.cshtml were changed adding CSS from bootstrap

Added Product Summary (PartialView)
----------------------------------
There is PartialView under Views/Shared  called ProductSummary.cshtml. All partial views are stored in this folder.
This PartialView is used to display each product box.
So we have done changes in the List.cshtml to include the partial view.
