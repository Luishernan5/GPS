README - Modelos matemáticos implementados
=========================================

Este documento resume las fórmulas, algoritmos y funciones matemáticas implementadas en el proyecto "GPS - Rutas seguras". Está pensado para la memoria de un proyecto escolar: explica qué se usa, dónde está implementado y las limitaciones.

Estructura
---------
- services/motor_rutas.py  — selección y análisis de rutas.
- services/mapas.py        — integración con Google Maps, cálculos de costes y resumen de rutas.
- services/casetas.py      — detección y validación de casetas (peajes).
- services/zonas.py        — operaciones geométricas y detección de intersecciones con zonas.
- tests/*                  — pruebas unitarias que verifican estas implementaciones.

Modelos y fórmulas
-----------------
A continuación se listan las funciones matemáticas y algoritmos con sus ecuaciones y comentario de implementación.

1) Decodificación de polylines (Google encoded polyline)
   - Archivo: services/motor_rutas.py
   - Función: decodificar_polilinea(codificada)
   - Descripción: algoritmo de decodificación Delta + bitwise usado por Google para comprimir listas de coordenadas.
   - No requiere ecuación matemática aparte de operaciones bitwise y recontrucción acumulativa de lat/lng.

2) Distancia "Manhattan" aproximada sobre la superficie terrestre
   - Archivo: services/motor_rutas.py
   - Función: distancia_manhattan_km(punto_a, punto_b)
   - Fórmula (implementada por componentes):
       lat_km = |lat2 - lat1| * 111.32
       lon_km = |lon2 - lon1| * 111.32 * cos(lat_media)
       distancia_manhattan = lat_km + lon_km
   - Nota: 111.32 km/grado es una aproximación media. Se usa para pesos en el grafo.

3) Algoritmo Dijkstra (camino mínimo)
   - Archivo: services/motor_rutas.py
   - Función: aplicar_dijkstra(grafo, origen, destino)
   - Descripción: implementación típica con cola de prioridad (heapq). Calcula camino de coste mínimo (suma de pesos). Aquí el grafo es una cadena: cada nodo conectado al siguiente.

4) Selección de la mejor ruta
   - Archivo: services/motor_rutas.py
   - Función: seleccionar_mejor_ruta(rutas)
   - Procedimiento: cada alternativa enviada por Google se decodifica, se construye un grafo en cadena y se ejecuta Dijkstra; se selecciona la ruta con menor distancia Manhattan.

5) Fórmula de Haversine (distancia geodésica)
   - Archivo: services/zonas.py
   - Función: distancia_haversine_m(punto_a, punto_b)
   - Ecuación:
       R = RADIO_TIERRA_M
       dlat = lat2 - lat1
       dlng = lon2 - lon1
       a = sin(dlat/2)^2 + cos(lat1) * cos(lat2) * sin(dlng/2)^2
       distancia = 2 * R * asin(min(1, sqrt(a)))
   - Resultado en metros.

6) Proyección local lat/lng → coordenadas métricas (x,y)
   - Archivos: services/casetas.py, services/zonas.py
   - Funciones: _convertir_metros_locales, _a_xy
   - Fórmulas:
       x = rad(lon) * R * cos(rad(lat_ref))
       y = rad(lat) * R
   - Nota: lat_ref es la latitud de referencia (media local) para reducir distorsión.

7) Distancia punto ↔ segmento (proyección ortogonal)
   - Archivos: services/casetas.py ( _distancia_punto_segmento_con_proporcion ), services/zonas.py ( distancia_punto_segmento_m )
   - Descripción: convertir punto e extremos a (x,y), proyectar punto sobre el segmento, limitar la proporción a [0,1], calcular distancia euclidiana entre el punto y su proyección.
   - Devuelve también la proporción (posición longitudinal sobre el segmento) cuando es útil.

8) Interpolación lineal
   - Archivo: services/casetas.py
   - Función: _interpolar_punto(punto_a, punto_b, proporcion)
   - Fórmula: punto = A + proporcion * (B - A)

9) Acumulación de distancias a lo largo de la polilínea
   - Archivo: services/casetas.py
   - Función: _extremos_validacion_caseta(...) y funciones asociadas
   - Descripción: a partir de un índice fraccional (índice + proporción) se busca un intervalo anterior y posterior acumulando distancias hasta cubrir un radio en metros para validar tramos con la API.

10) Cálculos económicos y monetarios
    - Archivo: services/mapas.py y servicios asociados
    - Funciones: _valor_monetario(dinero), cálculo de litros/costo
    - Fórmulas:
        litros = distancia_km / rendimiento_km_l
        fuel_cost = litros * precio_gasolina_mxn
    - Conversión de estructuras {units, nanos} a float: unidades + nanos/1e9

Limitaciones y consideraciones
-----------------------------
- La aproximación plana (x,y) y el uso de cos(lat_media) son adecuados para tramos locales y medianas distancias, pero pueden degradarse en rutas muy largas o extremas (cercanas a los polos).
- El uso de distancia Manhattan como peso es una heurística: funciona pero puede diferir de la distancia real a lo largo de la carretera.
- La detección de casetas combina búsquedas de Places y validación por Routes; se aplican umbrales (60 m, 25 m) para evitar falsos positivos de carreteras paralelas.
- Los cálculos económicos asumen coherencia de moneda y disponibilidad de precios por parte de Google (si no existen precios se generan advertencias).

Pruebas (tests relevantes)
-------------------------
- tests/prueba_servicios.py contiene pruebas unitarias para:
  - decodificación de polylines
  - Dijkstra en cadena
  - selección de mejor ruta
  - distancia punto-ruta y detección de intersecciones
  - filtros de casetas y validación
- tests/prueba_api.py contiene pruebas de endpoints y CRUD de zonas.

Sugerencias para la memoria escolar
----------------------------------
- Incluir pequeñas ilustraciones: proyección local, proyección de punto sobre segmento y esquema de polyline.
- Comparar resultados de usar Manhattan vs haversine como peso de aristas (pequeña tabla con ejemplos).
- Documentar los casos límite: polylines muy cortas, ausencia de respuesta de Google, peajes sin precio.

---

Generado automáticamente por la herramienta de desarrollo del proyecto.
Si quieres, puedo:
- Añadir figuras (SVG) y ecuaciones en LaTeX dentro del README.
- Generar un PDF con la memoria corta.

