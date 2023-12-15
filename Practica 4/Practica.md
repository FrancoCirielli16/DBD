# Diseño de Bases de Datos

### Práctica 4 :

### Dadas las siguientes relaciones, resolver utilizando SQL las consultas planteadas.


Ejercicio 1

- Cliente(-idCliente-, nombre, apellido, DNI, telefono, direccion)
- Factura (-nroTicket-, total, fecha, hora,idCliente (fk))
- Detalle(-nroTicket-, idProducto-, cantidad, preciounitario)
- Producto(-idProducto-, descripcion, precio, nombreP, stock)

1. Listar datos personales de clientes cuyo apellido comience con el string ‘Pe’. Ordenar por
DNI


~~~sql

    SELECT nombre,apellido,DNI,telefono,direccion
    FROM Cliente
    WHERE(apellido LIKE "Pe%")
    ORDER BY DNI
    
~~~

2. Listar nombre, apellido, DNI, teléfono y dirección de clientes que realizaron compras
solamente durante 2017.


~~~sql

    SELECT nombre,apellido,DNI,telefono,direccion
    FROM Cliente c INNER JOIN Factura f ON (c.idCliente = f.idCliente)
    WHERE (f.fecha BETWEEN 01/01/2017 and 31/12/2017)
    EXCEPT(
        SELECT nombre,apellido,DNI,telefono,direccion
        FROM Cliente c INNER JOIN Factura f ON (c.idCliente = f.idCliente)
        WHERE (f.fecha < 01/01/2017 and f.fecha > 31/12/2017)
    )

~~~

3. Listar nombre, descripción, precio y stock de productos vendidos al cliente con
DNI:45789456, pero que no fueron vendidos a clientes de apellido ‘Garcia’.


~~~sql

    SELECT p.nombreP,p.descripcion,p.precio,p.stock
    FROM Producto p 
    INNER JOIN Detalle d ON (p.idProducto = d.idProducto)
    INNER JOIN Factura f ON (d.nroTicket = f.nroTicket)
    INNER JOIN Cliente c ON (f.idCliente = c.idCliente)
    WHERE (c.DNI = 45789456)
    EXCEPT(
        SELECT p.nombreP,p.descripcion,p.precio,p.stock
        FROM Producto p 
        INNER JOIN Detalle d ON (p.idProducto = d.idProducto)
        INNER JOIN Factura f ON (d.nroTicket = f.nroTicket)
        INNER JOIN Cliente c ON (f.idCliente = c.idCliente)
        WHERE (c.apellido = "Garcia")
    )

~~~

4. Listar nombre, descripción, precio y stock de productos no vendidos a clientes que
tengan teléfono con característica: 221 (La característica está al comienzo del teléfono).
Ordenar por nombre.


~~~sql

    SELECT p.nombreP,p.descripcion,p.precio,p.stock
    FROM Producto p
    WHERE p.idProducto NOT IN (
        SELECT p.nombreP,p.descripcion,p.precio,p.stock
        FROM Producto p 
        INNER JOIN Detalle d ON (p.idProducto = d.idProducto)
        INNER JOIN Factura f ON (d.nroTicket = f.nroTicket)
        INNER JOIN Cliente c ON (f.idCliente = c.idCliente)
        WHERE(c.telefono LIKE "221%")
    )
    ORDER BY (p.nombreP)

~~~

5. Listar para cada producto: nombre, descripción, precio y cuantas veces fué vendido.
Tenga en cuenta que puede no haberse vendido nunca el producto.


~~~sql

    SELECT p.nombre, p.descripcion, p.precio, SUM(d.cantidad) as Cantidad
    FROM Producto p LEFT JOIN Detalle d (p.idProducto = d.idProducto)
    GROUP BY p.nombre, p.descripcion, p.precio

~~~

6. Listar nombre, apellido, DNI, teléfono y dirección de clientes que compraron los
productos con nombre ‘prod1’ y ‘prod2’ pero nunca compraron el producto con nombre
‘prod3’.


