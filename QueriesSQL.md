# Consultas SQL para Base de Datos Bancaria

## Enunciado 1: Clientes con múltiples cuentas y sus saldos totales

**Necesidad:** El banco necesita identificar a los clientes que tienen más de una cuenta, mostrar cuántas cuentas tiene cada uno y el saldo total acumulado en todas sus cuentas, ordenados por saldo total de mayor a menor.

**Consulta SQL:**
```sql
SELECT cl.id_cliente, cl.nombre,
COUNT(ct.num_cuenta) AS cantidad_cuentas,
SUM(ct.saldo) AS saldo_total
FROM cliente cl JOIN cuenta ct ON cl.id_cliente = ct.id_cliente
GROUP BY cl.id_cliente, cl.nombre
HAVING COUNT(ct.num_cuenta) > 1
ORDER BY saldo_total DESC;
```

## Enunciado 2: Comparativa entre depósitos y retiros por cliente

**Necesidad:** El departamento de análisis financiero necesita comparar los montos totales de depósitos y retiros realizados por cada cliente, para identificar patrones de comportamiento financiero.

**Consulta SQL:**
```sql
SELECT 
    cl.id_cliente, cl.nombre,
    COALESCE(SUM(CASE WHEN t.tipo_transaccion = 'deposito' THEN t.monto END), 0) AS total_depositos,
    COALESCE(SUM(CASE WHEN t.tipo_transaccion = 'retiro' THEN t.monto END), 0) AS total_retiros,
    (COALESCE(SUM(CASE WHEN t.tipo_transaccion = 'deposito' THEN t.monto END), 0) -
    COALESCE(SUM(CASE WHEN t.tipo_transaccion = 'retiro' THEN t.monto END), 0)) AS diferencia
FROM Cliente cl
JOIN Cuenta cu ON cl.id_cliente = cu.id_cliente
LEFT JOIN Transaccion t ON cu.num_cuenta = t.num_cuenta
GROUP BY cl.id_cliente, cl.nombre
ORDER BY diferencia DESC;
```

## Enunciado 3: Cuentas sin tarjetas asociadas

**Necesidad:** El departamento de tarjetas necesita identificar todas las cuentas que no tienen tarjetas asociadas para ofrecer productos específicos a estos clientes.

**Consulta SQL:**
```sql
SELECT cu.num_cuenta, cu.id_cliente, cl.nombre, cu.tipo_cuenta, cu.saldo, cu.fecha_apertura
FROM Cuenta cu
JOIN Cliente cl ON cu.id_cliente = cl.id_cliente
LEFT JOIN Tarjeta t ON cu.num_cuenta = t.num_cuenta
WHERE t.num_cuenta IS NULL
ORDER BY cu.num_cuenta;
```

## Enunciado 4: Análisis de saldos promedio por tipo de cuenta y comportamiento transaccional

**Necesidad:** La gerencia necesita un análisis comparativo del saldo promedio entre cuentas de ahorro y corriente, pero solo considerando aquellas cuentas que han tenido al menos una transacción en los últimos 30 días.

**Consulta SQL:**
```sql
SELECT cu.tipo_cuenta,
COUNT(DISTINCT cu.num_cuenta) AS cantidad_cuentas,
ROUND(AVG(cu.saldo), 2) AS saldo_promedio
FROM Cuenta cu
JOIN Transaccion t ON cu.num_cuenta = t.num_cuenta
WHERE t.fecha >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY cu.tipo_cuenta
```

## Enunciado 5: Clientes con transferencias pero sin retiros en cajeros

**Necesidad:** El equipo de marketing digital necesita identificar a los clientes que utilizan transferencias pero no realizan retiros por cajeros automáticos, para dirigir campañas de banca digital.

**Consulta SQL:**
```sql
SELECT DISTINCT cl.id_cliente, cl.nombre
FROM Cliente cl
JOIN Cuenta cu ON cl.id_cliente = cu.id_cliente
WHERE EXISTS (
    SELECT 1FROM Transaccion t
    WHERE t.num_cuenta = cu.num_cuenta AND t.tipo_transaccion = 'transferencia'
)
AND NOT EXISTS (
    SELECT 1 FROM Transaccion t2 JOIN Retiro r ON t2.id_transaccion = r.id_transaccion
    WHERE t2.num_cuenta = cu.num_cuenta AND t2.tipo_transaccion = 'retiro'
    AND r.canal = 'cajero'
)
```
