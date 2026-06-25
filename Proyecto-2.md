# Proyecto-2

Sistemas Operativos CS3015 2026-1
Proyecto 2: Memory Mappings en Selfie
Profesores Jorge Gonzalez, Mauricio Pinto
TAs Mariana Capu˜ nay
1 Implementar la syscallmmapenSelfie
La syscallmmappermite crear un mapping entre una regi´ on de memoria virtual de un proceso y una regi´ on de un
archivo. En esta implementaci´ on simplificada para Selfie, el mapping deber´ a asociar un rango de direcciones vir-
tuales del proceso con p´ aginas f ´ ısicas almacenadas en elpage cache. Estas p´ aginas f ´ ısicas contendr´ an el contenido
del archivo a partir de un determinado offset. De esta manera agregamos la posibilidad de que m´ ultiples procesos
compartan acceso a un archivo sin necesidad de acceder a memoria secundaria. Esta relaci´ on deber´ a registrarse
en el contexto del proceso mediante un atributomappings. Cada entrada demappingsdescribir´ a un rango de
direcciones virtuales del proceso y el archivo asociado a dicho rango.
Para este proyecto, se debe considerar el tama˜ no de p´ agina utilizado por Selfie, definido porPAGESIZE. En la
implementaci´ on base,PAGESIZEcorresponde a 4096 bytes. Este valor deber´ a usarse para calcular la cantidad de
p´ aginas necesarias, alinear direcciones y validar offsets.
La llamada propuesta paraSelfiepuede definirse usando valores de tipouint64 t:
uint64˙t mmap(uint64˙t addr, uint64˙t length, uint64˙t prot, uint64˙t fd, uint64˙t offset);
Donde:
•addr: direcci´ on virtual inicial donde se desea crear el mapping. Siaddr = 0, Selfie puede elegir una regi´ on
virtual libre.
•length: cantidad de bytes que se desea mapear. Internamente, este valor puede redondearse hacia arriba al
m´ ultiplo dePAGESIZEm´ as cercano.
•prot: permisos del mapping. Por ejemplo,0para lectura,1para escritura y2para lectura/escritura.
•fd: file descriptor (identificador) del archivo cuyo contenido ser´ a mapeado.
Nota
EnSelfie, elfile descriptor(fd) es un identificador local al proceso. Por ello, un mismo valor defd
no identifica globalmente al mismo archivo en todos los procesos. Dos procesos podr ´ ıan tener valores
distintos defdpara referirse al mismo archivo, o incluso el mismo valor defdpara referirse a archivos
diferentes.
•offset: posici´ on inicial dentro del archivo desde donde comenzar´ a el mapping. Se recomienda que sea
m´ ultiplo dePAGESIZE.
Importante
La syscallmmapno debe escribir cambios en el archivo al momento de crear el mapping. Su funci´ on principal
es crear la relaci´ on entre memoria virtual, tabla de p´ aginas ypage cache.
Los cambios realizados por el proceso sobre la memoria mapeada se almacenan inicialmente en loscache
frames. Para este proyecto, dichos cambios se escribir´ an en el archivo ´ unicamente cuando el proceso invoque
msync. Para comparar esta implementaci´ on simplificada con el comportamiento real en Linux, se recomienda
revisar la documentaci´ on oficial demmap, la cual est´ a disponible enhttps://man7.org/linux/man-pages/man2/
mmap.2.html.
Junio, 2026


Importante
Para este proyecto,considerar que cada test trabajar´ a con mappings asociados a un ´ unico archivo.
Espacio de direcciones
virtual del proceso
0x0000
0xFFFF
...
...
mmap
(addr, length)
Tabla de
p´ aginas
...
...
•
•
•
Cache frames
en memoria f ´ ısica
frame F0
frame F1
frame F2
...
Page cache
entries
(fileid, 0) -¿ F0
(fileid, 4096) -¿ F1
(fileid, 8192) -¿ F2
...
Archivo
file page 0
offset 0
file page 1
offset 4096
file page 2
offset 8192
...
mmapcrea entradas encontext-¿mappingsy actualiza la tabla de
p´ aginas para que las direcciones virtuales apunten acache frames. El
page cachepermite encontrar esos frames usandofileidyoffset.
Figure 1: Relaci´ on entre memoria virtual, tabla de p´ aginas,page cache, frames f ´ ısicos y archivo en una imple-
mentaci´ on simplificada demmap.
2 Implementar syscallmunmapenSelfie
La syscallmunmapelimina un mapping previamente creado. Para este proyecto, deber´ a definirla como:
uint64˙t munmap(uint64˙t addr);
A diferencia de Linux, esta versi´ on simplificada no recibelength. Por ello,addrdebe coincidir con la direcci´ on
virtual inicial de un mapping previamente registrado encontext-¿mappings.
En Linux,munmaprecibe tanto la direcci´ on inicial como el tama˜ no del rango a desmapear; por lo tanto, se
recomienda revisar la documentaci´ on oficial (disponible enhttps://man7.org/linux/man-pages/man2/mmap.2.
html) para entender la diferencia con la versi´ on reducida propuesta en este proyecto.
En esta versi´ on simplificada de Selfie,munmapno debe escribir los cambios en el archivo asociado. Su funci´ on
ser´ a eliminar la relaci´ on entre el rango de memoria virtual del proceso y las p´ aginas f ´ ısicas delpage cache. Por lo
tanto, si el proceso modifica una regi´ on mapeada y luego invoca ´ unicamentemunmap, dichos cambios no deber´ an
persistir en el archivo. La escritura de cambios al archivo deber´ a realizarse ´ unicamente mediantemsync.
3 ImplementarmsyncenSelfie
La syscallmsyncdebe permitir que los cambios realizados sobre una regi´ on mapeada en memoria se escriban de
vuelta en el archivo asociado. En este proyecto,msyncser´ a el ´ unico mecanismo encargado de persistir en el archivo
los cambios hechos sobre una regi´ on mapeada.
Nota
Pueden utilizar la syscallwritepara realizar la actualizaci´ on del archivo o crear una nueva syscallmsync.
En Linux,msyncsincroniza con el sistema de archivos los cambios hechos sobre una regi´ on previamente mapeada
conmmap. Una versi´ on simplificada de esta syscall en Selfie es:
uint64˙t msync(uint64˙t addr);
En esta versi´ on,addrdebe coincidir con la direcci´ on virtual inicial de un mapping previamente registrado en
context-¿mappings. A partir de esta direcci´ on, Selfie debe identificar el archivo asociado, el offset inicial y el
tama˜ no del mapping.
Para comparar esta implementaci´ on simplificada con el comportamiento real en Linux, se recomienda revisar la
documentaci´ on oficial demsync:https://man7.org/linux/man-pages/man2/msync.2.html
2

