# Consultas NoSQL para Sistema Bancario en MongoDB

A continuación se presentan 5 enunciados de consultas basados en las colecciones NoSQL del sistema bancario, junto con las soluciones utilizando operaciones avanzadas de MongoDB.

## 1. Análisis de Saldos por Tipo de Cuenta

**Enunciado:** El departamento financiero necesita un informe que muestre el saldo total, promedio, máximo y mínimo por cada tipo de cuenta (ahorro y corriente) para evaluar la distribución de fondos en el banco.

**Consulta MongoDB:**
```javascript
db.clientes.aggregate([
  { $unwind: "$cuentas" }, 
  {
    $group: {
      _id: "$cuentas.tipo_cuenta", 
      saldo_total: { $sum: "$cuentas.saldo" },
      saldo_promedio: { $avg: "$cuentas.saldo" },
      saldo_maximo: { $max: "$cuentas.saldo" },
      saldo_minimo: { $min: "$cuentas.saldo" },
      cantidad_cuentas: { $sum: 1 }
    }
  },
  {
    $project: {
      _id: 0,
      tipo_cuenta: "$_id",
      saldo_total: { $round: ["$saldo_total", 2] },
      saldo_promedio: { $round: ["$saldo_promedio", 2] },
      saldo_maximo: 1,
      saldo_minimo: 1,
      cantidad_cuentas: 1
    }
  },
  { $sort: { saldo_total: -1 } }
])
```

## 2. Patrones de Transacciones por Cliente

**Enunciado:** El equipo de análisis de comportamiento necesita identificar los patrones de transacciones de cada cliente, mostrando la cantidad y el monto total de transacciones por tipo (depósito, retiro, transferencia) para cada cliente.

**Consulta MongoDB:**
```javascript
db.transacciones.aggregate([
  {
    $group: {
      _id: {
        cliente_ref: "$cliente_ref",
        tipo: "$tipo_transaccion"
      },
      cantidad: { $sum: 1 },
      monto_total: { $sum: "$monto" }
    }
  },
  {
    $lookup: {
      from: "clientes",
      localField: "_id.cliente_ref",
      foreignField: "_id",
      as: "cliente"
    }
  },
  {
    $unwind: "$cliente"
  },
  {
    $project: {
      _id: 0,
      nombre_cliente: "$cliente.nombre",
      cedula: "$cliente.cedula",
      tipo_transaccion: "$_id.tipo",
      cantidad_transacciones: "$cantidad",
      monto_total: 1
    }
  },
  {
    $sort: {
      nombre_cliente: 1,
      tipo_transaccion: 1
    }
  }
])
```

## 3. Clientes con Múltiples Tarjetas de Crédito

**Enunciado:** El departamento de riesgo crediticio necesita identificar a los clientes que poseen más de una tarjeta de crédito, mostrando sus datos personales, cantidad de tarjetas y el detalle de cada una.

**Consulta MongoDB:**
```javascript
db.clientes.aggregate([
  { $unwind: "$cuentas" },
  { $unwind: "$cuentas.tarjetas" },
  { 
    $match: { 
      "cuentas.tarjetas.tipo_tarjeta": "credito" 
    } 
  },
  {
    $group: {
      _id: "$_id",
      nombre: { $first: "$nombre" },
      cedula: { $first: "$cedula" },
      correo: { $first: "$correo" },
      cantidad_tarjetas: { $sum: 1 },
      tarjetas: { 
        $push: {
          numero: "$cuentas.tarjetas.numero_tarjeta",
          fecha_expiracion: "$cuentas.tarjetas.fecha_expiracion",
          cuenta_asociada: "$cuentas.num_cuenta"
        }
      }
    }
  },
  { 
    $match: { 
      cantidad_tarjetas: { $gt: 1 } 
    } 
  },
  {
    $project: {
      _id: 0,
      nombre: 1,
      cedula: 1,
      correo: 1,
      cantidad_tarjetas: 1,
      tarjetas: 1
    }
  },
  { $sort: { cantidad_tarjetas: -1 } }
])
```

## 4. Análisis de Medios de Pago más Utilizados

**Enunciado:** El departamento de marketing necesita conocer cuáles son los medios de pago más utilizados para depósitos, agrupados por mes, para orientar sus campañas promocionales.

**Consulta MongoDB:**
```javascript
db.transacciones.aggregate([
  {
    $match: {
      tipo_transaccion: "deposito"
    }
  },
  {
    $addFields: {
      mes: { $month: "$fecha" },
      año: { $year: "$fecha" }
    }
  },
  {
    $group: {
      _id: {
        medio_pago: "$detalles_deposito.medio_pago",
        mes: "$mes",
        año: "$año"
      },
      cantidad: { $sum: 1 },
      monto_total: { $sum: "$monto" }
    }
  },
  {
    $sort: {
      "_id.año": 1,
      "_id.mes": 1,
      "cantidad": -1
    }
  },
  {
    $project: {
      _id: 0,
      medio_pago: "$_id.medio_pago",
      mes: "$_id.mes",
      año: "$_id.año",
      cantidad_depositos: "$cantidad",
      monto_total_depositado: "$monto_total"
    }
  }
])
```

## 5. Detección de Cuentas con Transacciones Sospechosas

**Enunciado:** El departamento de seguridad necesita identificar cuentas con patrones de transacciones sospechosas, definidas como aquellas que tienen más de 3 retiros en un mismo día con un monto total superior a 1,000,000 COP.

**Consulta MongoDB:**
```javascript
db.transacciones.aggregate([
  { $match: { tipo_transaccion: "retiro" } },
  {
    $group: {
      _id: {
        num_cuenta: "$num_cuenta",
        dia: { $dateToString: { format: "%Y-%m-%d", date: "$fecha" } }
      },
      cantidad: { $sum: 1 },
      total: { $sum: "$monto" }
    }
  },
  { $match: { cantidad: { $gt: 3 }, total: { $gt: 1000000 } } },
  { $sort: { total: -1 } }
])
```
