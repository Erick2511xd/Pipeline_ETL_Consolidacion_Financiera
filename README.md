# Pipeline ETL para Consolidación Financiera Regional (Perú / Brasil)

## 📌 Descripción del Proyecto
Este proyecto resuelve un desafío crítico en empresas multinacionales de la región LATAM: la centralización y homogeneización de transacciones financieras provenientes de diferentes filiales (Perú y Brasil). 

Las filiales operativas registran sus ventas diarias en sus respectivas monedas locales (Soles PEN y Reales BRL), utilizando configuraciones regionales distintas para los decimales (puntos vs. comas) y estructuras de texto mixtas para las fechas.

Se diseñó e implementó una solución **ETL (Extract, Transform, Load)** de nivel corporativo utilizando **SQL Server** y **SQL Server Integration Services (SSIS)** en Visual Studio para automatizar la extracción, limpieza de caracteres, conversión cambiaria dinámica y carga física en un Data Warehouse centralizado para control de gestión.

---

## 🏗️ Arquitectura del Flujo de Datos (Data Flow)
El pipeline procesa los datos de forma optimizada en memoria sin necesidad de etapas intermedias de scripting complejo:

```text
  [ OLE DB Source: Leer Ventas Crudas ] (Extracción transaccional PE / BR)
                   │
                   ▼
  [ Data Conversion ] (Homogeneización a tipos de datos compatibles)
                   │
                   ▼
  [ Lookup: Tipo_Cambio ] (Cruce con tabla maestra diaria de divisas)
                   │
                   ▼
  [ Derived Column: Limpieza_Financiera ] (Lógica condicional de conversión)
                   │
                   ▼
  [ OLE DB Destination: Fact_Ventas ] (Inyección en Data Warehouse Limpio)
```

---

## 🛠️ Reglas de Negocio Implementadas en la Columna Derivada

### 1. Unificación Monetaria (`Monto_PEN`)
Se utilizó una expresión condicional para limpiar la configuración regional brasileña (reemplazo de comas por puntos), castear la cadena a tipo dinero (`DT_CY`) y ejecutar la conversión de divisas en tiempo real solo si correspondía a la filial extranjera:
```text
[Codigo_Moneda] == "BRL" ? (DT_CY)(replace([Copy of Monto_Texto], ",", ".")) * [Factor_Conversion] : (DT_CY)(replace([Copy of Monto_Texto], ",", "."))
```

### 2. Estandarización Cronológica (`Fecha_Estandar`)
Se corrigieron las estructuras de barras regionales a guiones normados y se forzó el casteo al tipo de dato nativo de fecha global (`YYYY-MM-DD`):
```text
(DT_DBDATE)(replace([Copy of Fecha_Texto], "/", "-"))
```

---

## 📊 Resultados Técnicos y Verificación en SQL Server
El pipeline inyecta la información de forma exitosa en la tabla de hechos final, logrando la consistencia del dato requerida por las gerencias de finanzas corporativas.

```sql
SELECT 
    ID_Original AS [ID Ventas Crudas],
    Codigo_Moneda_Origen AS [Moneda Origen],
    Monto_Original AS [Texto Crudo de Origen],
    Monto_PEN AS [Monto Convertido a Soles (MONEY)],
    Fecha_Estandar AS [Fecha Limpia (DATE)]
FROM Datawarehouse.Fact_Ventas_Consolidadas;
```

---

## 🎯 Habilidades Demostradas
* **Arquitectura de Datos y Modelamiento Dimensional**
* **Integridad del Dato y Calidad de Datos (Data Quality)**
* **SQL Server Integration Services (SSIS) Avanzado**
* **Lógica de Negocio Financiera Internacional**