~~~sql

   SELECT c.nombre, c.apellido, c.DNI, c.telefono, c.direccion
   FROM Producto p 
    INNER JOIN Detalle d ON (p.idProducto = d.idProducto)
    INNER JOIN Factura f ON (d.nroTicket = f.nroTicket)
    INNER JOIN Cliente c ON (f.idCliente = c.idCliente)
    WHERE (p.nombreP = "prod1" AND (c.idCliente IN (
        SELECT c.idCliente
        FROM Producto p 
            INNER JOIN Detalle d ON (p.idProducto = d.idProducto)
            INNER JOIN Factura f ON (d.nroTicket = f.nroTicket)
            INNER JOIN Cliente c ON (f.idCliente = c.idCliente)
            WHERE (p.nombreP = "prod2"))) /*De los clientes que compraron prod1 me fijo con el IN si tambien compraron el prod2*/
    EXCEPT( 
        SELECT c.nombre, c.apellido, c.DNI, c.telefono, c.direccion
        FROM Producto p 
            INNER JOIN Detalle d ON (p.idProducto = d.idProducto)
            INNER JOIN Factura f ON (d.nroTicket = f.nroTicket)
            INNER JOIN Cliente c ON (f.idCliente = c.idCliente)
            WHERE (p.nombreP = "prod3")
    ) /*Le saco los clientes que compraron prod3 tambien*/

~~~
7. Listar nroTicket, total, fecha, hora y DNI del cliente, de aquellas facturas donde se haya
comprado el producto ‘prod38’ o la factura tenga fecha de 2019.


~~~sql

    SELECT f.nroTicket,f.total,f.fecha,f.hora,c.DNI
    FROM Producto p 
        INNER JOIN Factura f ON (d.nroTicket = f.nroTicket)
        INNER JOIN Cliente c ON (f.idCliente = c.idCliente) 
    WHERE ((f.fecha BETWEEN 01/01/2019 AND 31/12/2019) OR (p.nombreP = "prod38"))

~~~
8. Agregar un cliente con los siguientes datos: nombre:’Jorge Luis’, apellido:’Castor’,
DNI:40578999, teléfono:221-4400789, dirección:’11 entre 500 y 501 nro:2587’ y el id de
cliente: 500002. Se supone que el idCliente 500002 no existe.


~~~sql

    INSERT INTO Cliente (idCliente, nombre, apellido, DNI, telefono, direccion)
    VALUES(500002,"Jorge Luis","Castor","40578999","221-4400789","11 entre 500 y 501 nro:2587")

~~~
9. Listar nroTicket, total, fecha, hora para las facturas del cliente ´Jorge Pérez´ donde no
haya comprado el producto ´Z´.


~~~sql
    /*OPCION 1 NO 100% EXACTA*/
    SELECT f.nroTicket,f.total,f.fecha,f.hora
    FROM Facturas f 
            INNER JOIN Cliente c ON (f.idCliente = c.idCliente)
            INNER JOIN Detalle d ON (c.idCliente = d.idCliente)
            INNER JOIN Producto p ON(d.idCliente = p.idCliente)
    WHERE ((nombre="Jorge") AND (apellido="Perez"))
    EXCEPT(
        SELECT f.nroTicket,f.total,f.fecha,f.hora
        FROM Facturas f 
            INNER JOIN Cliente c ON (f.idCliente = c.idCliente)
            INNER JOIN Detalle d ON (c.idCliente = d.idCliente)
            INNER JOIN Producto p ON(d.idCliente = p.idCliente)
        WHERE (((nombre="Jorge") AND (apellido="Perez")) AND p.nombreP = "Z")
~~~

~~~sql
    /*OPCION 2*/
    SELECT f.nroTicket, total, fecha, hora
    FROM Producto p 
    INNER JOIN Detalle d ON (p.idProducto = d.idProducto)
    INNER JOIN Factura f ON (d.nroTicket = f.nroTicket) 
    INNER JOIN Cliente c ON (f.idCliente =  c.idCliente)
    /*CON NOT IN BASICAMENTE ESTOY DICIENDO QUE DEJE LOS QUE NO SEAN PRODUCTO Z*/
    WHERE (nombre="Jorge") AND (apellido="Perez") AND f.nroTicket NOT IN(
        SELECT f.nroTicket
        FROM Producto p 
        INNER JOIN Detalle d ON (p.idProducto = d.idProducto)
        INNER JOIN Factura f ON (d.nroTicket = f.nroTicket) 
        INNER JOIN Cliente c ON (f.idCliente =  c.idCliente)
        WHERE (nombreP="Z")
    )
~~~


10. Listar DNI, apellido y nombre de clientes donde el monto total comprado, teniendo en cuenta todas sus facturas, supere $10.000.000.


~~~sql

    SELECT c.DNI,c.apellido,c.nombre
    FROM Factura f INNER JOIN Cliente c ON (f.idCliente = c.idCliente)
    GROUP BY c.DNI,c.apellido,c.nombre
    HAVING SUM(f.total) > 10000000

~~~

Ejercicio 2

- AGENCIA (-RAZON_SOCIAL-, dirección, telef, e-mail)
- CIUDAD (-CODIGOPOSTAL-, nombreCiudad, añoCreación)
- CLIENTE (-DNI-, nombre, apellido, teléfono, dirección)
- VIAJE( -FECHA,HORA,DNI- , cpOrigen(fk), cpDestino(fk), razon_social(fk), descripcion) //cpOrigen y cpDestino corresponden a la ciudades origen y destino del viaje


1. Listar razón social, dirección y teléfono de agencias que realizaron viajes desde la ciudad de ‘La Plata’ (ciudad origen) y que el cliente tenga apellido ‘Roma’. Ordenar por razón social y luego por teléfono.

~~~sql

    SELECT a.RAZON_SOCIAL,a.direccion,a.telefono
    FROM VIAJE v 
    INNER JOIN CLIENTE c ON (v.DNI = c.DNI)
    INNER JOIN AGENCIA a ON (v.RAZON_SOCIAL = a.RAZON_SOCIAL)
    WHERE ((c.apellido = "Roma") AND (v.RAZON_SOCIAL IN (
        SELECT v.RAZON_SOCIAL
        FROM VIAJE v INNER JOIN CIUDAD c ON (v.cpOrigen = c.cpOrgien)
        WHERE (c.nombreCiudad = "La PLata")
    )))
    ORDER BY a.RAZON_SOCIAL, a.telefono

~~~

2. Listar fecha, hora, datos personales del cliente, ciudad origen y destino de viajes realizados
en enero de 2019 donde la descripción del viaje contenga el String ‘demorado’.

~~~sql

    SELECT v.fecha,v.hora,c.DNI,c.nombre,c.apellido,c.telefono,c.direccion,origen.nombreCiudad,destino.nombreCiudad
    FROM VIAJE v 
    INNER JOIN CLIENTE cl ON (v.DNI = cl.DNI)
    INNER JOIN CIUDAD origen ON (v.cpOrigen = origen.cpOrigen)
    INNER JOIN CIUDAD destino ON (v.cpDestino = destino.cpDestino)
    WHERE ((v.fecha BETWEEN 01/01/2019 AND 31/01/2019)AND(v.descricion LIKE "%demorado%"))

~~~

3. Reportar información de agencias que realizaron viajes durante 2019 o que tengan dirección
de mail que termine con ‘@jmail.com’.


~~~sql

    SELECT a.RAZON_SOCIAL, a.direccion, a.telef, a.email
    FROM AGENCIA a INNER JOIN VIAJE v (a.RAZON_SOCIAL = v.RAZON_SOCIAL)
    WHERE ((v.fecha BETWEEN "01/01/2019" and "31/12/2019") OR (a.email LIKE "%@jmail.com"))

~~~

4. Listar datos personales de clientes que viajaron solo con destino a la ciudad de ‘Coronel Brandsen’


~~~sql

    SELECT c.DNI, c.nombre, c.apellido, c.teléfono, c.dirección
    FROM VIAJE v 
    INNER JOIN CLIENTE c ON (v.DNI = c.DNI)
    INNER JOIN CIUDAD cd (v.cpDestino = c.cpDestino)
    WHERE(cd.nombreCiudad = "Coronel Brandsen")
    EXCEPT(
        SELECT c.DNI, c.nombre, c.apellido, c.teléfono, c.dirección
        FROM VIAJE v 
        INNER JOIN CLIENTE c ON (v.DNI = c.DNI)
        INNER JOIN CIUDAD cd (v.cpDestino = c.cpDestino)
        WHERE NOT (cd.nombreCiudad = "Coronel Brandsen")
    )

~~~

5. Informar cantidad de viajes de la agencia con razón social ‘TAXI Y’ realizados a ‘Villa Elisa’.


~~~sql

    SELECT COUNT as "viajes"
    FROM VIAJE v 
    INNER JOIN CIUDAD c (v.cpDestino = c.cpDestino)
    INNER JOIN AGENCIA a (a.RAZONSOCIAL = v.RAZONSOCIAL)
    WHERE (a.RAZONSOCIAL = "TAXI Y" and c.nombreCiudad = "Villa Elisa")

~~~

6. Listar nombre, apellido, dirección y teléfono de clientes que viajaron con todas las agencias.


~~~sql
    /* OPCION QUE NO DEJA LA CATEDRA CREO*/
    SELECT c.nombre, c.apellido, c.direccion, c.telefono
    FROM Cliente c INNER JOIN Viaje v ON (c.DNI = v.DNI)
    GROUP BY c.DNI, c.nombre, c.apellido, c.direccion, c.telefono
    HAVING COUNT(DISTINCT v.RAZON_SOCIAL) = (SELECT COUNT(*) FROM AGENCIA);

~~~

~~~sql

    SELECT c.nombre, c.apellido, c.direccion, c.telefono
    FROM Cliente c INNER JOIN Viaje v ON (c.DNI = v.DNI)
    GROUP BY c.DNI, c.nombre, c.apellido, c.direccion, c.telefono
    HAVING (SELECT COUNT(*) AS "cantidad razones unicas"
    FROM (
            SELECT RAZON_SOCIAL
            FROM AGENCIA
            GROUP BY RAZON_SOCIAL
        ) /*ESTARIA TOMANDO TODAS LAS AGENCIAS EN LAS QUE VIAJO EL CLIENTE Y CONTANDOLA SIN QUE SE REPITAN*/ 
    ) = (SELECT COUNT(*) FROM AGENCIA);

~~~
Tabla Cliente:

| DNI | Nombre | Apellido | Dirección | Teléfono |
|-----|--------|----------|-----------|----------|
| 001 | Juan   | Pérez    | Calle A   | 12345678 |
| 002 | María  | López    | Calle B   | 23456789 |
| 003 | Pedro  | Gómez    | Calle C   | 34567890 |

Tabla Agencia:

| RAZON_SOCIAL | Nombre |
|--------------|--------|
| A1           | Agencia Uno |
| A2           | Agencia Dos |
| A3           | Agencia Tres |

Tabla Viaje:

| DNI | RAZON_SOCIAL |
|-----|--------------|
| 001 | A1           |
| 001 | A2           |
| 002 | A1           |
| 002 | A2           |
| 002 | A3           |
| 003 | A1           |
| 003 | A2           |
| 003 | A3           |

Primero, unimos las tablas Cliente y Viaje utilizando el DNI:

| DNI | Nombre | Apellido | Dirección | Teléfono | RAZON_SOCIAL |
|-----|--------|----------|-----------|----------|--------------|
| 001 | Juan   | Pérez    | Calle A   | 12345678 | A1           |
| 001 | Juan   | Pérez    | Calle A   | 12345678 | A2           |
| 002 | María  | López    | Calle B   | 23456789 | A1           |
| 002 | María  | López    | Calle B   | 23456789 | A2           |
| 002 | María  | López    | Calle B   | 23456789 | A3           |
| 003 | Pedro  | Gómez    | Calle C   | 34567890 | A1           |
| 003 | Pedro  | Gómez    | Calle C   | 34567890 | A2           |
| 003 | Pedro  | Gómez    | Calle C   | 34567890 | A3           |

Luego, agrupamos por DNI y contamos el número de RAZON_SOCIAL distintos:


| DNI | Nombre | Apellido | Dirección | Teléfono | COUNT(DISTINCT RAZON_SOCIAL) |
|-----|--------|----------|-----------|----------|-----------------------------|
| 001 | Juan   | Pérez    | Calle A   | 12345678 | 2                           |
| 002 | María  | López    | Calle B   | 23456789 | 3                           |
| 003 | Pedro  | Gómez    | Calle C   | 34567890 | 3                           |

Finalmente, aplicamos el filtro HAVING para seleccionar solo aquellos clientes que hayan viajado con todas las agencias, es decir, que tengan un recuento de RAZON_SOCIAL igual al recuento total de agencias:


| DNI | Nombre | Apellido | Dirección | Teléfono | COUNT(DISTINCT RAZON_SOCIAL) |
|-----|--------|----------|-----------|----------|-----------------------------|
| 002 | María  | López    | Calle B   | 23456789 | 3                           |
| 003 | Pedro  | Gómez    | Calle C   | 34567890 | 3                           |




7. Modificar el cliente con DNI: 38495444 actualizando el teléfono a: 221-4400897.

~~~sql

   UPDATE CLIENTE SET telefono = "221-4400897" WHERE DNI = "38495444"

~~~

8. Listar razon_social, dirección y teléfono de la/s agencias que tengan mayor cantidad de viajes realizados.

~~~sql

    SELECT a.RAZONSOCIAL,a.direccion,a.telefono
    FROM AGENCIA a INNER JOIN VIAJES v ON(a.RAZONSOCIAL = v.RAZONSOCIAL)
    GROUP BY a.RAZONSOCIAL,a.direccion,a.telefono
    HAVING (COUNT(*) >= ALL (
        SELECT COUNT(*)  
        FROM VIAJE v
        GROUP BY v.RAZON_SOCIAL 
        )
    )

~~~

9. Reportar nombre, apellido, dirección y teléfono de clientes con al menos 10 viajes.

~~~sql

    SELECT c.nombre,c.apellido,c.direccion,c.telefono
    FROM CLIENTE c INNER JOIN VIAJES v ON (c.DNI = v.DNI)
    GROUP BY c.nombre,c.apellido,c.direccion,c.telefono
    HAVING (COUNT(*) >= 10)

~~~
10. Borrar al cliente con DNI 40325692.

~~~sql

    DELETE FROM CLIENTE WHERE DNI = "40325692"
    DELETE FROM VIAJE WHERE DNI = "40325692"
    

~~~

Ejericio 3:

- Club=(-codigoClub-, nombre, anioFundacion, codigoCiudad(FK))
- Ciudad=(-codigoCiudad-, nombre)
- Estadio=(-codigoEstadio-, codigoClub(FK), nombre, direccion)
- Jugador=(-DNI-, nombre, apellido, edad, codigoCiudad(FK))
- ClubJugador(-codigoClub, DNI-, desde, hasta)

1. Reportar nombre y anioFundacion de aquellos clubes de la ciudad de La Plata que no poseen estadio.  

~~~sql

    SELECT c.nombre,c.anioFundacion
    FROM Club c LEFT JOIN Estadio e ON (c.codigoClub = e.codigoClub)
    INNER JOIN Ciudad ciudad ON (c.codigoCiudad = ciudad.codigoCiudad)
    WHERE (ciudad.nombre = "La Plata" AND e.codigoEstadio is NULL)

~~~

2. Listar nombre de los clubes que no hayan tenido ni tengan jugadores de la ciudad de Berisso.
~~~sql

    SELECT c.nombre
    FROM Club c INNER JOIN ClubJugador cj ON (c.codigoClub = cj.codigoClub)
    WHERE (cj.DNI NOT IN(
        SELECT j.DNI
        FROM Jugador j INNER JOIN Ciudad ciudad ON(j.codigoCiudad = ciudad.codigoCiudad)
        WHERE(ciudad.nombre = "Berisso")
        )
    )
    

~~~
3. Mostrar DNI, nombre y apellido de aquellos jugadores que jugaron o juegan en el club
Gimnasia y Esgrima La PLata.
~~~sql

    SELECT j.DNI, j.nombre, j.apellido 
    FROM Jugador j 
    INNER JOIN ClubJugador cj ON (j.DNI = cj.DNI) 
    INNER JOIN Club c ON (c.codigoClub = cj.codigoClub)
    WHERE (c.nombre = "Gimnasia y Esgrima La Plata")
    

~~~
4. Mostrar DNI, nombre y apellido de aquellos jugadores que tengan más de 29 años y
hayan jugado o juegan en algún club de la ciudad de Córdoba.
~~~sql

    SELECT j.DNI, j.nombre, j.apellido 
    FROM Jugador j INNER JOIN ClubJugador cj ON (j.DNI = cj.DNI)
    INNER JOIN Club c ON (cj.codigoClub = c.codigoClub)
    INNER JOIN Ciudad ciu ON (c.codigoCiudad = ciu.codigoCiudad)
    WHERE j.edad > 29 AND ciu.nombre = "Cordoba"
    

~~~
5. Mostrar para cada club, nombre de club y la edad promedio de los jugadores que juegan
actualmente en cada uno.
~~~sql

    SELECT c.nombre, AVG(j.edad) as Promedio
    FROM Club c 
    LEFT JOIN ClubJugador cj ON (c.codigoClub = cj.codigoClub)
    INNER JOIN Jugador j ON (cj.DNI = j.DNI)
    WHERE cj.hasta IS NULL
    GROUP BY c.codigoClub, c.nombre
    

~~~
6. Listar para cada jugador: nombre, apellido, edad y cantidad de clubes diferentes en los
que jugó. (incluido el actual)
~~~sql

   
    SELECT j.nombre,j.apellido,j.edad,COUNT(*) as Cantidad
    FROM Jugador j INNER JOIN ClubJugador cj ON (j.DNI = cj.DNI)
    GROUP BY j.DNI,j.nombre,j.edad

~~~

7. Mostrar el nombre de los clubes que nunca hayan tenido jugadores de la ciudad de Mar
del Plata.
~~~sql

    SELECT club.nombre
    FROM Club club
    EXCEPT(
        SELECT club.nombre
        FROM Club club
        INNER JOIN ClubJugador cj ON (c.codigoClub = cj.codigoClub)
        INNER JOIN Jugador j ON(cj.DNI = j.DNI)
        INNER JOIN Ciudad c ON (j.codigoCiudad = c.codigoCiudad)
        WHERE (c.nombre = "Mar del Plata")
    )



~~~
8. Reportar el nombre y apellido de aquellos jugadores que hayan jugado en todos los
clubes.
~~~sql

   SELECT j.nombre,j.apellido 
   FROM Jugador j INNER JOIN ClubJugador cj ON (j.DNI = cj.DNI)
   GROUP BY j.nombre,j.apellido
   HAVING(DISTINCT c.codigoClub) = (SELECT COUNT(*) FROM Club)
    

~~~
9. Agregar con codigoClub 1234 el club “Estrella de Berisso” que se fundó en 1921 y que
pertenece a la ciudad de Berisso. Puede asumir que el codigoClub 1234 no existe en la
tabla Club.
~~~sql

   
    INSERT INTO Club(codigoClub,nombre,anioFundacion,CodigoCiudad(FK))
    VALUES("1234","Estrella de Berisso","1921",(
        SELECT codigoCiudad
        FROM Ciudad
        WHERE (nombre = "Berisso"))
    )

~~~

Ejercicio 4:

- PERSONA = (-DNI-, Apellido, Nombre, Fecha_Nacimiento, Estado_Civil, Genero)
- ALUMNO = (-DNI-, Legajo, Año_Ingreso)
- PROFESOR = (-DNI-, Matricula, Nro_Expediente)
- TITULO = (-Cod_Titulo-, Nombre, Descripción)
- TITULO-PROFESOR = (-Cod_Titulo, DNI-, Fecha)
- CURSO = (-Cod_Curso-, Nombre, Descripción, Fecha_Creacion, Duracion)
- ALUMNO-CURSO = (-DNI, Cod_Curso, Año-, Desempeño, Calificación)
- PROFESOR-CURSO = (-DNI, Cod_Curso, Fecha_Desde-, Fecha_Hasta)

1. Listar DNI, legajo y apellido y nombre de todos los alumnos que tegan año ingreso
inferior a 2014.
~~~sql

    SELECT a.DNI,a.legajo,p.apellido,p.nombre
    FROM PERSONA p INNER JOIN ALUMNO a ON (p.DNI = a.DNI)
    WHERE(a.Año_Ingreso < 2014)
   
    

~~~
2. Listar DNI, matricula, apellido y nombre de los profesores que dictan cursos que tengan
más 100 horas de duración. Ordenar por DNI
~~~sql

   SELECT p.DNI,pro.matricula,p.apellido,p.nombre
   FROM FROM PERSONA p INNER JOIN PROFESOR pro ON (p.DNI = pro.DNI)
   INNER JOIN PROFESOR-CURSO pc ON (pro.DNI = pc.DNI)
   INNER JOIN CURSO c ON (pc.Cod_Curso = c.Cod_Curso)
   WHERE (c.Duracion a >= 100)
   ORDER BY p.DNI
    

~~~
3. Listar el DNI, Apellido, Nombre, Género y Fecha de nacimiento de los alumnos inscriptos
al curso con nombre “Diseño de Bases de Datos” en 2019.
~~~sql

   SELECT a.DNI,p.Genero,p.apellido,p.nombre,p.Fecha_Nacimiento
   FROM FROM PERSONA p INNER JOIN ALUMNO a ON (p.DNI = a.DNI)
   INNER JOIN ALUMNO-CURSO ac ON (pro.DNI = ac.DNI)
   INNER JOIN CURSO c ON (pc.Cod_Curso = c.Cod_Curso)
   WHERE (c.nombre ="Diseño de Base de Datos" and ac.Año = "2019")
    

~~~
4. Listar el DNI, Apellido, Nombre y Calificación de aquellos alumnos que obtuvieron una
calificación superior a 9 en los cursos que dicta el profesor “Juan Garcia”. Dicho listado deberá estar ordenado por Apellido.

~~~sql

    SELECT a.DNI, p.Apellido, p.Nombre, ac.Calificacion
    FROM PERSONA p
    NATURAL JOIN ALUMNO a
    NATURAL JOIN ALUMNO_CURSO ac
    NATURAL JOIN PROFESOR_CURSO pc
    NATURAL JOIN PROFESOR prof
    NATURAL JOIN PERSONA p2
    WHERE (ac.Calificacion > 9 AND p2.Nombre = 'Juan' AND p2.Apellido = 'Garcia')
    ORDER BY p.Apellido;

    
~~~

~~~sql

    SELECT a.DNI,p.Apellido,p.Nombre,
    FROM PERSONA p INNER JOIN ALUMNO a ON (p.DNI = a.DNI)
    INNER JOIN ALUMNO-CURSO ac ON (pro.DNI = ac.DNI)
    INNER JOIN PROFESOR-CURSO pc ON(ac.Cod_Curso = pc.Cod_Curso)
    INNER JOIN PROFESOR prof ON (pc.DNI = prof.DNI)
    INNER JOIN PERSONA p2 ON (prof.DNI = p2.DNI)
    WHERE(ac.Calificacion >= 9 and p2.nombre = "Juan" and p2.Apellido = "Garcia")
    ORDER BY p.apellido
    
~~~
5. Listar el DNI, Apellido, Nombre y Matrícula de aquellos profesores que posean más de 3 títulos. Dicho listado deberá estar ordenado por Apellido y Nombre.
~~~sql

   SELECT p.DNI,p.Apellido,p.Nombre,p.Matricula
   FROM PERSONA p 
   NATURAL JOIN PROFESOR pro 
   NATURAL JOIN TITULO-PROFESOR tp
   GROUP BY p.DNI, per.Apellido, per.Nombre, p.Matricula
   HAVING (COUNT(*)>3)
   ORDER BY per.Apellido, per.Nombre
    

~~~
6. Listar el DNI, Apellido, Nombre, Cantidad de horas y Promedio de horas que dicta cada
profesor. La cantidad de horas se calcula como la suma de la duración de todos los
cursos que dicta.
~~~sql

   
    SELECT  p.DNI,p.Apellido,p.nombre, SUM(c.Duracion),AVG(c.Duracion)
    FROM PROFESOR p 
    INNER JOIN PERSONA per ON (p.DNI = per.DNI)
    INNER JOIN PROFESOR-CURSO pc ON(p.DNI = pc.DNI)
    INNER JOIN CURSO c ON (pc.Cod_Curso = c.Cod_Curso)
    GROUP BY p.DNI,p.Apellido,p.nombre

~~~
7. Listar Nombre, Descripción del curso que posea más alumnos inscriptos y del que posea
menos alumnos inscriptos durante 2019.
~~~sql

    (SELECT a.Nombre,a.Descripcion
    FROM CURSO c NATURAL JOIN ALUMNO-CURSO ac
    GROUP BY a.Nombre,a.Descripcion
    HAVING (COUNT(*) >= ALL (
            SELECT COUNT(*)  
            FROM ALUMNO-CURSO
            WHERE Año = "2019"
            GROUP BY Cod_Curso
            )
        )
    ) UNION (
        SELECT a.Nombre,a.Descripcion
        FROM CURSO c NATURAL JOIN ALUMNO-CURSO ac
        GROUP BY a.Nombre,a.Descripcion
        HAVING (COUNT(*) <= ALL (
            SELECT COUNT(*)  
            FROM ALUMNO-CURSO
            WHERE Año = "2019"
            GROUP BY Cod_Curso
            )
        )   
    )
    
~~~

8. Listar el DNI, Apellido, Nombre, Legajo de alumnos que realizaron cursos con nombre conteniendo el string ‘BD’ durante 2018 pero no realizaron ningún curso durante 2019.
~~~sql

    SELECT a.DNI,a.apellido,a.Nombre,a.Legajo
    FROM PERSONA p 
    INNER JOIN ALUMNO a ON (p.DNI = a.DNI)
    INNER JOIN ALUMNO-CURSO ac ON (a.DNI = ac.DNI)
    INNER JOIN CURSO c ON (ac.Cod_Curso = c.Cod_Curso)
    WHERE(c.nombre LIKE %BD% and ac.Año = "2018")
    EXCEPT(
        SELECT a.DNI,a.apellido,a.Nombre,a.Legajo
        FROM PERSONA p 
        INNER JOIN ALUMNO a ON (p.DNI = a.DNI)
        INNER JOIN ALUMNO-CURSO ac ON (a.DNI = ac.DNI)
        WHERE(ac.Año = "2019")
    )

~~~
9. Agregar un profesor con los datos que prefiera y agregarle el título con código: 25.
~~~sql

    INSERT INTO PERSONA(DNI,APELLIDO,Nombre,Fecha_Nacimiento,Estado_Civil,Genero)
    VALUE("13123","Cirielli","Fraco","16/12/2002","Soltero")
    INSERT INTO PROFESOR(DNI,Matricula,Nro_Expediente)
    VALUE("13123","32132","3123")
    INSERT INTO TITULO-PROFESOR(DNI,Cod_Titulo,Fecha)
    VALUE("13123","1111111","05/06/2023")
   
    

~~~
10. Modificar el estado civil del alumno cuyo legajo es ‘2020/09’, el nuevo estado civil es
divorciado.
~~~sql

   UPDATE PERSONA 
   SET Estado_civil="divorciado" 
   WHERE(
        SELECT DNI 
        FROM ALUMNO 
        WHERE a.Legajo="2020/09"
    )
    

~~~
11. Dar de baja el alumno con DNI 30568989. Realizar todas las bajas necesarias para no
dejar el conjunto de relaciones en estado inconsistente.
~~~sql
    DELETE FROM PERSONA WHERE(DNI = "30568989")
    DELETE FROM ALUMNO WHERE(DNI = "30568989")
    DELETE FROM ALUMNO-CURSO WHERE(DNI = "30568989")
~~~

Ejercicio 5:

- Localidad(-CodigoPostal-, nombreL, descripcion, #habitantes)
- Arbol(-nroArbol-, especie, años, calle, nro, codigoPostal(fk))
- Podador(-DNI-, nombre, apellido, telefono,fnac,codigoPostalVive(fk))
- Poda(-codPoda-,fecha, DNI(fk),nroArbol(fk))

1. Listar especie, años, calle, nro. y localidad de árboles podados por el podador ‘Juan Perez’ y por el podador ‘Jose Garcia’.

~~~sql
    
~~~

2. Reportar DNI, nombre, apellido, fnac y localidad donde viven podadores que tengan podas durante 2018.
~~~sql
    
~~~
3. Listar especie, años, calle, nro y localidad de árboles que no fueron podados nunca.
~~~sql
    
~~~

4. Reportar especie, años,calle, nro y localidad de árboles que fueron podados durante 2017 y no fueron podados durante 2018.
~~~sql
    
~~~
5. Reportar DNI, nombre, apellido, fnac y localidad donde viven podadores con apellido terminado con el string ‘ata’ y que el podador tenga al menos una poda durante 2018. Ordenar por apellido y nombre.
~~~sql
    
~~~
6. Listar DNI, apellido, nombre, teléfono y fecha de nacimiento de podadores que solo podaron árboles de especie ‘Coníferas’.
~~~sql
    
~~~
7. Listar especie de árboles que se encuentren en la localidad de ‘La Plata’ y también en la localidad de ‘Salta’.
~~~sql
    
~~~
8. Eliminar el podador con DNI: 22234566.
~~~sql
    
~~~
9. Reportar nombre, descripción y cantidad de habitantes de localidades que tengan menos de 100 árboles.
~~~sql
    
~~~

Ejercicio 6:

- Técnico (codTec, nombre, especialidad) // técnicos
- Repuesto (codRep, nombre, stock, precio) // repuestos
- RepuestoReparacion (nroReparac, codRep, cantidad, precio) //repuestos utilizados en reparaciones.
- Reparación (nroReparac, codTec, precio_total, fecha) //reparaciones realizadas.

1. Listar todos los repuestos, informando el nombre, stock y precio. Ordenar el
resultado por precio.
2. Listar nombre, stock, precio de repuesto que participaron en reparaciones durante
2019 y además no participaron en reparaciones del técnico ‘José Gonzalez’.
3. Listar el nombre, especialidad de técnicos que no participaron en ninguna
reparación. Ordenar por nombre ascendentemente.
4. Listar el nombre, especialidad de técnicos solo participaron en reparaciones durante
2018.
5. Listar para cada repuesto nombre, stock y cantidad de técnicos distintos que lo
utilizaron. Si un repuesto no participó en alguna reparación igual debe aparecer en
dicho listado.
6. Listar nombre y especialidad del técnico con mayor cantidad de reparaciones
realizadas y el técnico con menor cantidad de reparaciones.
7. Listar nombre, stock y precio de todos los repuestos con stock mayor a 0 y que
dicho repuesto no haya estado en reparaciones con precio_total superior a 10000.
8. Proyectar precio, fecha y precio total de aquellas reparaciones donde se utilizó algún
repuesto con precio en el momento de la reparación mayor a $1000 y menor a
$5000.
9. Listar nombre, stock y precio de repuestos que hayan sido utilizados en todas las
reparaciones
10. Listar fecha, técnico y precio total de aquellas reparaciones que necesitaron al
menos 10 repuestos distintos.
Ejercicio 7:
Banda(codigoB, nombreBanda, genero_musical, año_creacion)
Integrante (DNI, nombre, apellido,dirección,email, fecha_nacimiento,codigoB(fk))
Escenario(nroEscenario, nombre _ escenario, ubicación,cubierto, m2, descripción)
Recital(fecha,hora,nroEscenario, codigoB (fk))
Página 4 de 9
1. Listar DNI, nombre, apellido,dirección y email de integrantes nacidos entre 1980 y 1990 y
hayan realizado algún recital durante 2018.
2. Reportar nombre, género musical y año de creación de bandas que hayan realizado
recitales durante 2018, pero no hayan tocado durante 2017 .
3. Listar el cronograma de recitales del dia 04/12/2018. Se deberá listar: nombre de la banda
que ejecutará el recital, fecha, hora, y el nombre y ubicación del escenario correspondiente.
4. Listar DNI, nombre, apellido,email de integrantes que hayan tocado en el escenario con
nombre ‘Gustavo Cerati’ y en el escenario con nombre ‘Carlos Gardel’.
5. Reportar nombre, género musical y año de creación de bandas que tengan más de 8
integrantes.
6. Listar nombre de escenario, ubicación y descripción de escenarios que solo tuvieron
recitales con género musical rock and roll. Ordenar por nombre de escenario
7. Listar nombre, género musical y año de creación de bandas que hayan realizado recitales
en escenarios cubiertos durante 2018.// cubierto es true, false según corresponda
8. Reportar para cada escenario, nombre del escenario y cantidad de recitales durante 2018.
9. Modificar el nombre de la banda ‘Mempis la Blusera’ a: ‘Memphis la Blusera’.
Ejercicio 8- Dadas las siguientes relaciones
Equipo(codigoE, nombreE, descripcionE)
Integrante (DNI, nombre, apellido,ciudad,email, telefono,codigoE(fk))
Laguna(nroLaguna, nombreL, ubicación,extension, descripción)
TorneoPesca(codTorneo, fecha,hora,nroLaguna(fk), descripcion)
Inscripcion(codTorneo,codigoE,asistio, gano)// asistio y gano son true o false según
corresponda
1. Listar DNI, nombre, apellido y email de integrantes que sean de la ciudad ‘La Plata’ y estén
inscriptos en torneos a disputarse durante 2019.
2. Reportar nombre y descripción de equipos que solo se hayan inscripto en torneos de 2018.
3. Listar DNI, nombre, apellido,email y ciudad de integrantes que asistieron a torneos en la
laguna con nombre ‘La Salada, Coronel Granada’ y su equipo no tenga inscripciones a
torneos a disputarse en 2019.
4. Reportar nombre, y descripción de equipos que tengan al menos 5 integrantes.Ordenar por
nombre y descripción.
5. Reportar nombre, y descripción de equipos que tengan inscripciones en todas las lagunas.
6. Eliminar el equipo con código:10000.
7. Listar nombreL, ubicación,extensión y descripción de lagunas que no tuvieron torneos.
8. Reportar nombre, y descripción de equipos que tengan inscripciones a torneos a disputarse
durante 2019, pero no tienen inscripciones a torneos de 2018.
9. Listar DNI, nombre, apellido, ciudad y email de integrantes que ganaron algún torneo que se
disputó en la laguna con nombre: ‘Laguna de Chascomús’.
Página 5 de 9
Ejercicio 9-Dadas las siguientes relaciones
Proyecto(codProyecto, nombrP,descripcion, fechaInicioP, fechaFinP, fechaFinEstimada,DNIResponsable,
equipoBackend, equipoFrontend) //DNIResponsable corresponde a un empleado, equipoBackend y
equipoFrontend corresponden a un equipo
Equipo(codEquipo, nombreE, descripcionTecnologias,DNILider)//DNILider corresponde a un empleado
Empleado(DNI,nombre, apellido, telefono, direccion, fechaIngreso)
Empleado_Equipo(codEquipo,DNI, fechaInicio, fechaFin,descripcionRol)
1. Listar nombre, descripción, fecha de inicio y fecha de fin de proyectos ya finalizados que no fueron
terminados antes de la fecha de fin estimada.
2. Listar DNI, nombre, apellido, telefono, dirección y fecha de ingreso de empleados que no son, ni
fueron responsables de proyectos. Ordenar por apellido y nombre.
3. Listar DNI, nombre, apellido, teléfono y dirección de líderes de equipo que tenga más de un
equipo a cargo.
4. Listar DNI, nombre, apellido, teléfono y dirección de todos los empleados que trabajan en el
proyecto con nombre ‘Proyecto X’. No es necesario informar responsable y líderes.
5. Listar nombre de equipo y datos personales de líderes de equipos que no tengan empleados
asignados y trabajen con tecnología ‘Java’.
6. Modificar nombre, apellido y dirección del empleado con DNI: 40568965 con los datos que desee.
7. Listar DNI, nombre, apellido, teléfono y dirección de empleados que son responsables de
proyectos pero no han sido líderes de equipo.
8. Listar nombre de equipo y descripción de tecnologías de equipos que hayan sido asignados como
equipos frontend y backend.
9. Listar nombre, descripción, fecha de inicio, nombre y apellido de responsables de proyectos a
finalizar durante 2019.
10 Dadas las siguientes relaciones
Vehiculo = (patente, modelo, marca, peso, km)
Camion = (patente, largo, max_toneladas, cant_ruedas, tiene_acoplado)
Auto = (patente, es_electrico, tipo_motor)
Service = (fecha, patente, km_service, observaciones, monto)
Parte = (cod_parte, nombre, precio_parte)
Service_Parte = (fecha, patente, cod_parte, precio)
1. Listar todos los datos de aquellos camiones que tengan entre 4 y 8 ruedas, y que hayan realizado algún
service en los últimos 365 días. Ordenar por patente, modelo y marca.
2. Listar los autos que hayan realizado el service “cambio de aceite” antes de los 13.000 km o hayan realizado el
service “inspección general” que incluya la parte “filtro de combustible”.
3. Listar nombre y precio de todas las partes que aparezcan en más de 30 service que hayan salido (partes) más
de $4.000.
4. Dar de baja todos los camiones con más de 250.000 km.
5. Listar el nombre y precio de aquellas partes que figuren en todos los service realizados en el corriente año.
Página 6 de 9
6. Listar todos los autos cuyo tipo de motor sea eléctrico. Mostrar información de patente, modelo , marca y peso.
7. Dar de alta una parte, cuyo nombre sea “Aleron” y precio $&400.
8. Dar de baja todos los services que se realizaron al auto con patente ‘AWA564’.
9. Listar todos los vehículos que hayan tenido services durante el 2018.
Ejercicio 11 -
Modelo físico
Box = (nroBox,m2, ubicación, capacidad, ocupacion) //ocupación es un numérico indicando
cantidad de mascotas en el box actualmente, capacidad es una descripción.
Mascota = (codMascota,nombre, edad, raza, peso, telefonoContacto)
Veterinario = (matricula, CUIT, nombYAp, direccion, telefono)
Supervision = (codMascota,nroBox, fechaEntra, fechaSale?, matricula(fk),
descripcionEstadia) //fechaSale tiene valor null si la mascota está actualmente en el box
1. Listar para cada veterinario cantidad de supervisiones realizadas con fecha de salida
(fechaSale) durante enero de 2020. Indicar matricula, CUIT, nombre y apellido,
dirección, teléfono y cantidad de supervisiones.
2. Listar CUIT, matricula, nombre, apellido,dirección y teléfono de veterinarios que no
tengan mascotas bajo supervisión actualmente.
3. Listar nombre, edad, raza, peso y teléfono de contacto de mascotas fueron atendidas
por el veterinario ‘Oscar Lopez’. Ordenar por nombre y raza de manera ascendente.
4. Modificar nombre y apellido al veterinario con matricula: ‘MP 10000’, deberá
llamarse: ‘Pablo Lopez’.
5. Listar nombre, edad, raza, peso de mascotas que tengan supervisiones con el
veterinario con matricula : ‘MP 1000’ y con el veterinario con matricula: ‘MN 4545’.
6. Listar nroBox, m2, ubicación, capacidad y nombre de mascota para supervisiones con
fecha de entrada (fechaEntra) durante 2020.
Ejercicio 12 -
Modelo Físico
Barberia = (codBarberia, razon_social, direccion, telefono)
Cliente = (nroCliente,DNI, nombYAp, direccionC, fechaNacimiento, celular)
Barbero = (codEmpleado,DNIB, nombYApB, direccionB, telefonoContacto, mail)
Atencion = (codEmpleado,Fecha,hora,codBarberia(fk), nroCliente(fk),descTratamiento, valor)
Página 7 de 9
1. Listar DNI, nombYAp, direccionC, fechaNacimiento y celular de clientes que no tengan
atención durante 2020.
2. Listar para cada barbero cantidad de atenciones que realizaron durante 2018. Listar
DNIB, nombYApB, direccionB, telefonoContacto, mail y cantidad de atenciones.
3. Listar razón social, dirección y teléfono de barberias que tengan atenciones para el
cliente con DNI:22283566 . Ordenar por razón social y dirección ascendente.
4. Listar DNIB, nombYApB, direccionB, telefonoContacto y mail de barberos que tengan
atenciones con valor superior a 5000.
5. Listar DNI, nombYAp, direccionC, fechaNacimiento y celular de clientes que tengan
atenciones en la barbería con razón social: ‘Corta barba’ y también se hayan atendido
en la barbería con razón social: ‘Barberia Barbara’.
6. Eliminar el cliente con DNI: 22222222.
Ejercicio 13-
Modelo Físico
Club(IdClub,nombreClub,ciudad)
Complejo(IdComplejo,nombreComplejo, IdClub(fk))
Cancha(IdCancha,nombreCancha,IdComplejo(fk))
Entrenador(IdEntrenador, nombreEntrenador,fechaNacimiento, direccion)
Entrenamiento(IdEntrenamiento, fecha, IdEntrenador(fk), IdCancha(fk))
1- Listar nombre, fecha nacimiento y dirección de entrenadores que hayan tenido
entrenamientos durante 2020.
2- Listar para cada cancha del complejo “Complejo 1” , la cantidad de entrenamientos que se
realizaron durante el 2019. Informar nombre de la cancha y cantidad de entrenamientos.
3- Listar los complejos donde haya realizado entrenamientos el entrenador “Jorge Gonzalez”.
Informar nombre de complejo, ordenar el resultado de manera ascendente.
4- Listar nombre , fecha de nacimiento y dirección de entrenadores que hayan entrenado en
la cancha “Cancha 1” y en la Cancha “Cancha 2”.
5- Listar todos los clubes en los que entrena el entrenador “Marcos Perez”. Informar nombre
del club y ciudad.
Página 8 de 9
6- Eliminar los entrenamientos del entrenador ‘Juan Perez’.
Ejercicio 14 -
Modelo Físico
Cine (idCine, nombreC, direccion)
Sala (nroSala, nombreS, descripción, capacidad,idCine(fk))
Pelicula (idPeli, nombre, descripción, genero)
Funcion (nroFuncion, nroSala(fk), idPeli(fk), fecha, hora, ocupación)//ocupación indica cantidad de
espectadores de la función
1- Listar nombre, descripción y género de películas con funciones durante 2020.
2- Listar para cada Sala cantidad de espectadores que asistieron durante 2020. Indicar
nombre de la sala, nombre del cine y total de espectadores.
3- Listar nombre de cine y dirección para los cines que tienen o tuvieron función para la
película ‘Relic’. Ordenar por nombre de Cine y dirección desc.
4- Listar nombre ,descripción y género de películas que tienen función en la sala con nombre
‘Sala Lola Membrives’ o tienen función en el cine con nombre ‘Gran Rex’.
5- Listar nombre del cine y dirección de cines que tengan salas con capacidad superior a los
300 espectadores.
6- Agregar un cine con nombre cine ‘Cine Ricardo Darin’, dirección: ‘calle 2 nro 1900, La
Plata’ e idCine: 5000, asuma que no existe dicho id.
