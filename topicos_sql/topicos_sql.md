# Topicos SQL SERVER 

``` SQL
if not exists (select name from sys.databases WHERE name =N'miniDB')
begin
	create database miniDB
	COLLATE Latin1_General_100_CI_AS_SC_UTF8;
end
go


Use miniDB
go

-- Creacion de tablas
IF OBJECT_ID('clientes', 'U')IS NOT NULL Drop table clientes;

CREATE TABLE clientes(
	IdCliente INT not null,
	Nombre NVARCHAR (100),
	Edad INT,
	Ciudad NVARCHAR(100),
	constraint pk_clientes
	primary key(idcliente)
);
go
IF OBJECT_ID('productos', 'U')IS NOT NULL Drop table productos;

CREATE TABLE productos(
	idproducto INT primary key,
	NombreProducto NVARCHAR(200),
	Categoria NVARCHAR(200),
	Precio DECIMAL(12,2)
);
GO

/*
	========Insercion de registros en las tablas ===========
*/

INSERT INTO clientes
values (1, 'Ana Torres', 25, 'Ciudad de Mexico');

INSERT INTO clientes (IdCliente, Nombre, Edad, Ciudad)
values(2, 'Luis Perez', 34, 'Guadalajara');

INSERT INTO clientes (IdCliente, Edad, Nombre, Ciudad)
values(3, 15, 'Sofia Vaca', NULL)

INSERT INTO clientes (IdCliente, Nombre, Edad)
values(4, 'Natasha', 41)

INSERT INTO clientes (IdCliente, Nombre, Edad, Ciudad)
values (5, 'Pedro Lopez', 19, 'Chapuhuacan'),
	   (6, 'Laura Hernandez', 38, Null),
	   (7, 'Victor Trujillo', 25, 'Zacualtipan')



CREATE OR ALTER PROCEDURE sp_add_customer
 @Id INT, @Nombre NVARCHAR(100), @edad INT, @ciudad NVARCHAR(100)
 AS
 BEGIN
	INSERT INTO clientes (IdCliente, Nombre, Edad, Ciudad)
	VALUES (@Id, @Nombre, @edad, @ciudad );
END;
GO

EXEC sp_add_customer 8, 'Carlos Ruiz', 41, 'Monterrey';
EXEC sp_add_customer 9, 'Jose Angel Perez', 74, 'Salte si puedes';

Select * from clientes

SELECT COUNT(*) AS [Numero de Clientes]
From clientes;

-- Mostrar todos los clientes ordenados por edad de menor a mayor

Select UPPER(Nombre) as Cliente, edad, UPPER(ciudad)
from clientes
order by edad desc;

-- Listar los clientes que viven en Guadalajara
Select UPPER(Nombre) as Cliente, edad, UPPER(ciudad)
from clientes
Where Ciudad = 'Guadalajara';

-- Listar los clientes con una edad mayor o igual a 30
Select UPPER(Nombre) as Cliente, edad, UPPER(ciudad)
from clientes
Where edad >= 30;

-- Listar los clientes cuya ciudad sea nula
Select UPPER(Nombre) as Cliente, edad, UPPER(ciudad)
from clientes
Where Ciudad is null;

-- Remplazar en la consulta las ciudades nulas por la palabra DESCONOCIDA
-- Sin modificar los datos originales

SELECT UPPER (Nombre) as [Cliente], edad, ISNULL (UPPER(Ciudad), 'DESCONOCIDO') as 'Ciudad'
from clientes;

-- Selecciona los clientes que tengan edad entre 20 y 35 y que vivan en puebla o monterrey

SELECT UPPER (Nombre) as [Cliente], edad, ISNULL (UPPER(Ciudad), 'DESCONOCIDO') as 'Ciudad'
from clientes
Where edad between 20 and 35
	  AND
	  Ciudad IN ('Guadalajara','Monterrey')

select * from clientes;

update clientes
set Ciudad = 'Xochitlan'
where Ciudad is null;

update clientes
set Ciudad = 'Sin ciudad'
where Ciudad is null;

update clientes
set edad = 30
where IdCliente between 3 and 6;

update clientes
set Ciudad = 'Metropoli'
where Ciudad IN ('ciudad de Mexico', 'Guadalajara', 'Monterrey')

update clientes
set Nombre = 'Luis Perez',
	Edad = 27,
	Ciudad = 'Ciudad Gotica'
where IdCliente = 2;

update clientes
set Nombre = 'Cliente premium'
where Nombre like 'A%';

update clientes
set Nombre = 'silver costomer'
where Nombre like '%er%';

update clientes
set Edad = (Edad * 2)
where Edad >= 30 and Ciudad = 'Metropoli';


/*

========= Eliminar Datos ==========

*/

DELETE FROM clientes
WHERE edad between 25 and 30;

DELETE clientes
WHERE Nombre like '%r';

TRUNCATE TABLE clientes;

/*

======= Store procedure de update, delete


*/

-- Modifica los datos por id
Create or alter proc sp_update_customers
@id INT, @nombre nvarchar(100), 
@edad int, @ciudad nvarchar(100)
AS
BEGIN
	UPDATE clientes
	SET Nombre = @nombre,
		Edad = @edad,
		Ciudad = @ciudad
	Where IdCliente = @id;
END;

EXEC sp_update_customers 
7, 'Benito Cano', 24, 'Lima los pies';

select * from clientes

exec sp_update_customers 
@ciudad = 'Martinez de la Torre',
@edad = 56,
@id = 3,
@nombre = 'Toribio Trompudo';

-- Ejercicio completo donde se pueda insertar datos en una tabla
-- principal (encabezado) y una tabla detalle utilizando un sp.

CREATE TABLE ventas(
	Idventa INT IDENTITY (1,1) PRIMARY KEY,
	FechaVenta DATETIME NOT NULL DEFAULT GETDATE(),
	Cliente NVARCHAR(100) NOT NULL,
	Total DECIMAL (10,2) Null
);

-- Tabla detalle

CREATE TABLE DetalleVenta(
IdDetalle INT IDENTITY (1,1) PRIMARY KEY,
IdVenta INT NOT NULL,
Producto NVARCHAR(100) NOT NULL,
Cantidad INT NOT NULL,
Precio DECIMAL (10,2) NOT NULL,
CONSTRAINT pk_detalleVenta_venta
FOREIGN KEY (IdVenta)
REFERENCES Ventas(IdVenta)
);

-- Crear un tipo de tabla (table type)

-- Este tipo de tabla servira como estructura para enviar los detalles al sp

CREATE TYPE TipoDetalleVentas AS TABLE (
	Producto NVARCHAR(100),
	Cantidad INT,
	Precio DECIMAL(10,2)
);
go
-- Crear el store procedure
-- El sp insertara el encabezado y luego todos los detalles utilzando el tipo de tabla.

CREATE OR ALTER PROCEDURE InsertarVentaConDetalle
@Cliente NVARCHAR (100),
@Detalles TipoDetalleVentas READONLY
AS
BEGIN
	SET NOCOUNT ON;

	DECLARE @IdVenta INT;

	BEGIN TRY
		BEGIN TRANSACTION;

		-- Insertar en la tabla principal
		INSERT INTO ventas (Cliente)
		VALUES(@Cliente);

		-- Obtener el ID recien generado
		SET @IdVenta = SCOPE_IDENTITY();

		-- Insertar los detalles (Tabla Detalles)
		INSERT INTO DetalleVenta (IdVenta, Producto, Cantidad, precio)
		SELECT @IdVenta, Producto, Cantidad, Precio
		From @Detalles;

		-- Calcular el total de venta
		UPDATE Ventas
		SET Total = (SELECT SUM(Cantidad * Precio) FROM @Detalles)
		WHERE IdVenta = @IdVenta

		COMMIT TRANSACTION;
END TRY
BEGIN CATCH
		ROLLBACK TRANSACTION;
		THROW;
	END CATCH;
END;

-- Ejecutar el SP con datos de prueba


-- Declarar una variable tipo tabla
DECLARE @MisDetalles AS TipoDetalleVentas

-- Insertar productos en el Type Table
INSERT INTO @MisDetalles (Producto, Cantidad, Precio)
VALUES
('Laptop', 1, 15000),
('Mouse', 2, 300),
('Teclado', 1, 500),
('Pantalla', 5, 4500);

-- Ejecutar el SP
exec InsertarVentaConDetalle @Cliente='Uriel Edgar', @Detalles=@MisDetalles

Select * From ventas;
select * from DetalleVenta;

```

