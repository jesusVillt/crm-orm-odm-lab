# CRM ORM/ODM Lab

API REST de un CRM básico que combina un ORM (Sequelize + PostgreSQL) y un ODM (Mongoose + MongoDB).

## Stack

- Node.js 22, Express 5, CommonJS
- Sequelize + PostgreSQL 16 (`User`, `Company`, `Contact`)
- Mongoose + MongoDB 7 (`Activity`)
- Jest + Supertest
- GitHub Codespaces, Dev Containers, Docker Compose
- Supervisor (`npm run dev`)

## Arquitectura

```text
GitHub Codespace
│
├── app       Node.js 22  ──┬── Sequelize ──> postgres (PostgreSQL)
│                           └── Mongoose  ──> mongo    (MongoDB)
├── postgres
└── mongo
```

La aplicación se conecta por nombre de servicio (`postgres`, `mongo`). Las credenciales de desarrollo llegan como variables de entorno definidas en `.devcontainer/docker-compose.yml` (ver `.env.example`).

## Iniciar el Codespace

1. En GitHub: **Code → Codespaces → Create codespace on main**.
2. Espera a que se levanten los tres servicios (`app`, `postgres`, `mongo`). `postCreateCommand` ejecuta `npm install`.

## Instalar dependencias

```bash
npm install
```

## Seed y reset

```bash
npm run seed    # inserta datos deterministas (3 users, 4 companies, 8 contacts, 10 activities)
npm run reset   # elimina y recrea tablas/base de datos y vuelve a sembrar
```

## Iniciar la API

```bash
npm start       # node ./bin/www
npm run dev     # supervisor ./bin/www
```

Servidor en el puerto `3000` (variable `PORT`).

## Pruebas

```bash
npm test
```

Cada suite restablece PostgreSQL y MongoDB antes de ejecutarse y cierra las conexiones al terminar.

