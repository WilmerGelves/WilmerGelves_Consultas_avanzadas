
### CONSULTAS  SIMPLES 

```sql
-- 1. Seleccionar todos los productos con precio mayor a $50.
SELECT nombre,precio 
FROM productos
WHERE precio > 50;

-- 2. Consultar clientes registrados en una ciudad específica.
SELECT c.nombre 
FROM clientes c
JOIN ubicacion_cliente uc ON c.id = uc.id_cliente
JOIN ciudades cy ON uc.id_ciudad = cy.id
WHERE cy.ciudad = 'Buenos Aires';

-- 3. Mostrar empleados contratados en los últimos 2 años.
SELECT id, nombre, fecha_contratacion 
FROM empleados
WHERE fecha_contratacion >= CURDATE() - INTERVAL 2 YEAR;

-- 4. Seleccionar proveedores que suministran más de 5 productos.
SELECT p.nombre,COUNT(po.id_proveedor) AS productos_suministrados
FROM proveedores p 
JOIN productos po ON p.id = po.id_proveedor
GROUP BY  p.nombre
HAVING productos_suministrados > 5;

-- 5. Listar clientes que no tienen dirección registrada en UbicacionCliente.
SELECT c.nombre
FROM clientes c
LEFT JOIN ubicacion_cliente uc ON c.id = uc.id_cliente
WHERE uc.id_cliente IS NULL;

-- 6. Calcular el total de ventas por cada cliente.
SELECT c.nombre,SUM(p.total) AS total_por_cliente
FROM clientes c 
JOIN pedidos p ON c.id = p.id_cliente
GROUP BY c.id;

-- 7. Mostrar el salario promedio de los empleados.
SELECT AVG(salario) AS salario_promedio
FROM puestos;

-- 8. Consultar el tipo de productos disponibles en TiposProductos.
SELECT tipo,descripcion 
FROM tipos_de_productos;

-- 9. Seleccionar los 3 productos más caros.
SELECT nombre,precio 
FROM productos
ORDER BY precio DESC
LIMIT 3;

-- 10. Consultar el cliente con el mayor número de pedidos.
SELECT c.nombre,COUNT(p.id) AS pedidos_x_cliente
FROM clientes c 
JOIN pedidos p ON c.id = p.id_cliente
GROUP BY c.id
LIMIT 1;

```

### CONSULTAS  MULTITABLA 