## Funciones Integradas (Built-in Functions)

| Función                               | Descripción                                      | Ejemplo                                            |
| ------------------------------------- | ------------------------------------------------ | -------------------------------------------------- |
| `LEN(cadena)`                         | Longitud del texto (sin contar espacios finales) | `LEN('SQL Server ') → 10`                          |
| `LTRIM(cadena)`                       | Elimina espacios a la izquierda                  | `'  Hola' → 'Hola'`                                |
| `RTRIM(cadena)`                       | Elimina espacios a la derecha                    | `'Hola  ' → 'Hola'`                                |
| `LOWER(cadena)`                       | Convierte a minúsculas                           | `'HOLA' → 'hola'`                                  |
| `UPPER(cadena)`                       | Convierte a mayúsculas                           | `'hola' → 'HOLA'`                                  |
| `SUBSTRING(cadena, inicio, longitud)` | Extrae una parte del texto                       | `SUBSTRING('SQLServer', 4, 6) → 'Server'`          |
| `LEFT(cadena, n)`                     | Devuelve los primeros *n* caracteres             | `LEFT('SQLServer', 3) → 'SQL'`                     |
| `RIGHT(cadena, n)`                    | Devuelve los últimos *n* caracteres              | `RIGHT('SQLServer', 6) → 'Server'`                 |
| `CHARINDEX(subcadena, cadena)`        | Devuelve la posición de una subcadena            | `CHARINDEX('S', 'SQL Server') → 1`                 |
| `REPLACE(cadena, buscar, reemplazo)`  | Reemplaza texto                                  | `REPLACE('SQL 2022', '2022', '2025') → 'SQL 2025'` |
| `REVERSE(cadena)`                     | Invierte el texto                                | `REVERSE('SQL') → 'LQS'`                           |
| `CONCAT(val1, val2, ...)`             | Une varios valores en una sola cadena            | `CONCAT('Cliente ', Nombre)`                       |
| `CONCAT_WS(sep, val1, val2, ...)`     | Une valores con un separador                     | `CONCAT_WS('-', 'MX', '001') → 'MX-001'`           |


