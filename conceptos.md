# Conceptos mas importantes de Git
# Manual de estudio personal.

## Git Graph Model
- GRAPH: Es una forma de modelar la conexión entre nodos mediante aristas o líneas.
- Directed GRAPH: Es un modelado en que las aristas apunta a un nodo y define la relación entre los nodos.
- DAG (Directed Acyclic GRAPH): Es un modelo en que los nodos estan conectados mediantes flechas que no forman un ciclo.

Git es un modelo DAG, en que los nodos establecen la relación con el Padre o nodo predecesor.

## Objetos de Git

- **Commit Object**: Es una estructura de datos que almacena una instantánea (snapshot) del estado de todo el proyecto en un momento específico.
- **Tag**: Es un objeto que apunta a un commit específico para marcar hitos importantes como versiones.
   - **Lightweight tag**: Es un simple puntero, no almacena información adicional -> `git tag <nombre>`.
   - **Annotated tag**: Es un objeto completo que almacenan metadatos como el autor, email, fecha, etc. -> `git tag -a <nombre> -m "mensaje"`.
- **Tree**: Es un objeto que contiene una lista de archivos y directorios dentro de un directorio.
- **Blob**: Es un objeto que almacena el contenido de un archivo que esta siendo gestionado por Git.

## Git ID
Es el nombre de un objeto Git, esta compuesto por una cadena hexadecimal de 40 caracteres.\
También se le conoce como SHA-1, hash y checksum.
Un pequeño cambio produce un gran cambio en el SHA-1.

`git log`: mostrará el SHA-1 completo de los objetos en el repositorio.\
`git log --oneline`: mostrará los primeros 7 caracteres del SHA-1.

## Branch label
Es una etiqueta o rótulo que apunta a un commit específico dentro de una rama.\
La rama principal puede denominarse main o master (por convención).\
Se puede crear nuevas ramas a partir de un commit específico, para no afectar a la rama principal.\
Cada vez que se realiza un commit en dicha rama, el puntero avanza automáticamente hacia adelante.

## HEAD
Es una referencia al commit actual.\
Usualmente apunta a la rama actual.\
Solo un HEAD por repositorio.

## Detached HEAD
Es cuando un HEAD no apunta o referencia a una etiqueta de rama, sino que directamente a un SHA-1 de un commit.\
Esto se produce cuando se realiza un `git checkout` a un ID de un commit anterior.\
Para corregirlo se crea una nueva rama a partir del commit en que está el HEAD, y se realiza el chekout a partir de ese punto.

## Fast-Forward Merge
Al realizar un merge, el fast-forward se realiza cuando no hay commits nuevos desde que se creó la rama de origen.\
No se crea un nuevo commit, simplemente mueve el puntero de la rama destino (main) hacia adelante para que apunte al mismo commit de la rama origen (feature).\
El resultado es un historial de commits totalmente lineal.

## Merge Commit
Se realiza cuando las dos ramas que se están uniendo tienen nuevos commits desde que se separaron.\
En esta combina ambos commits, generando un nuevo commit con las modificaciones de ambas ramas.\
Puede fallar si el cambio en ambas ramas es en la misma línea o muy cercano entre ellas.\
Git avisa de algun inconveniente en el merge commit para su corrección.\
En este caso se muestra las dos ramas que existieron, por lo que el commit generado en este punto tendrá dos padres.

## Dangling commit
Esto sucede cuando se elimina una rama secundaria que contenía commits, estos quedan en el repositorio pero sin etiquetas de marca.\
Cada cierto tiempo Git realiza una limpieza de estos commits.