```sql
-- 1.Listar todos los pedidos y el cliente asociado.
SELECT p.id AS id_pedido ,p.fecha,p.total,c.nombre
FROM pedidos p
JOIN clientes c ON p.id_cliente = c.id
WHERE p.id IN (SELECT id FROM clientes c
);
-- 2.Mostrar la ubicación de cada cliente en sus pedidos.
SELECT c.nombre AS nombre_de_cliente,p.id AS	id_de_pedido,uc.direccion,cy.ciudad,dp.departamento,pa.pais
FROM clientes c
JOIN pedidos p ON c.id = p.id_cliente
JOIN ubicacion_cliente uc ON p.id_cliente = uc.id_cliente
JOIN ciudades cy ON uc.id_ciudad = cy.id
JOIN departamentos dp ON cy.id_departamento = dp.id
JOIN paises pa ON dp.id_pais = pa.id
ORDER BY p.id ASC ;
-- 3.Listar productos junto con el proveedor y tipo de producto.
SELECT p.nombre,tp.tipo,pr.nombre
FROM productos p
JOIN tipos_de_productos tp ON p.tipo_id = tp.id
JOIN proveedores pr ON p.id_proveedor = pr.id;

-- 4.Consultar todos los empleados que gestionan pedidos de clientes en una ciudad específica.
SELECT e.nombre AS Nombre_empleado
FROM empleados e
JOIN pedidos p ON e.id = p.id_empleado  
JOIN clientes c ON p.id_cliente = c.id
JOIN ubicacion_cliente uc ON c.id = uc.id_cliente
JOIN ciudades cy ON uc.id_ciudad = cy.id
WHERE cy.ciudad = 'Buenos Aires';

-- 5.Consultar los 5 productos más vendidos.
SELECT p.nombre,COUNT(dp.id_producto) AS cantidad_X_producto
FROM productos p
JOIN detalles_pedido dp ON p.id = dp.id_producto
GROUP BY p.nombre
LIMIT 5;

-- 6.Obtener la cantidad total de pedidos por cliente y ciudad.
SELECT c.nombre,COUNT(p.id_cliente) AS pedidos_x_cliente,cy.ciudad
FROM clientes c
JOIN pedidos p ON c.id = p.id_cliente
JOIN ubicacion_cliente uc ON c.id = uc.id_cliente
JOIN ciudades cy ON uc.id_ciudad = cy.id
GROUP BY c.nombre,cy.ciudad;

-- 7.Listar clientes y proveedores en la misma ciudad.
SELECT c.nombre AS Nombre_cliente,pr.nombre AS Nombre_proveedor
FROM clientes c
JOIN ubicacion_cliente uc ON c.id = uc.id_cliente
JOIN contacto_proveedores cp ON uc.id_ciudad = cp.id_ciudad
JOIN proveedores pr ON cp.id_proveedor = pr.id;

-- 8.Mostrar el total de ventas agrupado por tipo de producto.
SELECT SUM(p.total) AS Total_ventas,pr.nombre AS Nombre_producto
FROM pedidos p
JOIN detalles_pedido dp ON p.id = dp.id_pedido
JOIN productos pr ON dp.id_producto = pr.id
GROUP BY pr.nombre;

-- 9.Listar empleados que gestionan pedidos de productos de un proveedor específico.
SELECT e.nombre AS Nombre_empleado,pv.nombre 
FROM empleados e
JOIN pedidos p ON e.id = p.id_empleado
JOIN detalles_pedido dp ON p.id = dp.id_pedido
JOIN productos pr ON dp.id_producto = pr.id
JOIN proveedores pv ON pr.id_proveedor = pv.id;

-- 10.Obtener el ingreso total de cada proveedor a partir de los productos vendidos.
SELECT pr.nombre AS Nombre_proveedor,SUM(p.precio * dp.cantidad) AS Total_de_venta
FROM proveedores pr
JOIN productos p ON pr.id = p.id_proveedor
JOIN detalles_pedido dp ON p.id = dp.id_producto
JOIN pedidos pe ON dp.id_pedido = pe.id
GROUP BY pr.id;

```

### JOINS

