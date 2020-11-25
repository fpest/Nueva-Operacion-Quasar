# Nueva-Operacion-Quasar

El desarrollo está hecho en 3 niveles.
1er Nivel. Programa desarrollado en Visual Basic donde se ingresan las palabras de los mensajes que recibe cada uno de los satélites y la distancia de éstos a la nave. Se devuelve el mensaje completo y la ubicación de la nave si se pueden resolver.
2do Nivel. Se desarrolló una API en PHP (www.ncdsanluis.com.ar/topsecret/nivel2.php) que recibe información a traves de un HTTP POST con el siguiente formato:

```
 {
    "satellites": [
        {
            "name": "kenobi",
            "distance": 485.69,
            "message": [
                "hola",
                "",
                "estan",
                "",
                ""
            ]
        },
        {
            "name": "skywalker",
            "distance": 266.08,
            "message": [
                "",
                "como",
                "",
                "todos",
                "",
                ""
            ]
        },
        {
            "name": "sato",
            "distance": 600.5,
            "message": [
                "hola",
                "",
                "",
                "todos",
                "los",
                "edificios"
            ]
        }
    ]
}
```
y devuelve, si los datos son suficientes para calcularlos, la ubicación de la nave y el mensaje completo con en el siguiente formato:

```
{
    "possition": {
        "x": -100.003685625,
        "y": 75.50316225
    },
    "message": "hola como estan todos los edificios "
}
```
3er Nivel. Cómo en el caso del Nivel 2 también se desarrollón en PHP las diferencia es que el mensaje se recibe en diferentes POST a una url distinta a la anterior(www.ncdsanluis.com.ar/topsecret_split/nivel3.php?satellite_name=[nombre del satélite]) con la información de cada satélite en un formato como el siguiente:
```
{"distance": 485.69,
"message": [
    "este",
    "",
    "un",
    ""
]
}
```
esta información se va archivando en el servidor hasta que haya información suficiente para calcular y devolver como respuesta la ubicación de la nave y el mensaje completo. (www.ncdsanluis.com.ar/topsecret_split/nivel3.php). La respuesta tendrá el siguiente formato:
```
{
    "position": {
        "x": -99.9968859258,
        "y": 75.4895628516
    },
    "message": "este es un mensaje "
}
```
# Consideraciones
1) Se considera que el mensaje puede tener un desfazaje en las palabras recibidas.
2) No se conoce la cantidad de palabras en cada mensaje recibido por los satélites.
3) Con el fin de verificar que los datos de las distancias de los satélites a la nave estén bien, se calcula una vez conocida la ubicación de la nave las distancias de cada satélite a la nave y se compara con los datos, se considera para la comparación una tolerancia del 1%.
4) Las coordenadas de los satélites son las siguentes:
		S1: Kenobi(-500-200)
		S2: Skywalker(100,-100)
		S3: Sato(500,100) 
5) El desarrollo del cálculo de la ubicación de la nave es el siguiente:
```
d1 = distancia desde S. Kenobi a Nave
d2 = distancia desde S. Skywalker a Nave
d3 = distancia desde S. Sato a Nave

(x1,y1) Coordenadas del Satélite 1 (este satélite lo llevamos a (0,0))
(x2,y2) Coordenadas del Satélite 2 (considerando S1 en (0,0))
(x3,y3) Coordenadas del Satélite 3 (considerando S1 en (0,0))

(x0,y0) Será la ubicación de la Nave calculada.

(El cálculo se basa en que en algún punto de la circunferencia con un radio con la distancia d1 en el caso del satélite 1, d2 y d3 en los otros dos satélites se debe encontrar la nave.
Por lo tanto en el punto de corte de las tres circunferencias debe estar la nave)

d1^2 = x0^2 + y0^2 (por poner el s1 en el origen es es la distancia a la nave desde s1)

d2^2 = (x0-x2)^2 + (y0-y2)^2

     = x0^2 - 2x0x2 + x2^2 + y0^2 - 2y0y2 + y^2
     = x0^2 + y0^2 + x2^2 + y2^2 - 2x0x2 - 2y0y2

d1^2 - d2^2 = -x2^2 - y2^2 + 2x0x2 + 2y0y2

d1^2 - d2^2 + x2^2 + y2^2 - 2y0y2 = 2x0x2

por lo tanto:

x0 = (d1^2 - d2^2 + x2^2 + y2^2 - 2y0y2)/(2 x2)

* lo anterior relaciona el S1 con el S2, lo mismo para la relación entre S1 y S3. Por lo tanto:

x0 = (d1^2 - d3^2 + x3^2 + y3^2 - 2y0y3)/(2 x3)

*igualendo:

(d1^2 - d2^2 + x2^2 + y2^2 - 2y0y2)/(2 x2)=(d1^2 - d3^2 + x3^2 + y3^2 - 2y0y3)/(2 x3)

*desde aqui se despeja y0 quedando

y0 = A/B 

donde 

A= (x3/x2)(d1^2 - d2^2 + y2^2) + x3x2 - d1^2 + d3^2 - x3^2 - y3^2

B = (2x3y3/y2)-(2y3)

y x0 queda:

x0 = (d1^2 - d3^2 + x3^2 + y3^2 - 2y0y3)/(2 x3)