4 Pasos sugeridos
1. Agregar un atributomappingsal contexto; para cada mapping debe guardar como m ´ ınimo:
•Address inicial para mapear el page virtual
•Offset desde el que se hace el mapping del archivo.Se recomienda que el offset sea divisible por
PAGESIZE.
•Tama˜ no del mapping.
•File descriptor o identificador del archivo asociado al mapping.
2. Crear un espacio reservado para loscache frames. Podemos crear un bloque de memoria separado de la
memoria principal de selfie. Para ello, guiarnos de la funci´ oninit memory.
3. Crear una lista de entries delpage cache. Para ello podemos:
•Crear un bloque de memoria fijo de un tama˜ no razonable para almacenarentries.
•Crear una lista enlazada para almacenarentries.
Cadaentrydebe contener la informaci´ on ilustrada en la Tabla 4
Hint
Puedes revisar la implementaci´ on de las syscallsreadowritepara comprender la lectura y escritura de
archivos desdeSelfie.
Nota
Uncache framees una p´ agina f ´ ısica. A diferencia de las p´ aginas virtuales, loscache framesno pertenecen
a un proceso espec ´ ıfico. Estos pueden ser compartidos por varios procesos cuando sus mappings apuntan a
la misma p´ agina de un archivo. Para identificar qu´ e contenido almacena cadacache frame, elpage cache
mantiene entradas asociadas al archivo y al offset de p´ agina correspondiente.
File ID File page offset Cache frame
23 0 P0
23 4096 P1
23 8192 P2
Table 1: Ejemplo de entradas almacenadas en elpage cache.
Cada entrada delpage cacheindica qu´ ecache framecontiene una p´ agina espec ´ ıfica del archivo.Ese frame
puede ser compartido por varios procesos si sus mappings apuntan al mismo archivo y al mismo offset de
p´ agina.
Nota
Pueden mapear el archivo completo, y en el mapping de cada proceso s´ olo mantener una referencia a la
porci´ on mapeada por ese proceso.
Al acceder a una direcci´ on presente en un mapeo creado conmmap, se debe realizar la traducci´ on a direcci´ on
f ´ ısica de la siguiente manera:
•Verificamos si una direcci´ on se encuentra dentro del rango de unmapping.
•Si no est´ a dentro del rango, procedemos a traducir la memoria de manera normal. Sin embargo, si est´ a
dentro de unmapping, debemos:
–Hallar elcache frameal que se encuentra mapeada lapage.
–Sumar el offset de la direcci´ on base delcache frame.
3

Importante
Se les entregar´ a una versi´ on funcional de fork-wait. Si un proceso realizaforkdespu´ es de haber creado
un mapping, el proceso hijo debe heredar los mappings del proceso padre. Las p´ aginas f ´ ısicas delpage cache
no deben duplicarse. Tanto el padre como el hijo deben mapear sus p´ aginas virtuales a los mismoscache
frames.
5 Entregables
1. Carpeta .zip con:
•Archivoselfie.c
•Archivos de test
2. Tests que validen la correctitud de su implementaci´ on:
•Al menos un test en el cu´ al se cree unmappingde un archivo y se lea su contenido directamente de
memoria.
•Al menos un test en el cu´ al se cree unmappingde un archivo y se escriba sobre este, mostrando
posteriormente el cambio en memoria secundaria.
•Al menos un test en el cu´ al dos procesos crean unmapping del mismo archivo y lo modifican, siendo
capaces de observar estos cambios en memoria.
3. Presentaci´ on de 12 minutos donde presentan brevemente sus ideas, metodolog ´ ıa y muestran una demo de
sus tests. Las diapositivas tambi´ en ser´ an calificadas.
Importante
Se revisar´ an peri´ odicamente sus avances durante sesiones de clase. Estas revisiones tomar´ an parte de
la nota.
4