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



# (4) Cuando se ejecuta el comando como ls -l, el sistema operativo, ¿llama a algún programa de usuario? ¿A alguna llamada al sistema? ¿Cómo se conecta esto con FUSE? ¿Qué funciones de su código se ejecutan finalmente?

1) El programa ls realiza varias llamadas al sistema, entre ellas **stat** (para obtener los metadatos de los archivo), **opendir** y **readdir** (para abrir y leer el directorio). 
2) Esas llamadas a sistema van a kernel, donde el sistema operativo, al ver que el volumen está montado al VFS con FUSE, le transfiere la llamada a sistema al FUSE, que busca en la estructura *fuse_operations* la función correspondiente a cada llamada a sistema.
3) Se ejecuta en modo usuario la función que nosotros implementamos para poder realizar esa llamada a sistema. En este caso, se ejecutan las funciones **fat_fuse_getattr**, **fat_fuse_opendir** y **fat_fuse_readdir** para las llamadas a sistema **stat**, **opendir** y **readdir** respectivamente, devolviendo su resultado a FUSE, que a su vez se lo entrega al SO y el SO se encarga de darselo a ls para que el programa funcione correctamente.


# (5) ¿Por qué tienen que escribir las entradas de directorio manualmente pero no tienen que guardar la tabla FAT cada vez que la modifican?
Porque las únicas veces que se escribe (y por lo tanto necesitamos guardar) la tabla FAT es para marcar clusters como ocupados (y generar cadenas de clusters ocupados). 

Una vez ya asignados los clusters correspondientes a un archivo, mientras no necesitemos agregar clusters (porque nos quedamos sin espacio en los actuales clusters y queremos ampliar el tamaño del archivo) no será necesario modificar la tabla FAT, para actualizar el contenido del archivo solo necesitamos modificar el contenido de los clusters asignados al archivo. 

# (6) Para los sistemas de archivos FAT32, la tabla FAT, ¿siempre tiene el mismo tamaño? En caso de que sí, ¿qué tamaño tiene?
La tabla FAT no tiene siempre el mismo tamaño, el tamaño depende de la cantidad de clusters que tenga el sistema archivos, lo cual depende del tamaño del dispositivo o la imagen en la cual estemos utilizando el sistema de archivos FAT. Lo que sí sucede es que mantiene su tamaño una vez el sistema de archivos FAT está implementado en el dispositivo o la imagen, no es una estructura que crezca o decrezca de tamaño.

El tamaño de la tabla FAT es (aproximadamente) 4 bytes * cantidad de clusters del sistema de archivos. Por ejemplo, supongamos que el sector de datos es de 64 GB, con clusters de 512 bytes, entonces la tabla fat ocupa 4B * (64GB/512B) = 512 MB aproximadamente