# Listado de comandos de Git
## git init

 Sirve para inicializar un repositorio desde 0 en mi entorno local
 
## git clone

 Sirve para **descargar** un repositorio remoto dentro de la carpeta en que estoy del entorno local (usar url).
 
```bash
git clone https://github.com/usuario/repo.git
```
 
## git config

 Sirve para poder obtener informacion o actualizar informacion de configuracion en git, por ejemplo nombre y correo 

- '--global': aplica para todos los repositorios del usuario especifico.
- '--system': aplica para todos los repositorios de todos los usuarios.
- '--local': aplica para el repositorio actual (tambien puede no llevar **FLAG**).

Prioridad: local > global > system.

## git status

Para validar los posibles cambios en el commit.

## git add

Sirve para agregar un nuevo archivo o directorio al cual se le va a realizar el commit.

## git commit

Sirve para guardar los datos en repositorio local.

## git push

Implementa las modificacion del ultimo commit en el repositorio remoto.

## git log

Permite visualizar los commits realizados en el repositorio desde el mas recienta al mas antiguo.
			