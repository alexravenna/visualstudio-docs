---
title: LINQ to SQL O/R Designer overview
description: Explore LINQ to SQL tools in Visual Studio for object-relational mapping, including the Object Relational Designer (O/R Designer).
ms.date: 12/03/2024
ms.topic: overview
author: ghogen
ms.author: ghogen
manager: mijacobs
ms.subservice: data-tools
---

# LINQ to SQL tools in Visual Studio

LINQ to SQL was the first object-relational mapping technology released by Microsoft. It works well in basic scenarios and continues to be supported in Visual Studio, but it's no longer under active development. Use LINQ to SQL when maintaining a legacy application that's already using it, or in simple applications that use SQL Server and do not require multi-table mapping. In general, new applications should use the Entity Framework when an object-relational mapping layer is required.

## Install the LINQ to SQL tools

In Visual Studio, you create LINQ to SQL classes that represent SQL tables by using the **Object Relational Designer** (**O/R Designer**). The O/R designer is the UI for editing `.dbml` files. Editing `.dbml` files with a designer surface requires the LINQ to SQL tools which are not installed by default as part of any of the workloads of Visual Studio.

To install the LINQ to SQL tools, start the Visual Studio installer, choose **Modify**, then select the **Individual Components** tab, and then select **LINQ to SQL tools** under the **Code Tools** category.

## What is the O/R Designer

The **O/R Designer** has two distinct areas on its design surface: the entities pane on the left, and the methods pane on the right. The entities pane is the main design surface that displays the entity classes, associations, and inheritance hierarchies. The methods pane is the design surface that displays the <xref:System.Data.Linq.DataContext> methods that are mapped to stored procedures and functions.

The **O/R Designer** provides a visual design surface for creating [LINQ to SQL](/dotnet/framework/data/adonet/sql/linq/index) entity classes and associations (relationships) that are based on objects in a database. In other words, the **O/R Designer** creates an object model in an application that maps to objects in a database. It also generates a strongly-typed <xref:System.Data.Linq.DataContext> that sends and receives data between the entity classes and the database. The **O/R Designer** also provides functionality to map stored procedures and functions to <xref:System.Data.Linq.DataContext> methods for returning data and populating entity classes. Finally, the **O/R Designer** provides the ability to design inheritance relationships between entity classes.

## Open the O/R designer

To add a LINQ to SQL entity model to your project, choose **Project** > **Add New Item**, and then select **LINQ to SQL Classes** from the list of project items:

![Screenshot showing LINQ to SQL classes.](../data-tools/media/raddata-linq-to-sql-classes.png)

Visual Studio creates a `.dbml` file and adds it to your solution. This is the XML mapping file and its related code files.

![Screenshot showing LINQ to SQL classes in Solution Explorer.](../data-tools/media/raddata-linq-to-sql-classes-in-solution-explorer.png)

When you select the `.dbml` file, Visual Studio shows the **O/R Designer** surface that enables you to visually create the model. The following illustration shows the designer after the Northwind `Customers` and `Orders` tables have been dragged from **Server Explorer**. Note the relationship between the tables.

![Screenshot showing LINQ to SQL Designer.](../data-tools/media/raddata-linq-to-sql-designer.png)

> [!IMPORTANT]
> The **O/R Designer** is a simple object relational mapper because it supports only 1:1 mapping relationships. In other words, an entity class can have only a 1:1 mapping relationship with a database table or view. Complex mapping, such as mapping an entity class to a joined table, is not supported; use the Entity Framework for complex mapping. Additionally, the designer is a one-way code generator. This means that only changes that you make to the designer surface are reflected in the code file. Manual changes to the code file are not reflected in the **O/R Designer**. Any changes that you make manually in the code file are overwritten when the designer is saved and code is regenerated. For information about how to add user code and extend the classes generated by the **O/R Designer**, see [How to: Extend code generated by the O/R Designer](../data-tools/how-to-extend-code-generated-by-the-o-r-designer.md).

## Create and configure the DataContext

