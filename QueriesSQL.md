# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
SELECT 
    c.id_cliente,
    c.nombre,
    c.cedula,
    COUNT(cu.num_cuenta) AS cantidad_cuentas,
    SUM(cu.saldo) AS saldo_total
FROM 
    Cliente c
JOIN 
    Cuenta cu ON c.id_cliente = cu.id_cliente
GROUP BY 
    c.id_cliente, c.nombre, c.cedula
HAVING 
    COUNT(cu.num_cuenta) > 1
ORDER BY 
    saldo_total DESC;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
SELECT 
    c.id_cliente,
    c.nombre,
    (SELECT SUM(monto) 
     FROM Transaccion t 
     JOIN Cuenta cu ON t.num_cuenta = cu.num_cuenta 
     WHERE cu.id_cliente = c.id_cliente AND t.tipo_transaccion = 'deposito') AS total_depositos,
    (SELECT SUM(monto) 
     FROM Transaccion t 
     JOIN Cuenta cu ON t.num_cuenta = cu.num_cuenta 
     WHERE cu.id_cliente = c.id_cliente AND t.tipo_transaccion = 'retiro') AS total_retiros,
    (SELECT SUM(monto) 
     FROM Transaccion t 
     JOIN Cuenta cu ON t.num_cuenta = cu.num_cuenta 
     WHERE cu.id_cliente = c.id_cliente AND t.tipo_transaccion = 'deposito') -
    (SELECT SUM(monto) 
     FROM Transaccion t 
     JOIN Cuenta cu ON t.num_cuenta = cu.num_cuenta 
     WHERE cu.id_cliente = c.id_cliente AND t.tipo_transaccion = 'retiro') AS diferencia
FROM 
    Cliente c
ORDER BY 
    diferencia DESC;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
SELECT 
    c.num_cuenta,
    cl.nombre AS cliente,
    c.tipo_cuenta,
    c.saldo,
    c.fecha_apertura
FROM 
    Cuenta c
JOIN 
    Cliente cl ON c.id_cliente = cl.id_cliente
WHERE 
    c.num_cuenta NOT IN (
        SELECT num_cuenta 
        FROM Tarjeta
    )
ORDER BY 
    c.fecha_apertura DESC;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT 
    c.tipo_cuenta,
    COUNT(c.num_cuenta) AS cuentas_activas,
    AVG(c.saldo) AS saldo_promedio
FROM 
    Cuenta c
WHERE 
    c.num_cuenta IN (
        SELECT DISTINCT num_cuenta 
        FROM Transaccion 
        WHERE fecha >= CURRENT_DATE - INTERVAL '30 days'
    )
GROUP BY 
    c.tipo_cuenta
ORDER BY 
    saldo_promedio DESC;
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT DISTINCT
    cl.id_cliente,
    cl.nombre,
    cl.cedula,
    cl.correo
FROM 
    Cliente cl
JOIN 
    Cuenta cu ON cl.id_cliente = cu.id_cliente
JOIN 
    Transaccion t ON cu.num_cuenta = t.num_cuenta
JOIN 
    Transferencia tr ON t.id_transaccion = tr.id_transaccion
WHERE 
    cl.id_cliente NOT IN (
        SELECT DISTINCT cl2.id_cliente
        FROM Cliente cl2
        JOIN Cuenta cu2 ON cl2.id_cliente = cu2.id_cliente
        JOIN Transaccion t2 ON cu2.num_cuenta = t2.num_cuenta
        JOIN Retiro r ON t2.id_transaccion = r.id_transaccion
        WHERE r.canal = 'cajero'
    )
ORDER BY 
    cl.nombre;
```