```sql
-- 1.Obtener la lista de todos los pedidos con los nombres de clientes usando INNER JOIN.
SELECT p.id AS id_pedido,c.nombre AS nombre_cliente
FROM pedidos p
JOIN clientes c ON p.id_cliente = c.id
ORDER BY p.id;
-- 2.Listar los productos y proveedores que los suministran con INNER JOIN.
SELECT p.nombre,pr.nombre AS id_proveedor
FROM productos p
JOIN proveedores pr ON p.id_proveedor = pr.id;
-- 3.Mostrar los pedidos y las ubicaciones de los clientes con LEFT JOIN.
SELECT  p.id,c.nombre,uc.direccion,cy.ciudad,d.departamento,pa.pais
FROM pedidos p
LEFT JOIN clientes c ON p.id_cliente = c.id
LEFT JOIN ubicacion_cliente uc ON c.id = uc.id_cliente
LEFT JOIN ciudades cy ON uc.id_ciudad = cy.id
LEFT JOIN departamentos d ON cy.id_departamento = d.id
LEFT JOIN paises pa ON d.id_pais = pa.id
ORDER BY  p.id;

-- 4.Consultar los empleados que han registrado pedidos, incluyendo empleados sin pedidos (LEFT JOIN).
SELECT e.nombre,COUNT(p.id_empleado) AS pedidos_por_empleados
FROM empleados e
LEFT JOIN pedidos p ON e.id = p.id_empleado
GROUP BY e.id ;

-- 5.Obtener el tipo de producto y los productos asociados con INNER JOIN.
SELECT t.tipo AS Tipo_de_producto,p.nombre AS Nombre_producto
FROM tipos_de_productos t
JOIN productos p ON t.id = p.tipo_id;

-- 6.Listar todos los clientes y el número de pedidos realizados con COUNT y GROUP BY.
SELECT c.nombre AS Nombre_del_cliente,COUNT(p.id_cliente) AS Cantidad_de_pedidos
FROM clientes c 
JOIN pedidos p ON c.id = p.id_cliente
GROUP BY c.id;

-- 7.Combinar Pedidos y Empleados para mostrar qué empleados gestionaron pedidos específicos.
SELECT p.id AS Pedido, e.nombre As Empleado_responsable
FROM pedidos p
LEFT JOIN empleados e ON p.id_empleado = e.id
ORDER BY p.id; 

-- 8.Mostrar productos que no han sido pedidos (RIGHT JOIN).
SELECT COUNT(dp.id_producto) AS Veces_pedido_el_producto, p.nombre AS Nombre_de_producto
FROM detalles_pedido dp 
RIGHT JOIN productos p  ON dp.id_producto = p.id
GROUP BY p.id
HAVING Veces_pedido_el_producto < 1;

-- 9.Mostrar el total de pedidos y ubicación de clientes usando múltiples JOIN.
SELECT  c.nombre,COUNT(c.id) AS pedidos_x_cliente,uc.direccion,cy.ciudad,d.departamento,pa.pais
FROM pedidos p
JOIN clientes c ON p.id_cliente = c.id
JOIN ubicacion_cliente uc ON c.id = uc.id_cliente
JOIN ciudades cy ON uc.id_ciudad = cy.id
JOIN departamentos d ON cy.id_departamento = d.id
JOIN paises pa ON d.id_pais = pa.id
GROUP BY c.id,uc.id;
-- 10.Unir Proveedores, Productos, y TiposProductos para un listado completo de inventario.
SELECT pr.nombre AS Proveedor,p.nombre AS Nombre_producto,p.stock As Stock,tp.tipo AS Tipo_de_producto
FROM proveedores pr 
JOIN productos p ON pr.id = p.id_proveedor
JOIN tipos_de_productos tp ON p.tipo_id = tp.id;

```

### SUBCONSULTAS

```sql
-- 1.Consultar el producto más caro en cada categoría.
SELECT DISTINCT c.categoria AS Categoria,tp.tipo AS Tipo, p.nombre AS Nombre_producto,p.precio AS Precio_prodcuto
FROM categorias c 
JOIN tipos_de_productos tp ON c.id = tp.id_categoria
JOIN productos p ON tp.id = p.tipo_id
WHERE p.precio = (
	SELECT MAX(p2.precio)
    FROM productos p2
    WHERE p2.tipo_id = p.tipo_id
);
-- 2.Encontrar el cliente con mayor total en pedidos.
SELECT c.nombre, SUM(p.total) AS Monto_total
FROM clientes c
JOIN pedidos p ON c.id = p.id_cliente
GROUP BY c.id
HAVING SUM(p.total) = (
    SELECT MAX(Monto_total)
    FROM (
        SELECT SUM(total) AS Monto_total
        FROM pedidos
        GROUP BY id_cliente
    ) AS confirmacion
);

-- 3.Listar empleados que ganan más que el salario promedio.
SELECT e.nombre AS Nombre_empleado,p.salario AS Salario
FROM empleados e
JOIN puestos p ON e.id_puesto = p.id
WHERE p.salario > (
    SELECT AVG(salario)
    FROM puestos
);

-- 4.Consultar productos que han sido pedidos más de 5 veces.
SELECT p.nombre, COUNT(dp.id_producto) AS Veces_X_pedido
FROM productos p
JOIN detalles_pedido dp ON p.id = dp.id_producto
GROUP BY p.id
HAVING Veces_X_pedido > 5;

-- 5.Listar pedidos cuyo total es mayor al promedio de todos los pedidos.
SELECT p.id AS Id_de_pedido,p.total AS Monto
FROM pedidos p
WHERE p.total > (
	SELECT AVG(total)
    FROM pedidos
);
-- 6.Seleccionar los 3 proveedores con más productos.
SELECT pr.nombre AS Proveerdor, COUNT(p.id_proveedor) AS Cantidad_productos
FROM proveedores pr
JOIN productos p ON pr.id = p.id_proveedor
GROUP BY pr.id
ORDER BY Cantidad_productos DESC
LIMIT 3;

-- 7.Consultar productos con precio superior al promedio en su tipo.
SELECT p.nombre AS Nombre_producto,p.precio AS Precio_del_producto,tp.tipo AS Tipo_de_producto
FROM productos p 
JOIN tipos_de_productos tp ON p.tipo_id = tp.id
GROUP BY p.id
HAVING Precio_del_producto > (
	SELECT tp.tipo, AVG(p.precio) 
	FROM productos p
	JOIN tipos_de_productos tp ON p.tipo_id = tp.id
	GROUP BY tp.id
);
-- 8.Mostrar clientes que han realizado más pedidos que la media.
SELECT c.nombre AS Nombre_cliente,COUNT(p.id_cliente) AS Pedidos_x_cliente
FROM clientes c
JOIN pedidos p ON c.id = p.id_cliente
GROUP BY c.id
HAVING Pedidos_x_cliente > (
	SELECT (COUNT(id))/2 FROM pedidos
);

-- 9.Encontrar productos cuyo precio es mayor que el promedio de todos los productos.
SELECT p.nombre AS Nombre_producto,p.precio AS Precio_del_producto
FROM productos p 
GROUP BY p.id
HAVING Precio_del_producto > 
    (SELECT SUM(p2.precio) 
	FROM productos p2)
    /(SELECT COUNT(p3.id)
	FROM productos p3
);
-- 10.Mostrar empleados cuyo salario es menor al promedio del departamento.
SELECT e.nombre AS Nombre_Empleado,p.salario AS Salario
FROM empleados e
JOIN puestos p ON e.id_puesto = p.id
WHERE Salario < (
	SELECT AVG(salario)
    FROM puestos
);

```

