# taller-3.0
--Código completo para 3 funciones de Northwind
USE Northwind;
GO
DROP FUNCTION IF EXISTS dbo.fn_ProductosPorProveedor;
GO
CREATE FUNCTION dbo.fn_ProductosPorProveedor (
    @SupplierID INT
)
RETURNS TABLE
AS
RETURN (
    SELECT 
        ProductID,
        ProductName,
        UnitPrice,
        UnitsInStock,
        Discontinued
    FROM Products
    WHERE SupplierID = @SupplierID
);
GO
SELECT * FROM dbo.fn_ProductosPorProveedor(1);

DROP FUNCTION IF EXISTS dbo.fn_OrdenesPorCliente;
GO
CREATE FUNCTION dbo.fn_OrdenesPorCliente (
    @CustomerID NCHAR(5)
)
RETURNS TABLE
AS
RETURN (
    SELECT OrderID, OrderDate, ShipCountry, Freight
    FROM [Orders]
    WHERE CustomerID = @CustomerID
);
GO
-- Ejemplo de uso:
SELECT * FROM dbo.fn_OrdenesPorCliente('ALFKI');

DROP FUNCTION IF EXISTS dbo.fn_NombreFormateadoCliente;
GO
CREATE FUNCTION dbo.fn_NombreFormateadoCliente (
    @CustomerID NCHAR(5)
)
RETURNS NVARCHAR(200)
AS
BEGIN
    DECLARE @Nombre NVARCHAR(200);
    SELECT @Nombre = UPPER(Country + ': ' + CompanyName)
    FROM [Customers]
    WHERE CustomerID = @CustomerID;

    RETURN @Nombre;
END;
GO
SELECT dbo.fn_NombreFormateadoCliente('CACTU');

Función: dbo.fn_ProductosPorProveedor 


--Devuelve: Una tabla que contiene el ProductID, ProductName, UnitPrice, UnitsInStock y el estado Discontinued para el proveedor dado.

DROP FUNCTION IF EXISTS dbo.fn_ProductosPorProveedor;
GO
CREATE FUNCTION dbo.fn_ProductosPorProveedor (
    @SupplierID INT
)
RETURNS TABLE
AS
RETURN (
    SELECT 
        ProductID,
        ProductName,
        UnitPrice,
        UnitsInStock,
        Discontinued
    FROM Products
    WHERE SupplierID = @SupplierID
);
GO
SELECT * FROM dbo.fn_ProductosPorProveedor(1);





Función: dbo.fn_OrdenesPorCliente 
--Propósito: Devuelve una tabla con los pedidos realizados por un cliente específico.

DROP FUNCTION IF EXISTS dbo.fn_OrdenesPorCliente;
GO
CREATE FUNCTION dbo.fn_OrdenesPorCliente (
    @CustomerID NCHAR(5)
)
RETURNS TABLE
AS
RETURN (
    SELECT OrderID, OrderDate, ShipCountry, Freight
    FROM [Orders]
    WHERE CustomerID = @CustomerID
);
GO
-- Ejemplo de uso:
SELECT * FROM dbo.fn_OrdenesPorCliente('ALFKI');

Función: dbo.fn_NombreFormateadoCliente 
--Propósito: Devuelve una cadena de texto formateada que contiene el país y el nombre de la compañía de un cliente específico. 
--Devuelve: Una cadena NVARCHAR(200) en el formato 'PAÍS: NOMBREDECOMPAÑÍA' (por ejemplo, 'ALEMANIA: Alfreds Futterkiste').


DROP FUNCTION IF EXISTS dbo.fn_NombreFormateadoCliente;
GO
CREATE FUNCTION dbo.fn_NombreFormateadoCliente (
    @CustomerID NCHAR(5)
)
RETURNS NVARCHAR(200)
AS
BEGIN
    DECLARE @Nombre NVARCHAR(200);
    SELECT @Nombre = UPPER(Country + ': ' + CompanyName)
    FROM [Customers]
    WHERE CustomerID = @CustomerID;

    RETURN @Nombre;
END;
GO
-- Ejemplo de uso:
SELECT dbo.fn_NombreFormateadoCliente('CACTU');





--SOME

