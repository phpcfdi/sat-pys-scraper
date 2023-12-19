# phpcfdi/sat-pys-scraper

[![Source Code][badge-source]][source]
[![PHP Version][badge-php-version]][php-version]
[![Latest Version][badge-release]][release]
[![Software License][badge-license]][license]
[![Build Status][badge-build]][build]
[![Coverage Status][badge-coverage]][coverage]

> Herramienta para obtener y generar un listado de las clasificaciones del catálogo de productos y servicios del SAT

:us: The documentation of this project is in Spanish, as this is the natural language for the intended audience.

Es posible que, lo único que buscas es el **Listado de clasificaciones de productos y servicios del SAT**, 
si es ese el caso, es mejor consumir el recurso [phpcfdi/resources-pys](https://github.com/phpcfdi/resources-pys), 
en donde el listado es actualizado automáticamente.

## Acerca de phpcfdi/sat-pys-scraper

El SAT en el sitio de internet <http://pys.sat.gob.mx/PyS/catPyS.aspx> tiene publicada una clasificación de productos y servicios. 
Esta clasificación no pertenece oficialmente a los catálogos y no se encuentra publicada en ningún lugar.

Esta herramienta hace el *scrap* del sitio mencionado para obtener los 4 niveles de clasificación: Tipo, Segmento, Familia y Clase. 
Igualmente, la estructura se puede exportar como XML o como JSON.

## Instalación usando composer

A diferencia de otras librerías o componentes, este proyecto es una herramienta, por lo que probablemente nunca tengas que 
instalar el proyecto como una dependencia. Sin embargo, se puede hacer para que realices la parte de obtener las 
clasificaciones del sitio del SAT, pero tú mismo te encargues de procesar la estructura y usarla para tus propios 
propósitos, como por ejemplo, almacenar en una base de datos.

```shell
composer require phpcfdi/sat-pys-scraper
```

## Instalación usando Docker

Este proyecto provee un archivo `Dockerfile` para construir una imagen con todas sus dependencias. 
Se puede usar esta imagen para correr de forma local, para más información consulte 
el archivo [`README.Docker.md`](Docker.README.md).

```shell
# clonado del proyecto
git clone https://github.com/phpcfdi/sat-pys-scraper.git

# construcción de la imagen de Docker
docker build -t sat-pys-scraper sat-pys-scraper/

# ejecución de la herramienta
docker run -it --rm sat-pys-scraper --help
```

### Ayuda de `sat-pys-scraper` (script)

```text
sat-pys-scraper - Crea un archivo con la clasificación de productos y servicios del SAT.

Sintaxis:
    sat-pys-scraper help|-h|--help
    sat-pys-scraper destination-file [--quiet|-q] [--format|-f FORMAT]

Argumentos:
    destination-file
        Nombre del archivo XML para almacenar el resultado.
        Si se usa "-" o se omite entonces el resultado se manda a la salida estándar
        y se activa el modo de operación silencioso.
    --format|-f FORMAT
        Establece el formato de salida, default: xml, por el momento "xml" o "json".
    --sort|-s SORT
        Establece el orden de elementos, default: key, se puede usar "key" o "name".
    --quiet|-q
        Modo de operación silencioso.

Acerca de:
    Este script pertenece al proyecto https://github.com/phpcfdi/sat-pys-scraper
    y mantiene la autoría y licencia de todo el proyecto.
```

## Uso de la herramienta

Si usar el código de la herramienta, entonces es importante entender que la tarea trata de dos pasos:

1. Obtener del sitio del SAT el listado de tipos, segmentos, familias y clases.
2. Exportar el listado a un formato específico.

Para generar el listado de tipos, segmentos, familias y clases se usa el objeto `Generator`, que a su vez usa un 
objeto `Scraper` para realizar la descarga de información, que a su vez utiliza un objeto `Client` de `GuzzleHttp`.

En el siguiente ejemplo se muestra cómo generar la estructura e iterar sobre sus elementos.

- Al ejecutar `Generator::generate()` se devuelve un objeto de tipo `Types`.
- Se recorre la estructura con `foreach`.
- Se puede exportar usando `XmlExporter::export()`.

```php
<?php
use GuzzleHttp\Client;
use PhpCfdi\SatPysScraper\Generator;
use PhpCfdi\SatPysScraper\Scraper;
use PhpCfdi\SatPysScraper\XmlExporter;

$scraper = new Scraper(new Client());
$generator = new Generator($scraper);
$types = $generator->generate();
$types->sortByKey();

foreach ($types as $type) {
    printf("Tipo: %s - %s\n", $type->key, $type->name);
    foreach ($type as $segment) {
        printf("  Segmento: %s - %s\n", $segment->key, $segment->name);
        foreach ($segment as $family) {
            printf("    Familia: %s - %s\n", $family->key, $family->name);
            foreach ($family as $class) {
                printf("      Clase: %s - %s\n", $class->key, $class->name);
            }
        }
    }
}

$exporter = new XmlExporter();
$exporter->export('output.xml', $types);
```

### Tipos de datos

Un objeto `Types` es una colección iterable de objetos de tipo `Type`.
Un objeto `Type` contiene las propiedades `key` y `name`, y además es una colección iterable de objetos de tipo `Segment`.
Un objeto `Segment` contiene las propiedades `key` y `name`, y además es una colección iterable de objetos de tipo `Family`.
Un objeto `Family` contiene las propiedades `key` y `name`, y además es una colección iterable de objetos de tipo `Classification`.
Un objeto `Classification` solamente contiene las propiedades `key` y `name`.

Todos los objetos de datos implementan `JsonSerializable`, por lo que puedes usar esta característica para exportar a formato JSON.

## Soporte

Puedes obtener soporte abriendo un ticket en Github.

Adicionalmente, esta librería pertenece a la comunidad [PhpCfdi](https://www.phpcfdi.com),
así que puedes usar los canales oficiales de comunicación para obtener ayuda de la comunidad.

## Compatibilidad

Esta librería se mantendrá compatible con al menos la versión con
[soporte activo de PHP](https://www.php.net/supported-versions.php) más reciente.

También utilizamos [Versionado Semántico 2.0.0](docs/SEMVER.md) por lo que puedes usar esta librería
sin temor a romper tu aplicación.

| Version | PHP | Notes      |
|---------|-----|------------|
| 1.0.0   | 8.2 | 2023-12-13 |

## Contribuciones

Las contribuciones con bienvenidas. Por favor lee [CONTRIBUTING][] para más detalles
y recuerda revisar el archivo de tareas pendientes [TODO][] y el archivo [CHANGELOG][].

## Copyright and License

The `phpcfdi/sat-pys-scraper` tool is copyright © [PhpCfdi](https://www.phpcfdi.com/)
and licensed for use under the MIT License (MIT). Please see [LICENSE][] for more information.

[contributing]: https://github.com/phpcfdi/sat-pys-scraper/blob/main/CONTRIBUTING.md
[changelog]: https://github.com/phpcfdi/sat-pys-scraper/blob/main/docs/CHANGELOG.md
[todo]: https://github.com/phpcfdi/sat-pys-scraper/blob/main/docs/TODO.md

[source]: https://github.com/phpcfdi/sat-pys-scraper
[php-version]: https://github.com/phpcfdi/sat-pys-scraper/blob/main/composer.json
[release]: https://github.com/phpcfdi/sat-pys-scraper/releases
[license]: https://github.com/phpcfdi/sat-pys-scraper/blob/main/LICENSE
[build]: https://github.com/phpcfdi/sat-pys-scraper/actions/workflows/build.yml?query=branch:main
[coverage]: https://scrutinizer-ci.com/g/phpcfdi/sat-pys-scraper/code-structure/main/code-coverage

[badge-source]: https://img.shields.io/badge/source-phpcfdi/sat--pys--scraper-blue?style=flat-square
[badge-php-version]: https://img.shields.io/packagist/dependency-v/phpcfdi/sat-pys-scraper/php?style=flat-square
[badge-release]: https://img.shields.io/github/release/phpcfdi/sat-pys-scraper?style=flat-square
[badge-license]: https://img.shields.io/github/license/phpcfdi/sat-pys-scraper?style=flat-square
[badge-build]: https://img.shields.io/github/actions/workflow/status/phpcfdi/sat-pys-scraper/build.yml?branch=main&style=flat-square
[badge-coverage]: https://img.shields.io/scrutinizer/coverage/g/phpcfdi/sat-pys-scraper/main?style=flat-square