### PROCEDIMIENTOS  ALMACENADOS

```sql
-- 1.Crear un procedimiento para actualizar el precio de todos los productos de un proveedor.
DELIMITER $$
CREATE PROCEDURE actualizar_precio(
    IN proveedor INT,
    IN nuevo_precio DECIMAL(10,2)
)
BEGIN
UPDATE productos 
SET precio = nuevo_precio
WHERE id_proveedor = proveedor;
END $$
DELIMITER ;

CALL actualizar_precio('2','10000');

-- 2.Un procedimiento que devuelva la dirección de un cliente por ID.
DELIMITER $$
CREATE PROCEDURE direccion_cliente(
	IN ID_cliente VARCHAR(12)
)
BEGIN 
SELECT c.nombre AS Nombre_del_cliente, uc.direccion AS Direccion,cy.ciudad,dp.departamento,p.pais
FROM clientes c
JOIN ubicacion_cliente uc ON c.id = uc.id_cliente
JOIN ciudades cy ON uc.id_ciudad = cy.id
JOIN departamentos dp ON cy.id_departamento = dp.id
JOIN paises p ON dp.id_pais = p.id
WHERE c.num_identidad = ID_cliente;
END $$
DELIMITER ;
CALL direccion_cliente('456789012345');
-- 3.Crear un procedimiento que registre un pedido nuevo y sus detalles.
DELIMITER $$
CREATE PROCEDURE pedido_y_detalle(
	IN cliente INT,
    IN fecha_p DATE,
    IN empleado INT,
    IN monto_p DECIMAL(10,2),
    IN producto INT,
    IN unidades INT, 
    IN valor DECIMAL(10,2)
)
BEGIN 
DECLARE id_de_pedido INT ;
DECLARE mensaje VARCHAR(100);

INSERT INTO pedidos (id_cliente, fecha, id_empleado, total)
VALUES (cliente, fecha_p, empleado, monto_p);

SELECT LAST_INSERT_ID();    
SET id_de_pedido = LAST_INSERT_ID();

INSERT INTO detalles_pedido(id_pedido,id_producto,cantidad,precio)
VALUES (id_de_pedido,producto,unidades,valor);
END $$

DELIMITER ;

CALL pedido_y_detalle('2','2023-09-20','1','14000','24','1','14000');


-- 4.Un procedimiento para calcular el total de ventas de un cliente.
DELIMITER $$ 
CREATE PROCEDURE total_ventas_cliente(
	IN cliente VARCHAR(12)
)
BEGIN 
SELECT c.nombre AS Nombre_del_cliente,
	   SUM(p.total) AS Compra
FROM clientes c
JOIN pedidos p ON c.id = p.id_cliente
WHERE c.num_identidad = cliente
GROUP BY p.id_cliente;
END $$
DELIMITER ;

CALL total_ventas_cliente('234567890123');

-- 5.Crear un procedimiento para obtener los empleados por puesto.
DELIMITER $$
CREATE PROCEDURE empleados_x_puesto(
	IN puesto VARCHAR(100)
)
BEGIN 
SELECT e.nombre AS Nombre_empleado, p.puesto AS Puesto
FROM empleados e 
JOIN puestos p ON e.id_puesto = p.id
WHERE p.puesto = puesto
ORDER BY e.id ;
END $$
DELIMITER ;

CALL empleados_x_puesto('Encargado');

-- 6.Un procedimiento que actualice el salario de empleados por puesto.
DELIMITER $$

CREATE PROCEDURE actualizar_salario(
	IN puesto INT,
    IN Nuevo_salario DECIMAL(10,2)
)
BEGIN 
DECLARE mensaje VARCHAR(100);

UPDATE puestos 
SET salario = Nuevo_salario
WHERE id = puesto;
END $$

DELIMITER ;

CALL actualizar_salario('1','2000');

-- 7.Crear un procedimiento que liste los pedidos entre dos fechas.
DELIMITER $$

CREATE PROCEDURE ListarPedidosEntreFechas(
    IN fecha_inicio DATE,
    IN fecha_fin DATE
)
BEGIN
    SELECT c.nombre AS Nombre_del_cliente,p.id AS Id_pedido,p.fecha AS Fecha,p.total AS Total
    FROM clientes c
    JOIN pedidos p ON c.id = p.id_cliente
    WHERE fecha BETWEEN fecha_inicio AND fecha_fin;
END $$

DELIMITER ;

CALL ListarPedidosEntreFechas('2024-01-01', '2024-10-10');
-- 8.Un procedimiento para aplicar un descuento a productos de una categoría.
DELIMITER $$

CREATE PROCEDURE AplicarDescuentoCategoria(
    IN id_categoria INT,         
    IN porcentaje_descuento DECIMAL(5,2)  
)
BEGIN
    
    UPDATE productos p
    JOIN tipos_de_productos tdp ON p.tipo_id = tdp.id
    SET p.precio = p.precio - (p.precio * porcentaje_descuento / 100)
    WHERE tdp.id_categoria = id_categoria;

    -- Retornar la cantidad de productos actualizados
    SELECT ROW_COUNT() AS productos_actualizados;
END $$

DELIMITER ;

CALL AplicarDescuentoCategoria(1, 20.00);

-- 9.Crear un procedimiento que liste todos los proveedores de un tipo de producto.
DELIMITER $$

CREATE PROCEDURE listar_proveedores_X_tipo(
	IN tipo INT  
)
BEGIN 

SELECT  DISTINCT p.nombre AS Nombre_de_proveedor
FROM proveedores p
JOIN productos pr ON p.id = pr.id_proveedor
JOIN tipos_de_productos tp ON pr.tipo_id = tp.id
WHERE tp.id = tipo;
END $$

DELIMITER ;

CALL listar_proveedores_X_tipo('3');-- Ingresar el id del tipo de producto. 

-- 10.Un procedimiento que devuelva el pedido de mayor valor.
DELIMITER $$

CREATE PROCEDURE pedido_con_mayor_valor(
)
BEGIN 
SELECT id AS Pedido,MAX(total) AS Valor_del_pedido 
FROM pedidos
GROUP BY id
ORDER BY total DESC
LIMIT 1;
END $$
DELIMITER ;

CALL pedido_con_mayor_valor();

```