CREATE OR ALTER FUNCTION dbo.fn_ClientesResumenCompleto()
RETURNS TABLE
AS
RETURN
(
    SELECT 
        c.CustomerID,
        c.CompanyName,
        c.EmailAddress,
        c.Phone,
        COUNT(DISTINCT soh.SalesOrderID) AS TotalPedidos,
        SUM(sod.LineTotal) AS TotalComprado,
        MAX(soh.OrderDate) AS FechaUltimaCompra,
        CASE 
            WHEN MAX(soh.OrderDate) >= DATEADD(year, -1, GETDATE()) THEN CAST(1 AS BIT)
            ELSE CAST(0 AS BIT)
        END AS ClienteActivoUltimoAnio
    FROM SalesLT.Customer c
    LEFT JOIN SalesLT.SalesOrderHeader soh ON c.CustomerID = soh.CustomerID
    LEFT JOIN SalesLT.SalesOrderDetail sod ON soh.SalesOrderID = sod.SalesOrderID
    GROUP BY c.CustomerID, c.CompanyName, c.EmailAddress, c.Phone
);

Prueba:
SELECT TOP 100 * FROM dbo.fn_ClientesResumenCompleto() ORDER BY TotalComprado DESC;



--Detalle Completo de Productos con Margen y Fecha Última Venta

CREATE OR ALTER FUNCTION dbo.fn_ProductosDetalleCompleto()
RETURNS TABLE
AS
RETURN
(
    SELECT 
        p.ProductID,
        p.Name,
        p.ProductNumber,
        p.ListPrice,
        p.StandardCost,
        CASE 
            WHEN p.ListPrice = 0 THEN 0
            ELSE ROUND((p.ListPrice - p.StandardCost) * 100.0 / p.ListPrice, 2)
        END AS MargenPorcentaje,
        MAX(soh.OrderDate) AS FechaUltimaVenta
    FROM SalesLT.Product p
    LEFT JOIN SalesLT.SalesOrderDetail sod ON p.ProductID = sod.ProductID
    LEFT JOIN SalesLT.SalesOrderHeader soh ON sod.SalesOrderID = soh.SalesOrderID
    GROUP BY p.ProductID, p.Name, p.ProductNumber, p.ListPrice, p.StandardCost
);


Prueba:
SELECT TOP 100 * FROM dbo.fn_ProductosDetalleCompleto() ORDER BY MargenPorcentaje DESC;




--Ventas Detalladas con Información de Cliente y Producto

CREATE OR ALTER FUNCTION dbo.fn_VentasDetalleCompleto()
RETURNS TABLE
AS
RETURN
(
    SELECT 
        soh.SalesOrderID,
        soh.OrderDate,
        c.CustomerID,
        c.CompanyName,
        sod.ProductID,
        p.Name AS Producto,
        sod.OrderQty,
        sod.LineTotal
    FROM SalesLT.SalesOrderHeader soh
    INNER JOIN SalesLT.Customer c ON soh.CustomerID = c.CustomerID
    INNER JOIN SalesLT.SalesOrderDetail sod ON soh.SalesOrderID = sod.SalesOrderID
    INNER JOIN SalesLT.Product p ON sod.ProductID = p.ProductID
);


--Clientes con Pedidos No Cancelados (Resumen Detallado)

RETURNS TABLE
AS
RETURN
(
    SELECT 
        c.CustomerID,
        c.CompanyName,
        c.EmailAddress,
        soh.SalesOrderID,
        soh.OrderDate,
        DATEDIFF(DAY, soh.OrderDate, GETDATE()) AS DiasDesdeUltimaOrden,
        soh.Status,
        COUNT(sod.ProductID) AS CantidadProductos,
        SUM(sod.LineTotal) AS TotalPedido
    FROM SalesLT.Customer c
    INNER JOIN SalesLT.SalesOrderHeader soh ON c.CustomerID = soh.CustomerID
    INNER JOIN SalesLT.SalesOrderDetail sod ON soh.SalesOrderID = sod.SalesOrderID
    WHERE soh.Status <> 4 -- Estado 4 = Cancelado (ajustar si no aplica)
    GROUP BY c.CustomerID, c.CompanyName, c.EmailAddress, soh.SalesOrderID, soh.OrderDate, soh.Status
);


Prueba: 

SELECT TOP 100 * FROM dbo.fn_ClientesPedidosNoCancelados() ORDER BY DiasDesdeUltimaOrden ASC;





