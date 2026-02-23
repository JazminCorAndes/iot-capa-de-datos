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
from django.db.models import Avg
from django.db.models.functions import TruncDate
from django.http import JsonResponse
from .models import Data

def daily_average(request):
    results = (
        Data.objects
        .annotate(day=TruncDate('timestamp'))
        .values('day', 'city')
        .annotate(avg_temp=Avg('value'))
        .order_by('day')
    )
    return JsonResponse(list(results), safe=False)
```

------------------------------------------------------------------------

## ⏱ Implementación en TimescaleDB

``` python
from django.db import connection
from django.http import JsonResponse

def daily_average(request):
    with connection.cursor() as cursor:
        cursor.execute("""
            SELECT time_bucket('1 day', timestamp) AS day,
                   city,
                   AVG(value) AS avg_temp
            FROM data
            WHERE timestamp >= NOW() - INTERVAL '7 days'
            GROUP BY day, city
            ORDER BY day;
        """)
        rows = cursor.fetchall()

    results = [
        {"day": row[0], "city": row[1], "avg_temp": row[2]}
        for row in rows
    ]

    return JsonResponse(results, safe=False)
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