```SQL
-- Funciones
SELECT
Nombre AS [Nombre Fuente],
LTRIM(UPPER(Nombre)) AS Mayusculas,
LOWER(Nombre) AS Minusculas,
LEN(Nombre) AS Longitud,
SUBSTRING(Nombre, 1,3) AS Prefijo,
LTRIM(Nombre) AS [Sin Espacios Izquierda],
CONCAT(Nombre, ' = ', Edad) AS [Nombre Edad],
UPPER(REPLACE(TRIM(Ciudad), 'chapulhuacan', 'Chapu')) AS [Ciudad Normal]
FROM clientes;

Select * from clientes

insert into clientes(IdCliente, Nombre, Edad, Ciudad)
VALUES (8, 'Luis Lopez', 45, 'Achichilco')

insert into clientes(IdCliente, Nombre, Edad, Ciudad)
VALUES (9, 'German Galindo', 32, 'Achichilco2')

insert into clientes(IdCliente, Nombre, Edad, Ciudad)
VALUES (10, 'Jael Porfirio', 19, 'Achichilco3')

insert into clientes(IdCliente, Nombre, Edad, Ciudad)
VALUES (11, 'Roberto Estrada', 19, 'chapulhuacan')

-- Crear una tabla a partir de una consulta
SELECT TOP 0
idCliente,
Nombre AS [Nombre Fuente],
LTRIM(UPPER(Nombre)) AS Mayusculas,
LOWER(Nombre) AS Minusculas,
LEN(Nombre) AS Longitud,
SUBSTRING(Nombre, 1,3) AS Prefijo,
LTRIM(Nombre) AS [Sin Espacios Izquierda],
CONCAT(Nombre, ' = ', Edad) AS [Nombre Edad],
UPPER(REPLACE(TRIM(Ciudad), 'chapulhuacan', 'Chapu')) AS [Ciudad Normal]
INTO stage_clientes
FROM clientes;

-- Agrega un constraint a la tabla
Alter table stage_clientes
add constraint pk_stage_clientes
primary key(idCliente)

SELECT * FROM stage_clientes

-- Insertar datos a partir de una consulta 
INSERT INTO stage_clientes (IdCliente, 
			[Nombre Fuente], 
			Mayusculas,	
			Minusculas, 
			Longitud, 
			Prefijo,
			[Sin Espacios Izquierda],
			[Nombre Edad], [Ciudad Normal])

SELECT
idCliente,
Nombre AS [Nombre Fuente],
LTRIM(UPPER(Nombre)) AS Mayusculas,
LOWER(Nombre) AS Minusculas,
LEN(Nombre) AS Longitud,
SUBSTRING(Nombre, 1,3) AS Prefijo,
LTRIM(Nombre) AS [Sin Espacios Izquierda],
CONCAT(Nombre, ' = ', Edad) AS [Nombre Edad],
UPPER(REPLACE(TRIM(Ciudad), 'chapulhuacan', 'Chapu')) AS [Ciudad Normal]
FROM clientes;

```

