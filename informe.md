# (1) Cuando se ejecuta el main con la opción -d, ¿qué se está mostrando en la pantalla?
El -d significa modo debugging, por lo que se muestra una descripción de todas las operaciones que se realizan sobre la imagen FAT que montamos

# (2) ¿Hay alguna manera de saber el nombre del archivo guardado en el cluster 157?
Si el archivo que guarda su contenido en el cluster 157 está dentro del árbol de directorios, existe una manera de saber el nombre, pero es un proceso largo.

1) (En la FAT Table) Leer en la FAT table el indice del cluster del directorio raíz (usualmente el 2)
2) (En la FAT Table) Analizar si el directorio raíz finaliza ahí o si utiliza otros clusters.
   1) Si utiliza otros clusters además de su cluster inicial, ir "recorriendo" los indices de los clusters utilizados por el directorio raíz hasta llegar al indice del último cluster (Que está marcado con el código de final de la cadena de clusters). Si mientras "recorriamos" los indices de los clusters pasamos por el cluster 157, entonces significa que el directorio raíz es el que está guardado en el cluster 157, caso contrario saltar al paso 3.
   2) Si no se utilizan otros clusters, entonces saltar al paso 3.
3) (En la zona de datos/clusters) Acceder al cluster/a los clusters correspondientes al directorio raíz
4) Leer cada entrada de directorio para obtener: nombre de archivo, tipo de archivo y su cluster incial, y repetir los pasos 1 y 2 en caso de ser un archivo o los pasos 1,2,3 y 4 en caso de ser un directorio. Repetir este proceso de manera recursiva hasta encontrar el archivo que utiliza el cluster 157 para guardar su contenido.

# (3) ¿Dónde se guardan las entradas de directorio? ¿Cuántos archivos puede tener adentro un directorio en FAT32?
Las entradas de directorio se guardan en el cluster (o los clusters en caso de ser varios) asignado al directorio.

Luego de una investigación, descubrimos que en la práctica, un directorio en FAT32 puede tener 2^16 - 2 = 65,534 archivos cuyo nombre sea corto (8 caracteres el nombre y 3 la extensión) (El -2 es por la entrada de "." y "..").

Pero teóricamente, el tamaño máximo de un archivo es 2^32 - 1 bytes (este límite es una consecuencia de que la entrada de tamaño de archivo en fat_dir_entry sea de 32 bits)...
Por lo que si creamos un directorio de ese tamaño, podriamos tener (2^32 - 1) /32 = 134,217,727 entradas de directorio (de 32 bytes), por lo que podríamos tener 134,217,725 = 2^27 - 3 archivos en un directorio.

AUN NO SABEMOS A QUE SE DEBE ESTA DISCREPANCIA

# (4) Cuando se ejecuta el comando como ls -l, el sistema operativo, ¿llama a algún programa de usuario? ¿A alguna llamada al sistema? ¿Cómo se conecta esto con FUSE? ¿Qué funciones de su código se ejecutan finalmente?


# (5) ¿Por qué tienen que escribir las entradas de directorio manualmente pero no tienen que guardar la tabla FAT cada vez que la modifican?


# (5) Para los sistemas de archivos FAT32, la tabla FAT, ¿siempre tiene el mismo tamaño? En caso de que sí, ¿qué tamaño tiene?