## Endpoints

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/health` | Health check |
| GET | `/users` | Listar usuarios |
| GET | `/users/:id` | Obtener usuario |
| POST | `/users` | Crear usuario |
| PUT | `/users/:id` | Actualizar usuario |
| DELETE | `/users/:id` | Eliminar usuario |
| GET | `/companies` | Listar compañías (`?industry=`) |
| GET | `/companies/:id` | Obtener compañía |
| POST | `/companies` | Crear compañía |
| PUT | `/companies/:id` | Actualizar compañía |
| DELETE | `/companies/:id` | Eliminar compañía |
| GET | `/contacts` | Listar contactos |
| GET | `/contacts/:id` | Obtener contacto |
| POST | `/contacts` | Crear contacto |
| PUT | `/contacts/:id` | Actualizar contacto |
| DELETE | `/contacts/:id` | Eliminar contacto |
| GET | `/activities` | Listar actividades (`?type=`) |
| GET | `/activities/:id` | Obtener actividad |
| POST | `/activities` | Crear actividad |
| PUT | `/activities/:id` | Actualizar actividad |
| DELETE | `/activities/:id` | Eliminar actividad |

Los errores se devuelven como JSON: `{ "error": "Contact not found" }`.


## Respuestas

**1. Dos motores.**
Activity es buen candidato para un base documental ya que, como se puede ver en el codigo, es un esquema flexible,
si nos vamos a seed.js y nos fijamos en activities vemos que existe el campo metadata el cual tiene una estructura diferente
para cada documento. Por otro lado user, company y contact tiene una estructura más clara, también company y contact establecen
una relación entre si.

**2. ORM vs ODM.**
Me gusta entender al ORM y al ODM como herramientas que nos permiten llevar nuestra base de datos hacia adentro de 
nuestro código a través de objetos. En este trabajo se utilizó Mongoose y Sequelize, su principal diferencia esta
en que Mongoose es un ODM, o sea es orientado a documentos, mientras que Sequelize es para el modelo relacional.



**3. Configuración por variables de entorno.**
Las credenciales están definidas desde el punto .env.example de la aplicación,no es buena práctica ponerlas
en cualquier otro lado, pues al subirlas al repositorio las estaríamos exponiendo, lo correcto sería
que el .env quedará oculto antes de subirlo al repositorio. En cuanto los nombres
de los host utilizados para conectarnos con PostgreSQL y MongoDB estos son: postgres y mongo.Gracias a que con Docker podemos
usar los nombres de los servicios como si fueran los de un host, por eso no ponemos tal cual "localhost".


**4. Asociacones.**
Una compañía tiene muchos contactos, pero un contacto está asociado únicamente a una compañía. La
llave foránea es companyId que vive en la tablita de contactos, pues de esa forma se puede cumplir que
una misma compañía tenga asociados muchos contactos. El alias contacts nos sirve para nombrar la
relación y luego la podemos usar, de hecho en uno de los retos teníamos que traer los contactos
asociados a cada compañía, para eso es este alias, pues en este caso es el que nos permite
decir que una compañía tiene varios contactos.



**5. Eager loading.**
Eager loading. En el Reto 05, ¿qu ́e diferencia habr ́ıa entre traer la compa ̃n ́ıa
y luego hacer una segunda consulta para sus contactos, y traerlos en la misma
consulta con include? ¿Cu ́al es preferible y por qu ́e?
La diferencia estaría pues en la cantidad de consultas que terminaríamos haciendo. 
Si ya sabemos que siempre vamos a necesitar estos datos juntos, entonces es más optimo de esta 
forma, en vez de tener que hacer consultas separadas.

**6. Instancia vs consulta.**
La ventaja principal está en que, en el update que usamos en contactos, tenemos
acceso a la instancia, desde antes, lo que significa que podemos acceder a sus atributos. Mientras
que en el segundo enfoque simplemente se modifica, ya no podemos acceder a él.

**7. Esquema flexible.**
El tipo de dato es mongoose.Schema.Types.Mixed, este es el que nos hace la magia
de poder guardar diferentes tipos de estructuras sin problemas, la desventaja 
es precisamente su ventaja: podemos guardar cualquier tipo de cosa, entonces las 
validaciones se nos pueden complicar.

**8. Sin ref.**
No podemos hacer uso de populate en este caso, porque para que esto funcionara
implicaría que tanto users como contacts fueran también documentos de Mongodb, pues populate
solo funciona para estos. En cuanto a las consecuencias de integridad, el problema que tenemos es que si
por ejemplo borramos un usuario, entonces este todavía aparecerá ahí referenciado en Activity.

**9. Documento actualizado.**
Devolvía el documento antes de ser actualizado, pues es el comportamiento por defecto
que tiene findByIdAndUpdate. Para solucionar esto se le paso como parámetro el siguiente 
objeto: { new: true }. De esta forma ya regresaba el objeto actualizado. 

**10. Pruebas de comportamiento.**
Probar el comportamiento tiene la principal ventaja de que queda más abierta la 
solución. Cada quien tiene un razonamiento diferente y pudo haber encontrado 
una solución más simple, más compleja, pero en este caso basta con que 
haga lo que deba de hacer.


**11. Repetibilidad.**
Este código nos cierra las conexiones con las bases de datos y también las regresas
a su estado inicial, como podemos intuir del reset(). Esto permite que las bases de datos
tengan el estado que se les dio en el archivo seed.js y así unas pruebas no se terminan
afectando con otras, pues todas terminan partiendo de la misma base de datos.

beforeAll(async () => {
  await connectSequelize();
  await connectMongoose();
  await reset();
});

afterAll(async () => {
  await closeSequelize();
  await closeMongoose();
});


**12. Tu experiencia.**
En lo personal para mí el reto más difícil fue el número 7, pues fue el que más tiempo
me tomó resolver debido a una serie de distracciones. Para empezar estaba tratando de modificar, sin darme
cuenta, el objeto a través del modelo, o sea tenía:
await Contact.update(req.body, { fields: ['firstName', 'lastName', 'email', 'phone', 'companyId'] });
en lugar de:
await contact.update(req.body, { fields: ['firstName', 'lastName', 'email', 'phone', 'companyId'] });
Ese pequeño error pasó desapercibido, por mucho tiempo y estaba convencido de que algo estaba mal
con la lógica, pero solo fue un error de escritura.


## Evidencia
<img width="679" height="319" alt="imagen" src="https://github.com/user-attachments/assets/332ffc46-4cb9-49bb-a41a-f7360763fc82" />