### FUNCIONES  DEFINIDAS POR EL USUARIO

```sql
-- 1.Crear una función que reciba una fecha y devuelva los días transcurridos.
DELIMITER $$

CREATE FUNCTION dias_transcurridos(fecha DATE)
RETURNS INT
DETERMINISTIC
BEGIN
    RETURN DATEDIFF(CURDATE(), fecha); 
END $$

DELIMITER ;

SELECT dias_transcurridos('2024-11-01') AS Diferencia_en_dias;

-- 2.Crear una función para calcular el total con impuesto de un monto.
DELIMITER $$

CREATE FUNCTION calcular_total_con_impuesto(monto DECIMAL(10,2), tasa_impuesto DECIMAL(5,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE total_con_impuesto DECIMAL(10,2);
    SET total_con_impuesto = monto + (monto * tasa_impuesto / 100);
    RETURN total_con_impuesto;
END $$

DELIMITER ;

SELECT calcular_total_con_impuesto(1000, 16) AS Total_con_impuesto;

-- 3.Una función que devuelva el total de pedidos de un cliente específico.
DELIMITER $$

CREATE FUNCTION total_pedidos_cliente(id_cliente INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(10,2);
    SELECT SUM(total) INTO total
    FROM pedidos
    WHERE id_cliente = id_cliente;
    RETURN total;
END $$

DELIMITER ;

SELECT total_pedidos_cliente(1) AS Total_pedido;

-- 4.Crear una función para aplicar un descuento a un producto.
DELIMITER $$

CREATE FUNCTION aplicar_descuento_producto(id_producto INT, descuento DECIMAL(5,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE nuevo_precio DECIMAL(10,2);
    SELECT precio - (precio * descuento / 100) INTO nuevo_precio
    FROM productos
    WHERE id = id_producto;
    RETURN nuevo_precio;
END $$

DELIMITER ;

SELECT aplicar_descuento_producto(1, 10) AS Nuevo_precio;

-- 5.Una función que indique si un cliente tiene dirección registrada.
DELIMITER $$

CREATE FUNCTION tiene_direccion_cliente(id_cliente INT)
RETURNS BOOLEAN
DETERMINISTIC
BEGIN
    DECLARE tiene_direccion BOOLEAN;
    

    SELECT CASE 
        WHEN COUNT(*) > 0 THEN TRUE 
        ELSE FALSE 
    END INTO tiene_direccion
    FROM ubicacion_cliente
    WHERE id_cliente = id_cliente;


    RETURN tiene_direccion;
END $$

DELIMITER ;

SELECT tiene_direccion_cliente(1) AS tiene_direccion_cliente_1;

-- 6.Crear una función que devuelva el salario anual de un empleado.
DELIMITER $$

CREATE FUNCTION salario_anual_empleado(id_empleado INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE salario_anual DECIMAL(10,2);
    SELECT salario * 12 INTO salario_anual
    FROM puestos
    JOIN empleados ON puestos.id = empleados.id_puesto
    WHERE empleados.id = id_empleado;
    RETURN salario_anual;
END $$

DELIMITER ;

SELECT salario_anual_empleado(1) AS salario_anual;

-- 7.Una función para calcular el total de ventas de un tipo de producto.
DELIMITER $$

CREATE FUNCTION total_ventas_tipo_producto(id_tipo_producto INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE total_ventas DECIMAL(10,2);
    SELECT SUM(dp.cantidad * p.precio) INTO total_ventas
    FROM detalles_pedido dp
    JOIN productos p ON dp.id_producto = p.id
    WHERE p.tipo_id = id_tipo_producto;
    RETURN total_ventas;
END $$

DELIMITER ;

SELECT total_ventas_tipo_producto(1) AS Total_x_tipo_Producto;

-- 8.Crear una función para devolver el nombre de un cliente por ID.
DELIMITER $$

CREATE FUNCTION nombre_cliente_por_id(id_cliente INT)
RETURNS VARCHAR(100)
DETERMINISTIC
BEGIN
    DECLARE nombre_cliente VARCHAR(100);
    SELECT nombre INTO nombre_cliente
    FROM clientes
    WHERE id = id_cliente;
    RETURN nombre_cliente;
END $$

DELIMITER ;

SELECT nombre_cliente_por_id(1) AS Nombre_del_Cliente;
-- 9.Una función que reciba el ID de un pedido y devuelva su total.
DELIMITER $$

CREATE FUNCTION total_pedido_por_id(id_pedido INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(10,2);
    SELECT total INTO total
    FROM pedidos
    WHERE id = id_pedido;
    RETURN total;
END $$

DELIMITER ;

SELECT total_pedido_por_id(1) AS Total;

-- 10.Crear una función que indique si un producto está en inventario.
DELIMITER $$

CREATE FUNCTION producto_en_inventario(id_producto INT)
RETURNS BOOLEAN
DETERMINISTIC
BEGIN
    DECLARE en_inventario BOOLEAN;
    SELECT CASE WHEN stock > 0 THEN TRUE ELSE FALSE END
    INTO en_inventario
    FROM productos
    WHERE id = id_producto;
    RETURN en_inventario;
END $$

DELIMITER ;

SELECT producto_en_inventario(1) AS Respuesta;

```

