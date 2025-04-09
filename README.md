# TP_AVANZADO
EJERCICIO 3: la Tabla T (Transacciones)
SQL
-- ---------------------------------
START TRANSACTION;
INSERT INTO T (id, s) VALUES (4, 'fourth');
SELECT * FROM T ;
ROLLBACK;
SELECT * FROM T;
-- ---------------------------------
*Cuantas tuplas tiene T (antes de este ejercicio)?*
Antes de empezar este bloque de comandos, 
la tabla T tiene 3 tuplas (con id 1, 2 y 3).
Resultado del SELECT * FROM T ; (dentro de la transacción):
-- ---------------------------------
![image](https://github.com/user-attachments/assets/161c58cd-a9a6-4763-bbe7-2873abf8d544)
-- ---------------------------------


Resultado del SELECT * FROM T; (después del ROLLBACK):
-- ---------------------------------
![image](https://github.com/user-attachments/assets/2afc8614-133c-4c60-88d6-3e54ea035baf)
-- ---------------------------------

*Justifique la respuesta:*
La instrucción START TRANSACTION; inicia un bloque de operaciones. Dentro de este bloque,
se inserta una nueva fila con id = 4. El primer SELECT muestra el estado de la tabla en ese momento,
incluyendo la nueva fila. Sin embargo, la instrucción ROLLBACK; deshace todos los cambios realizados
dentro de esta transacción. Por lo tanto, el segundo SELECT muestra la tabla volviendo a su estado
anterior, sin la fila con id = 4.

EJERCICIO 4: la Tabla T. 2da parte
SQL
-- ------------------------------------
INSERT INTO T (id, s) VALUES (5, 'fifth');
COMMIT;
SELECT * FROM T;
-- ------------------------------------
*¿Que se obtiene como resultado de ejecutar la sentencia SELECT * FROM T?*
El resultado de la sentencia SELECT * FROM T; será:
-- ---------------------------------
![image](https://github.com/user-attachments/assets/ddcf9217-8acb-4f7f-93d5-7b8c7a706374)
-- ---------------------------------
*Justifíquelo:*
La sentencia INSERT INTO T (id, s) VALUES (5, 'fifth'); añade una nueva fila a la tabla T.
La instrucción COMMIT; guarda de forma permanente todos los cambios realizados desde la última vez 
que se inició una transacción (en este caso, la inserción de la fila con id = 5). Por lo tanto, 
la sentencia SELECT * FROM T; final muestra todas las filas de la tabla, incluyendo la recién insertada.
El COMMIT asegura que los datos se escriban de forma duradera en la base de datos.

EJERCICIO 5: la Tabla T. 3era parte 
SQL
------------------------------------
START TRANSACTION;
DELETE FROM T WHERE id > 1;
INSERT INTO T (id, s) VALUES (2, 'second');
INSERT INTO T (id, s) VALUES (3, 'third');
SELECT * FROM T;
ROLLBACK;
SELECT * FROM T;
------------------------------------

*¿Que se obtiene como resultado de ejecutar la sentencia SELECT * FROM T?*
Primer SELECT * FROM T; (antes del ROLLBACK):

id | s      | si
---+--------+----
 1 | first  |
 2 | second |
 3 | third  |
(Asumiendo el estado inicial o después del COMMIT del Ejercicio 4)

Segundo SELECT * FROM T; (después del ROLLBACK):
-- ---------------------------------
![image](https://github.com/user-attachments/assets/9f02e581-416e-4537-b758-b337bbfda68d)
-- ---------------------------------
*Justifíquelo*
 Al iniciar la transacción (START TRANSACTION), eliminamos filas y luego insertamos otras. 
 El primer SELECT muestra estos cambios temporales. El ROLLBACK deshace todos los cambios 
 hechos dentro de la transacción, por lo que el segundo SELECT devuelve la tabla a su estado
 justo antes del START TRANSACTION.
 EJERCICIO 6 : la Tabla T. Prueba de Errores (Contexto Supabase/PostgreSQL)

SQL
------------------------------------
DELETE FROM T WHERE id > 1;
COMMIT;
INSERT INTO T (id, s) VALUES (2, 'La prueba de errores comienza aqui');
SELECT (1/0) AS dummy FROM T;
DELETE FROM T WHERE id = 7777 ;
------------------------------------
*Como se comportará el Código? Que resultados dará?*

 INSERT INTO T (id, s) VALUES (2, 'La prueba de errores comienza aqui');;--ERROR:  22001: value too long for type character varying(30)
 SELECT (1/0) AS dummy FROM T; -- Error: Intento de división por cero
 DELETE FROM T WHERE id = 7777 ; -- Este código NO se ejecutará debido al error anterior
 
 Restricciones de Tipo de Datos: Es fundamental respetar las restricciones definidas en el 
 esquema de la base de datos. En este caso, la columna s solo puede almacenar cadenas de hasta 30 caracteres.
 Validación de Datos: Antes de intentar insertar o actualizar datos, es importante validar
 su longitud para asegurarse de que cumplen con las restricciones de las columnas correspondientes.
 Esto se puede hacer tanto en la aplicación (antes de enviar la consulta a la base de datos) como en
 la propia base de datos (a través de CHECK constraints, aunque no se definieron en este ejemplo).
Manejo de Errores: La aplicación que interactúa con la base de datos debe estar preparada para manejar
este tipo de errores de inserción o actualización, informando al usuario de manera clara y evitando que la aplicación falle.

*En que estado queda T (cual es su contenido)*
 T queda con la fila id = 1 (si no se borró) y sin la fila id = 2 debido al error de longitud.
 -- ---------------------------------
 ![image](https://github.com/user-attachments/assets/9128e774-076e-4eb1-9658-cc2b5ac7f80f)
 -- ---------------------------------

*Como se evitan estos errores*
Error de longitud de VARCHAR: Validar la longitud de los datos antes de la inserción (en la aplicación o con restricciones en la base de datos).
Error de división por cero: Asegurarse de que el divisor no sea cero mediante lógica condicional (IF, CASE) o funciones de manejo seguro de división.
Error de código no ejecutado: Corregir los errores que detienen la ejecución (los anteriores) y manejar las excepciones en la aplicación para que un error
no interrumpa todo el flujo.

*Como quedaría el código?*
SQL
------------------------------------
-- Solución para el error de longitud (truncando):
DELETE FROM T WHERE id > 1;
COMMIT;
INSERT INTO T (id, s) VALUES (2, LEFT('La prueba de errores comienza aqui', 30));

-- Solución para el error de división por cero (usando CASE):
SELECT
    CASE
        WHEN 0 = 0 THEN NULL -- O algún valor seguro
        ELSE (1/0)
    END AS dummy
FROM T;

-- Ahora esta línea se ejecutará (si los errores anteriores se corrigieron)
DELETE FROM T WHERE id = 7777 ;

*¿Qué aprendemos de los resultados?*
Múltiples errores detienen el flujo: Un error en una sentencia SQL puede impedir que las sentencias posteriores en el mismo lote se ejecuten.
La importancia del esquema: Las restricciones de la tabla (como la longitud del VARCHAR) deben ser respetadas.
Robustez en las consultas: Evitar operaciones que puedan generar errores en tiempo de ejecución (como la división por cero) es crucial.
Manejo integral de errores: La prevención en la base de datos (consultas seguras, validación de esquema) y el manejo en la aplicación (captura de excepciones) son necesarios para un sistema confiable.