After you add a **LINQ to SQL Classes** item to a project and open the **O/R Designer**, the empty design surface represents an empty <xref:System.Data.Linq.DataContext> ready to be configured. the <xref:System.Data.Linq.DataContext> is configured with connection information provided by the first item that is dragged onto the design surface. Therefore, the <xref:System.Data.Linq.DataContext> is configured by using connection information from the first item dropped onto the design surface. For more information about the <xref:System.Data.Linq.DataContext> class see, [DataContext methods (O/R Designer)](../data-tools/datacontext-methods-o-r-designer.md).

## Create entity classes that map to database tables and views

You can create entity classes mapped to tables and views by dragging database tables and views from **Server Explorer** or **Database Explorer** onto the **O/R Designer**. As indicated in the previous section, the <xref:System.Data.Linq.DataContext> is configured with connection information provided by the first item that is dragged onto the design surface. If a subsequent item that uses a different connection is added to the **O/R Designer**, you can change the connection for the <xref:System.Data.Linq.DataContext>. For more information, see [How to: Create LINQ to SQL classes mapped to tables and views (O/R Designer)](../data-tools/how-to-create-linq-to-sql-classes-mapped-to-tables-and-views-o-r-designer.md).

## Create DataContext methods that call stored procedures and functions

You can create <xref:System.Data.Linq.DataContext> methods that call (are mapped to) stored procedures and functions by dragging them from **Server Explorer** or **Database Explorer** onto the **O/R Designer**. Stored procedures and functions are added to the **O/R Designer** as methods of the <xref:System.Data.Linq.DataContext>.

> [!NOTE]
> When you drag stored procedures and functions from **Server Explorer** or **Database Explorer** onto the **O/R Designer**, the return type of the generated <xref:System.Data.Linq.DataContext> method differs depending on where you drop the item. For more information, see [DataContext methods (O/R Designer)](../data-tools/datacontext-methods-o-r-designer.md).

## Configure a DataContext to use stored procedures to save data between entity classes and a database

As stated earlier, you can create <xref:System.Data.Linq.DataContext> methods that call stored procedures and functions. Additionally, you can also assign stored procedures that are used for the default LINQ to SQL run-time behavior, which performs inserts, updates, and deletes. For more information, see [How to: Assign stored procedures to perform updates, inserts, and deletes (O/R Designer)](../data-tools/how-to-assign-stored-procedures-to-perform-updates-inserts-and-deletes-o-r-designer.md).

## Inheritance and the O/R designer

Like other objects, LINQ to SQL classes can use inheritance and be derived from other classes. In a database, inheritance relationships are created in several ways. The **O/R Designer** supports the concept of single-table inheritance as it is often implemented in relational systems. For more information, see [How to: Configure inheritance by using the O/R Designer](../data-tools/how-to-configure-inheritance-by-using-the-o-r-designer.md).

## LINQ to SQL queries

The entity classes created by the **O/R Designer** are designed for use with [Language Integrated Query (LINQ)](/dotnet/csharp/linq/). For more information, see [How to: Query for information](/dotnet/framework/data/adonet/sql/linq/how-to-query-for-information).

## Separate the generated DataContext and entity class code into different namespaces

The **O/R Designer** provides the **Context Namespace** and **Entity Namespace** properties on the <xref:System.Data.Linq.DataContext>. These properties determine what namespace the <xref:System.Data.Linq.DataContext> and entity class code is generated into. By default, these properties are empty and the <xref:System.Data.Linq.DataContext> and entity classes are generated into the application's namespace. To generate the code into a namespace other than the application's namespace, enter a value into the **Context Namespace** and/or **Entity Namespace** properties.

## Reference content

- <xref:System.Linq>
- <xref:System.Data.Linq>

## See also

- [LINQ to SQL (.NET Framework)](/dotnet/framework/data/adonet/sql/linq/index)
- [Frequently asked questions (.NET Framework)](/dotnet/framework/data/adonet/sql/linq/frequently-asked-questions)