### TRIGGERS

```sql
-- 1.Crear un trigger que registre en HistorialSalarios cada cambio de salario de empleados.
-- 2.Crear un trigger que evite borrar productos con pedidos activos.
DELIMITER $$

CREATE TRIGGER before_product_delete
BEFORE DELETE ON productos
FOR EACH ROW
BEGIN
    DECLARE pedidos_activos INT;

    -- Verificamos si el producto está relacionado con algún pedido activo
    SELECT COUNT(*)
    INTO pedidos_activos
    FROM detalles_pedido
    WHERE id_producto = OLD.id
      AND id_pedido IN (SELECT id FROM pedidos WHERE fecha >= CURDATE());

    
    IF pedidos_activos > 0 THEN
        -- Lanza un error y evita que el producto sea eliminado
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'No se puede eliminar el producto porque hay pedidos activos que lo contienen.';
    END IF;
END $$

DELIMITER ;

-- 3.Un trigger que registre en HistorialPedidos cada actualización en Pedidos.
-- 4.Crear un trigger que actualice el inventario al registrar un pedido.
-- 5.Un trigger que evite actualizaciones de precio a menos de $1.
-- 6.Crear un trigger que registre la fecha de creación de un pedido en HistorialPedidos.
-- 7.Un trigger que mantenga el precio total de cada pedido en Pedidos.
-- 8.Crear un trigger para validar que UbicacionCliente no esté vacío al crear un cliente.
-- 9.Un trigger que registre en LogActividades cada modificación en Proveedores.
-- 10.Crear un trigger que registre en HistorialContratos cada cambio en Empleados.
```

