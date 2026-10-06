# Diseño y validación de resultados de Liga MX con XML y DTD

## Propósito
Representar resultados y estadísticas de los partidos del domingo 27 de septiembre de 2026 y validar su estructura con una DTD externa.

## Estructura del repositorio
- README.md
- xml/resultados.xml
- xml/resultados-invalido.xml: prueba intencional con equipo visitante ausente.
- dtd/resultados.dtd

## Resultados de la jornada 10, Apertura 2026
| Local | Marcador | Visitante |
|---|---|---|
| Pumas UNAM | 2-3 | Atlético de San Luis |
| León | 2-1 | FC Juárez |
| Necaxa | 2-4 | América |

Se incluyen únicamente los partidos del domingo, no los del viernes y sábado. Se utiliza la fecha local de México: los juegos nocturnos de León y Necaxa aparecen el 28 de septiembre en UTC, pero corresponden al domingo 27 en sus sedes.

## Modelo y respuestas de análisis
El elemento raíz es liga. Una liga contiene una o más jornadas y cada jornada contiene uno o más partidos. Cada partido tiene exactamente un equipoLocal y un equipoVisitante. El marcador separa golesLocal y golesVisitante. Las estadísticas pertenecen a cada equipo y se agrupan en local y visitante, cuyo orden es obligatorio.

Son obligatorios los equipos, el marcador y los atributos de identificación, fecha y estado. El estadio, las estadísticas y la fuente de cada partido son opcionales. Una estadística ausente significa que no fue registrada, no que su valor sea cero.

## Elementos y atributos
| Información | Representación | Justificación |
|---|---|---|
| Jornada | Elemento jornada | Agrupa partidos |
| Fecha | Atributo fecha | Identifica la fecha de jornada y partido |
| ID del partido | Atributo id | Identificador único |
| Equipo local | Elemento equipoLocal | Nombre del equipo |
| Equipo visitante | Elemento equipoVisitante | Nombre del equipo |
| Goles | Elementos golesLocal y golesVisitante | Facilitan consultas separadas |
| Estadio | Elemento estadio | Dato opcional del partido |
| Estado del partido | Atributo estado | Valor de un conjunto enumerado |
| Posesión | Elemento posesion | Porcentaje del equipo |
| Tarjetas | Elemento tarjeta con tipo y cantidad | Permite registrar amarillas y rojas |

## Estadísticas seleccionadas
Posesión en porcentaje, tiros totales, tiros a puerta, faltas y tiros de esquina. Se agrupan por equipo para evitar repetir sus nombres en cada estadística y facilitar consultas. El modelo permite tarjetas, pero no se incluyen cantidades que no estén disponibles en los datos consultados.

## Cardinalidades y atributos
| Regla | Expresión DTD |
|---|---|
| Una o más jornadas | liga (jornada+) |
| Uno o más partidos | jornada (partido+) |
| Exactamente un local y un visitante | partido (equipoLocal,equipoVisitante,marcador,estadio?,estadisticas?) |
| Estadística opcional | posesion? |
| Cero o más tarjetas | tarjeta* |

id se declara ID y #REQUIRED: exige un identificador presente, con sintaxis de ID XML y único en todo el documento. CDATA no detectaría duplicados. fuente utiliza #IMPLIED y puede omitirse. estado es una enumeración obligatoria.

La ruta desde xml/resultados.xml al DTD es ../dtd/resultados.dtd.

## Validación
Desde la carpeta raíz del proyecto, con xmllint instalado:
```bash
xmllint --noout --valid xml/resultados.xml
xmllint --noout --valid xml/resultados-invalido.xml
```
El primer comando debe terminar sin errores; el segundo debe detectar el equipo visitante ausente.

Se verificó el documento correcto y se probaron los seis errores por separado mediante lxml.etree. Los seis documentos estaban bien formados y fueron rechazados por la DTD. El archivo inválido guardado contiene únicamente la primera prueba.

| Prueba | ¿Bien formado? | ¿Válido? | Error detectado |
|---|---|---|---|
| Falta visitante | Sí | No | Element partido content does not follow the DTD, expecting (equipoLocal , equipoVisitante , marcador , estadio? , estadisticas?), got (equipoLocal marcador estadio estadisticas) |
| Dos locales | Sí | No | Element partido content does not follow the DTD, expecting (equipoLocal , equipoVisitante , marcador , estadio? , estadisticas?), got (equipoLocal equipoLocal equipoVisitante marcador estadio estadisticas) |
| Orden incorrecto | Sí | No | Element partido content does not follow the DTD, expecting (equipoLocal , equipoVisitante , marcador , estadio? , estadisticas?), got (equipoVisitante equipoLocal marcador estadio estadisticas) |
| Falta atributo obligatorio | Sí | No | Element partido does not carry attribute estado |
| ID duplicado | Sí | No | ID P001 already defined |
| Elemento desconocido | Sí | No | No declaration for element desconocido |

XML bien formado significa que cumple la sintaxis XML; XML válido significa que además cumple la DTD. La DTD no comprueba rangos numéricos, fechas reales, porcentajes que sumen 100 ni coherencia deportiva del marcador; requieren validación adicional.

## Decisiones ante cambios
Se permitió omitir estadísticas no disponibles mediante ?. Se contemplaron tarjetas mediante tarjeta*. Los tres partidos reales caben en el modelo sin modificarlo. Si se agrega una nueva estadística, se debe declarar su elemento y actualizar el modelo de contenido antes de usarla.

## Fuentes
Consulta: 6 de octubre de 2026.
- [Resultados ESPN México del 27 de septiembre](https://www.espn.com.mx/futbol/resultados/_/liga/mex.1/fecha/20260927).
- [Datos JSON de ESPN](https://site.api.espn.com/apis/site/v2/sports/soccer/mex.1/scoreboard?dates=20260927): marcadores, estadios y estadísticas. Se utilizó este servicio público de ESPN porque la página de resultados mostró una verificación de JavaScript.
- [Línea Directa: resultados del domingo](https://lineadirectaportal.com/deportes/america-leon-y-san-luis-suman-puntos-importantes-en-la-liga-mx-2026-09-27__1710004): corroboración de los tres marcadores.

## Subir a GitHub
Crear un repositorio vacío llamado liga-mx-xml. Desde esta carpeta:
```bash
git init
git add README.md
git commit -m "Documentación del modelo XML"
git add dtd/resultados.dtd
git commit -m "Definición de DTD y atributos"
git add xml/resultados.xml
git commit -m "Resultados y estadísticas del 27 de septiembre de 2026"
git add xml/resultados-invalido.xml
git commit -m "Prueba negativa de validación"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/liga-mx-xml.git
git push -u origin main
```
Sustituir TU_USUARIO por el usuario de GitHub. Estos comandos agrupan los archivos finales; no representan un historial de actividades que no se haya realizado. Entregar el enlace real del repositorio después de subirlo.
