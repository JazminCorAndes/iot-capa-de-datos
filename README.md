# Reto -- Capa de Datos IoT

**Curso:** Internet de las Cosas (IoT)\
**Universidad de los Andes**\
**Repositorio:** https://github.com/JazminCorAndes/iot-capa-de-datos

------------------------------------------------------------------------

## 📌 Descripción del Reto

En este repositorio se encuentran las modificaciones realizadas a las
aplicaciones desarrolladas en el tutorial de la capa de datos,
implementando una nueva consulta en:

-   Aplicación con PostgreSQL
-   Aplicación con TimescaleDB

El objetivo fue diseñar una consulta interesante sobre la entidad `Data`
y sus entidades relacionadas, exponerla mediante un endpoint REST que
retorna un JSON, redesplegar las aplicaciones en la nube y comparar su
desempeño mediante pruebas de carga con JMeter.

------------------------------------------------------------------------

## 🔧 Modificaciones Realizadas

Para facilitar la comparación, se subieron primero los archivos
originales del tutorial y posteriormente se aplicaron los cambios
necesarios.

### Nueva Consulta Implementada

Endpoint creado:

    /dailyAvg/

### 🎯 Propósito de la Consulta

Calcular el promedio diario de temperatura por ciudad durante los
últimos 7 días.

Esta consulta permite:

-   Análisis agregados sobre datos de sensores.
-   Simular consultas reales de monitoreo ambiental.
-   Implementar agregación temporal (clave en sistemas IoT).

------------------------------------------------------------------------

## 🗄 Implementación en PostgreSQL

``` python
from django.http import JsonResponse
from django.db import connection
from django.utils import timezone
from datetime import timedelta

def daily_avg_by_city(request):
    end = timezone.now()
    start = end - timedelta(days=7)

    with connection.cursor() as cursor:
        cursor.execute("""
            SELECT 
                c.name AS city,
                m.name AS measurement,
                DATE(d.base_time) AS day,
                AVG(d.avg_value) AS avg_value
            FROM "realtimeGraph_data" d
            JOIN "realtimeGraph_station" s ON d.station_id = s.id
            JOIN "realtimeGraph_location" l ON s.location_id = l.id
            JOIN "realtimeGraph_city" c ON l.city_id = c.id
            JOIN "realtimeGraph_measurement" m ON d.measurement_id = m.id
            WHERE d.base_time >= %s AND d.base_time <= %s
            GROUP BY city, measurement, day
            ORDER BY city, measurement, day;
        """, [start, end])

        rows = cursor.fetchall()

    data = []
    for city, measurement, day, avg_value in rows:
        data.append({
            "city": city,
            "measurement": measurement,
            "day": day.isoformat(),
            "avg_value": float(avg_value) if avg_value is not None else None
        })

    return JsonResponse(data, safe=False)

```

------------------------------------------------------------------------

## ⏱ Implementación en TimescaleDB

``` python
from django.http import JsonResponse
from django.db import connection
from django.utils import timezone
from datetime import timedelta

def daily_avg_by_city(request):
    end = timezone.now()
    start = end - timedelta(days=7)

    with connection.cursor() as cursor:
        cursor.execute("""
            SELECT 
                c.name AS city,
                m.name AS measurement,
                time_bucket('1 day', d.base_time) AS day,
                AVG(d.avg_value) AS avg_value
            FROM "realtimeGraph_data" d
            JOIN "realtimeGraph_station" s ON d.station_id = s.id
            JOIN "realtimeGraph_location" l ON s.location_id = l.id
            JOIN "realtimeGraph_city" c ON l.city_id = c.id
            JOIN "realtimeGraph_measurement" m ON d.measurement_id = m.id
            WHERE d.base_time >= %s AND d.base_time <= %s
            GROUP BY city, measurement, day
            ORDER BY city, measurement, day;
        """, [start, end])

        rows = cursor.fetchall()

    data = []
    for city, measurement, day, avg_value in rows:
        data.append({
            "city": city,
            "measurement": measurement,
            "day": day.isoformat(),
            "avg_value": float(avg_value) if avg_value is not None else None
        })

    return JsonResponse(data, safe=False)

```

------------------------------------------------------------------------

## 🌐 Redespliegue en la Nube

Las aplicaciones fueron redesplegadas en instancias de AWS EC2,
reiniciando los servicios después de aplicar las modificaciones.

Verificación mediante:

    http://<IP_PUBLICA>/dailyAvg/

------------------------------------------------------------------------

## 📊 Pruebas de Carga

Se realizaron pruebas de carga utilizando Apache JMeter:

-   60 requests
-   Comparación entre PostgreSQL y TimescaleDB
-   Métricas evaluadas:
    -   Latencia promedio
    -   Throughput
    -   Desviación estándar
    -   Error %

Los scripts de prueba se incluyen en el repositorio.

------------------------------------------------------------------------

## 📈 Resultados Observados

-   En consultas históricas amplias (tutorial original), TimescaleDB
    mostró mejor rendimiento.
-   En ventanas cortas (7 días), PostgreSQL presentó tiempos
    competitivos.
-   Ambos motores tuvieron 0% de errores.

------------------------------------------------------------------------

## 🧠 Conclusión Técnica

En sistemas IoT:

-   El volumen y patrón de acceso a los datos determinan el rendimiento.
-   TimescaleDB ofrece ventajas claras en análisis temporal intensivo.
-   PostgreSQL puede ser suficiente en escenarios de menor escala.

La elección del motor debe basarse en:

-   Frecuencia de escritura\
-   Tamaño histórico de los datos\
-   Tipo de consultas analíticas requeridas


------------------------------------------------------------------------

## 👥 Autores

Juan Esteban Mejia Izasa
Jazmin Natalia Cordoba Puerto
