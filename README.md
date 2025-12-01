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