## Funciones de Fecha

| Función                               | Descripción                        |
| ------------------------------------- | ---------------------------------- |
| `GETDATE()`                           | Devuelve la fecha y hora actual    |
| `DATEADD(intervalo, cantidad, fecha)` | Suma o resta unidades de tiempo    |
| `DATEDIFF(intervalo, fecha1, fecha2)` | Calcula la diferencia entre fechas |
---
``` SQL
use NORTHWND
GO
SELECT
OrderDate,
GETDATE() AS [Fecha Actual],
DATEADD (Day, 10, OrderDate) AS [FechaMas10Dias],
DATEPART(Quarter, OrderDate) AS [Trimestre],
DATEPART(Month, OrderDate) AS [MesConNumero],
DATENAME(Month, OrderDate) AS [MesConNombre],
DATENAME(WEEKDAY, OrderDate) AS [NombreDia],
DATEDIFF(Day, OrderDate, GETDATE()) AS [Dias Transcurrido],
DATEDIFF(YEAR, OrderDate, GETDATE()) AS [AÑOS TRANSCURRIDOS],
DATEDIFF(YEAR, '2003-07-13', GETDATE()) AS [EdadJaen]
from Orders;
```

## Manejo de Valores Nulos

| Función                                 | Descripción                                 | Ejemplo                                                     |
| --------------------------------------- | ------------------------------------------- | ----------------------------------------------------------- |
| `ISNULL(expresión, valor)`              | Reemplaza `NULL` por un valor definido.     | `SELECT ISNULL(phone, 'Sin teléfono');`                     |
| `COALESCE(expresión1, expresión2, ...)` | Devuelve el primer valor no nulo.           | `SELECT COALESCE(email, secondary_email, 'No disponible');` |
| `NULLIF(expresión1, expresión2)`        | Devuelve `NULL` si los valores son iguales. | `SELECT NULLIF(salary, 0);`                                 |

```SQL

Use miniDB

CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    Email NVARCHAR(100),
    SecondaryEmail NVARCHAR(100),
    Phone NVARCHAR(20),
    Salary DECIMAL(10,2),
    Bonus DECIMAL(10,2)
);

INSERT INTO Employees (EmployeeID, FirstName, LastName, Email, SecondaryEmail, Phone, Salary, Bonus)
VALUES(1, 'Ana', 'Lopez', 'ana.lopez@empresa.com', NULL, '555-2345', 12000, 100),
	  (2, 'Carlos', 'Ramirez', NULL, 'c.ramirez@empresa.com', NULL, 9500, NULL),
      (3, 'Laura', 'Gomez', NULL, NULL, '555-8900', 0, 500),
      (4, 'Jorge', 'Diaz', 'jorge.diaz@empresa.com', NULL, NULL, 15000, 0);

-- Ejercicio1 - ISNULL
-- Mostrar el nombre completo del empleado junto con su numero de telefono,
-- Sino tiene telefono, mostrar el texto 'No disponible'

SELECT CONCAT(FirstName, ' ', LastName) AS [FULLNAME],
	   ISNULL(phone, 'No Disponible') AS [PHONE]
FROM Employees;

-- Ejercicio 2. Mostrar el nombre del empleado y su correo de contacto

SELECT CONCAT (FirstName, ' ', LastName) AS NombreCompleto, 
email, 
secondaryEmail,
COALESCE(email, secondaryEmail, 'Sin Correo') AS Correo_Contacto
from Employees;

-- Ejercicio 3, NULLIF
-- Mostrar el nomre del empleado, su salario y el resultado de
-- NULLIF(salary, 0), para detectar quien tiene salario cero.

SELECT CONCAT (FirstName, ' ', LastName) AS NombreCompleto,
salary,
NULLIF(salary, 0) AS [SalarioEvaluable]
From Employees;

-- Evita error de division por cero:

SELECT FirstName, Bonus, (Bonus/NULLIF(salary, 0)) AS Bonus_Salario
FROM Employees;
```

## Expresiones Condicionales Case

Permite crear condiciones dentro de una consulta

Sintaxis:

```SQL
CASE
WHEN condicion1 THEN resultado1
WHEN condicion2 THEN resultado2
ELSE resultado_por_defecto
END
```
