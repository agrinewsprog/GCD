# 📰 Sistema de Newsletters - Guía de Instalación

## Fase 1: Base de Datos + Seed ✅

### 📋 Paso 1: Crear las Tablas

Ejecuta el script de creación de tablas:

```bash
cd GCD-api
mysql -h localhost -P 3307 -u root -proot123 gcd_db < database/newsletter-system.sql
```

Esto crea:

- ✅ `newsletter_types` - Tipos de newsletter con reglas de envío
- ✅ `newsletter_schedules` - Fechas programadas para todo el año
- ✅ Columna `newsletter_schedule_id` en `campaign_actions`

### 📋 Paso 2: Insertar Newsletter Types

```bash
mysql -h localhost -P 3307 -u root -proot123 gcd_db < database/newsletter-seed.sql
```

Esto inserta:

- ✅ 6 newsletters de aviNews España (miércoles)
- ✅ 6 newsletters de aviNews International (jueves)

### 📋 Paso 3: Generar Fechas del Año

```bash
cd GCD-api
node scripts/generate-newsletter-schedules.js 2026
```

Esto genera todas las fechas para 2026 según las reglas definidas.

**Salida esperada:**

```
🗓️  Generando schedules de newsletters para 2026...

✅ Encontrados 12 tipos de newsletter

📰 aviNews - Spain - News aviNews España
   Wednesday semana 1 (monthly)
   ✓ 12 fechas generadas
   📅 Ejemplos: 2026-01-07, 2026-02-04, 2026-03-04...

📰 aviNews - Spain - News ovoNews
   Wednesday semana 2 (monthly)
   ✓ 12 fechas generadas
   📅 Ejemplos: 2026-01-14, 2026-02-11, 2026-03-11...

...

✅ Proceso completado!
   Total: 132 schedules generados para 2026

📊 Resumen del año 2026:
   Total schedules: 132
   Disponibles: 132
   Asignados: 0
```

### 🔍 Verificar Instalación

```sql
-- Ver todos los newsletter types
SELECT
  nt.id,
  m.name as medio,
  nt.region,
  nt.name as newsletter,
  nt.day_of_week as dia,
  CONCAT('Semana ', nt.week_of_month) as semana,
  nt.frequency as frecuencia
FROM newsletter_types nt
JOIN mediums m ON nt.medium_id = m.id
ORDER BY m.name, nt.region;

-- Ver fechas generadas para aviNews España (primeros 5)
SELECT
  ns.id,
  nt.name as newsletter,
  ns.scheduled_date as fecha,
  DAYNAME(ns.scheduled_date) as dia_semana,
  ns.is_available as disponible
FROM newsletter_schedules ns
JOIN newsletter_types nt ON ns.newsletter_type_id = nt.id
WHERE nt.medium_id = (SELECT id FROM mediums WHERE name = 'aviNews')
  AND nt.region = 'Spain'
ORDER BY ns.scheduled_date
LIMIT 5;
```

## 📊 Estructura de Datos

### newsletter_types

```
id | medium_id | region | name | day_of_week | week_of_month | frequency | frequency_offset
1  | 1         | Spain  | News aviNews España | Wednesday | 1 | monthly | 0
2  | 1         | Spain  | News ovoNews | Wednesday | 2 | monthly | 0
...
```

### newsletter_schedules

```
id | newsletter_type_id | scheduled_date | year | month | is_available
1  | 1                  | 2026-01-07    | 2026 | 1     | TRUE
2  | 1                  | 2026-02-04    | 2026 | 2     | TRUE
...
```

## 🎯 Ejemplos de Fechas Generadas

### aviNews España (Miércoles)

- **News aviNews España** (Semana 1): 7 ene, 4 feb, 4 mar, 1 abr, 6 may, 3 jun...
- **News ovoNews** (Semana 2): 14 ene, 11 feb, 11 mar, 8 abr, 13 may, 10 jun...
- **News PoultryPro** (Semana 2): 14 ene, 11 feb, 11 mar, 8 abr, 13 may, 10 jun...
- **News On Air** (Semana 3): 21 ene, 18 feb, 18 mar, 15 abr, 20 may, 17 jun...
- **News MeatPro** (Semana 4): 28 ene, 25 feb, 25 mar, 22 abr, 27 may, 24 jun...
- **Incubanews** (Semana 4, cada 2 meses): 25 feb, 22 abr, 24 jun, 26 ago, 28 oct, 23 dic

### aviNews International (Jueves)

- **News aviNews Int** (Semana 1): 8 ene, 5 feb, 5 mar, 2 abr, 7 may, 4 jun...
- **ovoNews** (Semana 2): 15 ene, 12 feb, 12 mar, 9 abr, 14 may, 11 jun...
- Y así sucesivamente...

## 🔧 Regenerar Fechas

Si necesitas regenerar las fechas (por ejemplo, para 2027):

```bash
node scripts/generate-newsletter-schedules.js 2027
```

El script detecta automáticamente fechas duplicadas y las omite.

## ❓ Troubleshooting

### Error: "No hay newsletter types definidos"

**Solución**: Ejecuta primero el seed:

```bash
mysql -h localhost -P 3307 -u root -proot123 gcd_db < database/newsletter-seed.sql
```

### Error: "ER_NO_SUCH_TABLE: Table 'gcd_db.newsletter_types' doesn't exist"

**Solución**: Ejecuta primero la creación de tablas:

```bash
mysql -h localhost -P 3307 -u root -proot123 gcd_db < database/newsletter-system.sql
```

### Verificar que aviNews existe

```sql
SELECT id, name FROM mediums WHERE name = 'aviNews';
```

Si no existe, el seed lo creará automáticamente.

## 🎉 ¡Fase 1 Completada!

Ahora tienes:

- ✅ Tablas creadas
- ✅ Newsletter types de aviNews configurados
- ✅ Todas las fechas de 2026 generadas
- ✅ Sistema listo para asignar a campañas

### 🚀 Siguiente Paso

**Fase 2**: Crear endpoints del backend para consultar newsletters disponibles.

**Endpoints necesarios:**

- `GET /api/newsletters/types?medium_id=1&region=Spain` - Listar newsletter types
- `GET /api/newsletters/schedules?type_id=1&available=true` - Fechas disponibles
- `POST /api/campaigns/:id/actions` - Modificar para vincular con newsletter_schedule_id