### CONSULTAS   COMBINADAS 

```sql
-- 1.Crear una función que aplique un descuento sobre el precio de un producto si pertenece a una categoría específica.
	-- i.Crear una función CalcularDescuento que reciba el tipo_id del producto y el precio original, y aplique un descuento del 10% si el tipo es "Electrónica".
	
	-- ii.Realizar una consulta para mostrar el nombre del producto, el precio original y el precio con descuento.
	
DELIMITER $$

CREATE FUNCTION CalcularDescuento(tipo_id INT, precio_original DECIMAL(10,2))
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE descuento DECIMAL(10,2) DEFAULT 0;

    
    IF tipo_id = (SELECT id FROM tipos_de_productos WHERE tipo = 'Electrónica') THEN
        SET descuento = precio_original * 0.10; 
        RETURN precio_original - descuento;  
    ELSE
        RETURN precio_original;  -- Si no es de "Electrónica", devolvemos el precio original
    END IF;
END $$

DELIMITER ;


SELECT 
    p.nombre AS producto,
    p.precio AS precio_original,
    CalcularDescuento(t.id, p.precio) AS precio_con_descuento
FROM 
    productos p
JOIN 
    tipos_de_productos t ON p.tipo_id = t.id;
    
    
-- 2. Crear una función que calcule la edad de un cliente en función de su fecha de nacimiento y luego usarla para listar solo los clientes mayores de 18 años.

	-- i. Crear la función CalcularEdad que reciba la fecha de nacimiento y calcule la edad.
	-- ii. Consultar todos los clientes y mostrar solo aquellos que sean mayores de 18 años.
	
-- 3.Crear una función que calcule el precio final de un producto aplicando un impuesto del 15% y luego mostrar una lista de productos con el precio final incluido.
	-- i.Crear la función CalcularImpuesto que reciba el precio del producto y aplique el impuesto.
	-- ii. Mostrar el nombre del producto, el precio original y el precio final con impuesto.
	
-- 4. Crear una función que calcule el total de los pedidos de un cliente y usarla para mostrar los clientes con total de pedidos mayor a $1000.
	-- i.Crear la función TotalPedidosCliente que reciba el ID de un cliente y calcule el total de 	todos sus pedidos.
	-- ii.Realizar una consulta que muestre el nombre del cliente y su total de pedidos, y filtrar clientes con un total mayor a $1000.
	
-- 5. Crear una función que calcule el salario anual de un empleado y usarla para listar todos los empleados con un salario anual mayor a $50,000.
	-- i.Crear la función SalarioAnual que reciba el salario mensual y lo multiplique por 12.
	-- ii. Realizar una consulta que muestre el nombre del empleado y su salario anual, filtrando empleados con salario mayor a $50,000.
	
-- 6. Crear una función que calcule la bonificación de un empleado (10% de su salario) y mostrar el salario ajustado de cada empleado.
	-- i.Crear una función Bonificacion que reciba el salario y calcule el 10%.
	-- ii.Realizar una consulta que muestre el salario ajustado (salario + bonificación).
	
-- 7. Crear una función que calcule los días desde el último pedido de un cliente y mostrar clientes que hayan hecho un pedido en los últimos 30 días.
	-- i.Crear la función DiasDesdeUltimoPedido que reciba el ID de un cliente y calcule los días desde su último pedido.
	-- ii. Realizar una consulta que muestre solo a los clientes con pedidos en los últimos 30 días.
	
-- 8. Crear una función que calcule el total en inventario (cantidad x precio) de cada producto y listar productos con inventario superior a $500.
	-- i. Crear la función TotalInventarioProducto que multiplique cantidad y precio de un producto.
	-- ii. Realizar una consulta que muestre el nombre del producto y su total en inventario, filtrando los productos con inventario superior a $500.
	
-- 9. Crear un trigger y una tabla para mantener un historial de precios de productos. Cada vez que el precio de un producto cambia, el trigger debe guardar el ID del producto, el precio antiguo, el nuevo precio y la fecha de cambio.
	-- i.Crear la tabla HistorialPrecios.
    -- ii.Crear el trigger RegistroCambioPrecio en la tabla Productos para registrar los cambios de precio.

-- 10.Crear un procedimiento almacenado que genere un reporte de ventas mensual para cada empleado. El procedimiento debe recibir como parámetros el mes y el año, y devolver una lista de empleados con el total de ventas que gestionaron en ese periodo.
	-- i. Crear el procedimiento ReporteVentasMensuales.
	-- ii.Usar una subconsulta que agrupe las ventas por empleado y que filtre por el mes y el año.
	
-- 11. Realizar una consulta compleja que devuelva el producto más vendido para cada proveedor, mostrando el nombre del proveedor, el nombre del producto y la cantidad vendida.

	-- i. Utilizar una subconsulta para calcular la cantidad total de ventas por producto.
	-- ii.Filtrar en la consulta principal para obtener el producto más vendido de cada proveedor.
	
-- 12. Crear una función que calcule el estado de stock de un producto y lo clasifique en “Alto”, “Medio” o “Bajo” en función de su cantidad. Usar esta función en una consulta para listar todos los productos y su estado de stock.
	-- i.Crear la función EstadoStock que reciba la cantidad de un producto.
	-- ii.En la consulta principal, utilizar la función para clasificar el estado de stock de cada producto.
	
-- 13. Crear un trigger que, al insertar un nuevo pedido, disminuya automáticamente la cantidad en stock del producto. El trigger debe también prevenir que se inserte el pedido si el stock es insuficiente.
	-- i. Crear el trigger ActualizarInventario en la tabla DetallesPedido.
	-- ii.Controlar que no se permita la inserción si la cantidad es mayor que el stock disponible.
	
-- 14. Crear un procedimiento almacenado que genere un informe de clientes inactivos (aquellos que no han realizado pedidos en los últimos 6 meses).
	-- i. Crear el procedimiento ClientesInactivos.
	-- ii.Filtrar clientes que no tengan pedidos recientes usando una subconsulta.
	

```

