# ASE251S4-db — Agro Pacayales S.A.C.
**Plataforma de Persistencia Políglota como Código (Database-as-Code)**  
**Curso:** ASE251S4 | **Tecnologías:** Docker, Microsoft SQL Server 2022, MongoDB 8.0  
**Propósito del Sistema:** Comparativa de Costos vs Ganancias para la Toma de Decisiones

---

## 🌾 1. Propósito y Valor del Negocio
Agro Pacayales S.A.C. es una empresa agrícola orientada al cultivo y exportación de palto Hass y arándanos. Esta plataforma resuelve la necesidad crítica de **calcular con precisión matemática cuánto cuesta mantener cada parcela en insumos y labores versus cuánto rinde su cosecha en el mercado internacional**.

* **Microsoft SQL Server (Kardex e Inmutabilidad Financiera):** Controla compras, stock físico, costos unitarios de aplicación por parcela y facturación final de exportación.
* **MongoDB (Catálogo Dinámico y Georreferenciación):** Almacena las geometrías satelitales GeoJSON (`Polygon`), las fichas técnicas variables (NPK, SENASA) y la trazabilidad de calidad de los lotes cosechados.

---

## 📂 2. Estructura del Repositorio

```text
ASE251S4-db/
├── .env.example              # Variables de configuración y contraseñas
├── .gitignore                # Protección de archivos locales
├── README.md                 # Guía de despliegue y pruebas
├── docs/
│   ├── architecture.md       # Arquitectura técnica y dependencias
│   └── data-ownership.md     # Matriz de propiedad de datos y regla de oro
├── sqlserver/
│   ├── Dockerfile            # Imagen base de SQL Server 2022
│   ├── docker/
│   │   ├── entrypoint.sh     # Script orquestador del contenedor
│   │   └── init.sh           # Runner que aplica scripts ordenados
│   └── init/
│       ├── 01_database.sql   # Creación de agropacayales_db
│       ├── 02_schemas.sql    # Esquemas agropacayales y finanzas
│       ├── 03_tables.sql     # Tablas usuarios, stock, compras, labores, liquidación
│       ├── 04_constraints.sql# Restricciones CHECK, UNIQUE y FKs
│       ├── 05_indexes.sql    # Índices optimizados para reportes de costos
│       ├── 06_views.sql      # Vista comparativa Costos vs Ganancias
│       ├── 07_procedures.sql # Procedimiento de análisis de rentabilidad
│       └── 08_seed.sql       # Datos iniciales con costos e ingresos
└── mongodb/
    ├── Dockerfile            # Imagen base de MongoDB 8.0
    └── init/
        ├── 01_database.js    # Conexión a agropacayales_db
        ├── 02_collections.js # Colecciones parcelas, insumos, cosechas, fichas
        ├── 03_validators.js  # Reglas estrictas $jsonSchema
        ├── 04_indexes.js     # Índices geoespaciales 2dsphere
        └── 05_seed.js        # Semilla inicial enlazada por IDs
```

---

## 🔨 3. Construcción de Imágenes Independientes (Etapa 3)

Ejecuta en la terminal de PowerShell desde la raíz de `ASE251S4-db`:

```bash
# 1. Construir imagen de SQL Server
docker build -t ase251s4-db-sqlserver:1.0 ./sqlserver

# 2. Construir imagen de MongoDB
docker build -t ase251s4-db-mongodb:1.0 ./mongodb

# 3. Verificar las imágenes creadas
docker images
```

---

## 🚀 4. Despliegue de Contenedores y Prueba Funcional

```bash
# 1. Levantar contenedor de SQL Server
docker run -d --name sqlserver-db -p 1433:1433 -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=AgroPacayales2026!" ase251s4-db-sqlserver:1.0

# 2. Levantar contenedor de MongoDB
docker run -d --name mongodb-db -p 27017:27017 ase251s4-db-mongodb:1.0

# 3. Verificar estado de los procesos
docker ps
```

---

## 🔍 5. Verificación de Logs de Inicialización

Comprueba que las dependencias bloqueantes se hayan ejecutado en orden lógico:

```bash
# Logs de SQL Server
docker logs sqlserver-db

# Logs de MongoDB
docker logs mongodb-db
```

Verás las confirmaciones en consola:
```text
[INFO] Starting Microsoft SQL Server...
[INFO] Executing 01_database.sql
[INFO] Executing 02_schemas.sql
[INFO] Executing 03_tables.sql
...
[SUCCESS] All initialization scripts executed.
```

---

## 📊 6. Consulta Clave: Toma de Decisiones (Costos vs Ganancias)

Conéctate a SQL Server (mediante SSMS o `sqlcmd`) y ejecuta:

```sql
USE agropacayales_db;
GO

-- Consulta comparativa de rentabilidad por parcela
SELECT * FROM finanzas.vw_comparativa_costos_ganancias;

-- Informe gerencial con veredicto de inversión para una parcela
EXEC finanzas.sp_analizar_rentabilidad_parcela 'PARC-NORTE-A1';
GO
```

### Resultado que apoya la toma de decisiones:
* **Inversión en Insumos:** \$1,450.00 USD (Insecticida BioProtect Pro + NPK 20-20-20).
* **Ingreso por Exportación:** \$14,080.00 USD (4,400 kg de Palta Hass Cat 1 a Rotterdam).
* **Ganancia Neta:** +\$12,630.00 USD.
* **Retorno de Inversión (ROI):** 871%.
* **Veredicto Gerencial:** **`ALTA RENTABILIDAD (EXPANDIR CULTIVO)`**.
