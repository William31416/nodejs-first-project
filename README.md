
# Node.js Kick-off Workshop: Proyecto Real

## Introducción
Bienvenidos al proyecto de introducción a Node.js. Este proyecto está diseñado para aplicar los conceptos básicos de Node.js y Express que has aprendido durante la primera semana del curso. Crearás una API RESTful que interactúa con el sistema de archivos en lugar de una base de datos.

## Instrucciones de Entrega
1. Crea un repositorio en GitHub llamado `nodejs-first-project`.
2. Sigue las instrucciones y completa los objetivos establecidos en este documento.
3. Sube tu proyecto a GitHub y comparte el enlace del repositorio.

## Objetivos
- Configurar un entorno de desarrollo Node.js.
- Crear un servidor web utilizando Node.js y Express.
- Implementar rutas y middlewares en Express.
- Leer y escribir datos en el sistema de archivos utilizando el módulo `fs`.
- Manejar errores y asegurar la API con middlewares.

## Descripción del Proyecto
Crearás una API RESTful para gestionar una lista de tareas (To-Do List). Las tareas se almacenarán en un archivo JSON en el sistema de archivos.

## Historias de Usuario

### 1. Como usuario, quiero poder crear una nueva tarea para agregarla a mi lista de tareas.
- **Ruta:** POST `/tasks`
- **Cuerpo de la solicitud:**
  ```json
  {
    "title": "Nombre de la tarea",
    "description": "Descripción de la tarea"
  }
  ```
- **Respuesta:**
  ```json
  {
    "message": "Tarea creada exitosamente",
    "task": {
      "id": 1,
      "title": "Nombre de la tarea",
      "description": "Descripción de la tarea"
    }
  }
  ```

### 2. Como usuario, quiero poder ver todas mis tareas para revisarlas.
- **Ruta:** GET `/tasks`
- **Respuesta:**
  ```json
  [
    {
      "id": 1,
      "title": "Nombre de la tarea",
      "description": "Descripción de la tarea"
    },
    ...
  ]
  ```

### 3. Como usuario, quiero poder ver una tarea específica por su ID para conocer sus detalles.
- **Ruta:** GET `/tasks/:id`
- **Respuesta:**
  ```json
  {
    "id": 1,
    "title": "Nombre de la tarea",
    "description": "Descripción de la tarea"
  }
  ```

### 4. Como usuario, quiero poder actualizar una tarea existente para modificar su información.
- **Ruta:** PUT `/tasks/:id`
- **Cuerpo de la solicitud:**
  ```json
  {
    "title": "Nuevo nombre de la tarea",
    "description": "Nueva descripción de la tarea"
  }
  ```
- **Respuesta:**
  ```json
  {
    "message": "Tarea actualizada exitosamente",
    "task": {
      "id": 1,
      "title": "Nuevo nombre de la tarea",
      "description": "Nueva descripción de la tarea"
    }
  }
  ```

### 5. Como usuario, quiero poder eliminar una tarea para mantener mi lista organizada.
- **Ruta:** DELETE `/tasks/:id`
- **Respuesta:**
  ```json
  {
    "message": "Tarea eliminada exitosamente"
  }
  ```

## Requisitos del Proyecto

### 1. Configuración del Entorno
- Descarga e instala Node.js desde [nodejs.org](https://nodejs.org/). Recomendamos la versión LTS.
- Verifica la instalación con los siguientes comandos:
  ```sh
  node -v
  npm -v
  ```

### 2. Inicialización del Proyecto
- Inicia un nuevo proyecto Node.js:
  ```sh
  mkdir nodejs-first-project
  cd nodejs-first-project
  npm init -y
  ```

### 3. Instalación de Dependencias
- Instala Express:
  ```sh
  npm install express
  ```

### 4. Creación de la API RESTful

#### Estructura del Proyecto
```
nodejs-first-project/
├── data/
│   └── tasks.json
├── src/
│   ├── routes/
│   │   └── tasks.js
│   ├── middlewares/
│   │   └── errorHandler.js
│   └── app.js
├── package.json
└── index.js
```

#### 1. Crear el archivo `tasks.json`
- Crea la carpeta `data` y el archivo `tasks.json`:
  ```sh
  mkdir data
  echo "[]" > data/tasks.json
  ```

#### 2. Crear el Servidor con Express
- Crea la carpeta `src` y el archivo `app.js`:
  ```js
  const express = require("express"); // Importamos Express
  const tasksRoutes = require("./routes/tasks"); // Importamos las rutas de la API
  const errorHandler = require("./middlewares/errorHandler"); // Importamos el middleware para manejo de errores

  const app = express(); // Instanciamos Express
  const PORT = 3000; // Puerto del servidor en donde se ejecutará la API

  app.use(express.json()); // Middleware para parsear el cuerpo de las solicitudes en formato JSON. Tambien conocido como middleware de aplicación.
  app.use("/tasks", tasksRoutes); // Middleware para manejar las rutas de la API. Tambien conocido como middleware de montaje o de enrutamiento.
  app.use(errorHandler); // Middleware para manejar errores.

  app.listen(PORT, () => {
    console.log(`Server running at http://localhost:${PORT}/`);
  });
  ```

#### 3. Crear las Rutas de la API
- Crea la carpeta `routes` y el archivo `tasks.js`:
  ```js
  const express = require("express");
  const fs = require("fs");
  const path = require("path");

  const router = express.Router();
  const tasksFilePath = path.join(__dirname, "../../data/tasks.json");

  // Leer tareas desde el archivo
  const readTasks = () => {
    const tasksData = fs.readFileSync(tasksFilePath); // Leer el archivo. Este poderoso metodo nos permite leer archivos de manera sincrona.
    return JSON.parse(tasksData); // Retornar los datos en formato JSON.
  };

  // Escribir tareas en el archivo
  const writeTasks = (tasks) => {
    fs.writeFileSync(tasksFilePath, JSON.stringify(tasks, null, 2)); // Escribir los datos en el archivo. Este poderoso metodo nos permite escribir archivos de manera sincrona.
  };

  // Crear una nueva tarea
  router.post("/", (req, res) => {
    const tasks = readTasks();
    const newTask = {
      id: tasks.length + 1, // simulamos un id autoincrementable
      title: req.body.title, // obtenemos el titulo de la tarea desde el cuerpo de la solicitud
      description: req.body.description, // obtenemos la descripcion de la tarea desde el cuerpo de la solicitud
    };
    tasks.push(newTask);
    writeTasks(tasks);
    res.status(201).json({ message: "Tarea creada exitosamente", task: newTask });
  });

  // Obtener todas las tareas
  router.get("/", (req, res) => {
    const tasks = readTasks();
    res.json(tasks);
  });

  // Obtener una tarea por ID
  router.get("/:id", (req, res) => {
    const tasks = readTasks();
    const task = tasks.find((t) => t.id === parseInt(req.params.id));
    if (!task) {
      return res.status(404).json({ message: "Tarea no encontrada" });
    }
    res.json(task);
  });

  // Actualizar una tarea por ID
  router.put("/:id", (req, res) => {
    const tasks = readTasks();
    const taskIndex = tasks.findIndex((t) => t.id === parseInt(req.params.id));
    if (taskIndex === -1) {
      return res.status(404).json({ message: "Tarea no encontrada" });
    }
    const updatedTask = {
      ...tasks[taskIndex],
      title: req.body.title,
      description: req.body.description,
    };
    tasks[taskIndex] = updatedTask;
    writeTasks(tasks);
    res.json({ message: "Tarea actualizada exitosamente", task: updatedTask });
  });

  // Eliminar una tarea por ID
  router.delete("/:id", (req, res) => {
    const tasks = readTasks();
    const newTasks = tasks.filter((t) => t.id !== parseInt(req.params.id));
    if (tasks.length === newTasks.length) {
      return res.status(404).json({ message: "Tarea no encontrada" });
    }
    writeTasks(newTasks);
    res.json({ message: "Tarea eliminada exitosamente" });
  });

  module.exports = router;
  ```

#### 4. Crear Middleware para Manejo de Errores
- Crea la carpeta `middlewares` y el archivo `errorHandler.js`:
  ```js
  const errorHandler = (err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({ message: "Ocurrió un error en el servidor" });
  };

  module.exports = errorHandler;
  ```

#### 5. Archivo de Entrada del Proyecto
- Crea el archivo `index.js` en la raíz del proyecto:
  ```js
  require("./src/app");
  ```

## 5. Ejecución del Proyecto
- Ejecuta el proyecto con el siguiente comando:
  ```sh
  node index.js
  ```

- Deberás ver un mensaje similar al siguiente:
  ```
  Server running at http://localhost:3000/
  ```

- Abre tu navegador y accede a [http://localhost:3000/tasks](http://localhost:3000/tasks) para probar la API.

## 6. Probando la API (Ejemplo con POST)

Para probar nuestra API, podemos utilizar herramientas como [Postman](https://www.postman.com/) o [Insomnia](https://insomnia.rest/). Descarga e instala una de estas aplicaciones para realizar las siguientes pruebas.

Para este ejemplo, consideraremos que estamos utilizando Postman. Una vez descargado e instalado, sigue los pasos a continuación:

1. Abre Postman y encontrarás una interfaz similar a la siguiente:

<img src="./assets/postman_1.png" alt="Postman" width="500">

2. Le daremos click al boton `new` para crear una colección de peticiones. 

3. Se nos abrirá una venta donde elegiremos colección. En postman, las colecciones son grupos de peticiones que se pueden ejecutar juntas. Es mas que todo para organizar las peticiones que haremos a nuestra API y poder compartirlas con otros desarrolladores.

<img src="./assets/postman_2.png" alt="Postman" width="500">

4. Le damos un nombre a nuestra colección, en este caso `Node.js Kick-off Project`. A este punto puedes indagar más sobre funcionalidades de postman como variables de entorno, autenticación, etc. Te dejaré un link a la documentación oficial de postman [aquí](https://learning.postman.com/docs/getting-started/introduction/). Y un video de youtube que explica como usar postman mas a profundidad [aquí](https://www.youtube.com/watch?v=iFDQ3NFs95M&list=PLDbrnXa6SAzUsLG1gjECgFYLSZDov09fO).

<img src="./assets/postman_3.png" alt="Postman" width="500">

<img src="./assets/postman_4.png" alt="Postman" width="500">

5. Ahora crearemos una petición para crear una tarea. Para ello, le daremos click al boton `...` que se encuentra al lado de la colección que acabamos de crear y seleccionaremos `Add request`.

<img src="./assets/postman_5.png" alt="Postman" width="500">

6. Se nos abrirá una ventana donde podremos configurar nuestra petición. Le daremos un nombre a nuestra petición, en este caso `Create Task`. En el campo `Request URL` colocaremos la dirección de nuestra API, en este caso `http://localhost:3000/tasks`. En el campo `Request type` seleccionaremos `POST`. En el campo `Body` seleccionaremos `raw` y en el tipo de dato seleccionaremos `JSON` y en el campo inferior colocaremos el siguiente JSON:

```json
{
  "title": "Comprar leche",
  "description": "Ir al supermercado y comprar leche"
}
```

<img src="./assets/postman_6.png" alt="Postman" width="500">

<img src="./assets/postman_7.png" alt="Postman" width="500">

7. Ahora le daremos click al boton `Send` y veremos la respuesta de nuestra API.

<img src="./assets/postman_8.png" alt="Postman" width="500">

A este punto, ya debes suponer que el archivo `tasks.json` creado en la carpeta `data` contiene la tarea que acabamos de crear. 

<img src="./assets/tasks.json_1.png" alt="Postman" width="500">

Si no ves una respuesta similar a la que se muestra en la imagen, es posible que haya un error en tu código. Revisa el paso a paso y asegúrate de que todo esté configurado correctamente.

## 7. Probando la API (Resto de Verbos HTTP)

Como trabajo autonomo, prueba el resto de los verbos HTTP que se mencionan en las historias de usuario los cuales son:

- GET `/tasks`
- GET `/tasks/:id`
- PUT `/tasks/:id`
- DELETE `/tasks/:id`

## 8. Preguntas de Reflexión y trabajo investigativo

1. ¿Qué es el filesystem (fs) en Node.js y para qué se utiliza?
R/: El módulo fs en Node.js proporciona una API para interactuar con el sistema de archivos. Se utiliza para leer, escribir, eliminar, renombrar y manipular archivos y directorios en el sistema.

2. ¿Qué es un middleware en Express y cuál es su propósito?
R/: Un middleware en Express es una función que procesa las solicitudes entrantes antes de que lleguen a la ruta final. Su propósito es realizar tareas como autenticación, validación de datos, y manejo de errores.

3. ¿Qué es un endpoint en una API RESTful y cuál es su función?
R/: Un endpoint en una API RESTful es una URL específica a la que se pueden enviar solicitudes HTTP. Su función es permitir la interacción con los recursos de la API, realizando operaciones como crear, leer, actualizar o eliminar datos.

4. ¿Qué son los verbos HTTP y cuáles son los más comunes?
R/: Los verbos HTTP son métodos que indican la acción que se quiere realizar en un recurso. Los más comunes son:
- GET: Es como pedir ver algo. Por ejemplo, cuando abres una página web o consultas el perfil de un usuario en redes sociales, estás usando el verbo GET para obtener esa información.
- POST: Lo usamos para enviar datos nuevos al servidor. Por ejemplo, cuando publicas una foto en redes sociales o llenas un formulario en línea para realizar una compra, estás usando POST para enviar esa información al servidor.
- PUT: Es para actualizar algo que ya existe. Por ejemplo, si editas tu perfil en una aplicación o ajustas los detalles de un producto en un carrito de compras en línea, estás usando PUT para actualizar esos datos.
- DELETE: Lo usamos para eliminar algo. Por ejemplo, si borras un mensaje de correo electrónico o eliminas una publicación en redes sociales, estás usando DELETE para quitar ese contenido del servidor.
- PATCH: Similar a PUT, pero se usa cuando quieres hacer cambios parciales en algo sin necesidad de enviar toda la información. Por ejemplo, si solo actualizas la dirección de envío en tu perfil de compras en línea, podrías usar PATCH.
- HEAD: Es como hacer una solicitud de información sin realmente obtener el contenido completo. Por ejemplo, cuando un navegador web solicita los encabezados de una página sin cargar toda la página, está usando HEAD.
- OPTIONS: Sirve para preguntar qué opciones están disponibles en un recurso. Por ejemplo, si quieres saber qué métodos y funciones puedes usar en una API, podrías usar OPTIONS para obtener esa información.

5. ¿Qué es JSON y por qué es utilizado en las API RESTful?
R/: JSON (JavaScript Object Notation) es un formato de intercambio de datos, ligero y fácil de leer y escribir para humanos, y fácil de parsear y generar para máquinas. Es utilizado en las API RESTful por su simplicidad y compatibilidad con la mayoría de los lenguajes de programación.

6. En lo que respecta al envio de datos a lo largo de los verbos http responde:
    - ¿Qué es el body de una petición?
      R/: El body de una petición es la parte de la solicitud HTTP que contiene los datos enviados al servidor, típicamente en verbos como POST y PUT.
    
    - ¿Qué es el body de una respuesta?
      R/: El body de una respuesta es la parte de la respuesta HTTP que contiene los datos enviados desde el servidor al cliente, como el contenido de una página web o los datos solicitados por una API.
    
    - ¿Qué es el query de una petición?
      R/: El query de una petición es una cadena de texto en la URL que contiene pares de clave-valor, utilizados para enviar información adicional al servidor en una solicitud GET.
    
    - ¿Qué es el params de una petición?
      R/: El params de una petición se refiere a los parámetros que forman parte de la ruta de la URL, utilizados para identificar recursos específicos, por ejemplo, /usuarios/:id.

7. En lo que respecta al verbo POST responde:
    - ¿Qué es un verbo POST y cuál es su propósito?
      R/: Un verbo POST se utiliza para enviar datos al servidor para crear un nuevo recurso. Su propósito es agregar datos a una base de datos o sistema.
    
    - ¿Cuándo se utiliza un verbo POST?
      R/: Se utiliza un verbo POST cuando se necesita enviar datos al servidor para crear un nuevo recurso, como un nuevo usuario en un sistema.
    
    - ¿En qué se diferencia un verbo POST de los otros verbos HTTP como GET, PUT y DELETE?
      R/: - POST: Crea nuevos recursos.
          - GET: Obtiene recursos existentes.
          - PUT: Actualiza recursos existentes.
          - DELETE: Elimina recursos existentes.
    
    - ¿Como se envian datos en un verbo POST?
      R/: Los datos en un verbo POST se envían en el body de la solicitud HTTP, generalmente en formato JSON.

8. En lo que respecta al verbo GET responde:
    - ¿Qué es un verbo GET y cuál es su propósito?
      R/: Un verbo GET se utiliza para solicitar datos de un servidor. Su propósito es obtener información sin modificarla.
    
    - ¿Cuándo se utiliza un verbo GET?
      R/: Se utiliza un verbo GET cuando se necesita obtener datos del servidor, como recuperar información de un usuario.
    
    - ¿En qué se diferencia un verbo GET de los otros verbos HTTP como POST, PUT y DELETE?
      R/: - GET: Solo obtiene datos.
          - POST: Envía y crea nuevos datos.
          - PUT: Envía y actualiza datos.
          - DELETE: Elimina datos.

9. En lo que respecta al verbo PUT responde:
    - ¿Qué es un verbo PUT y cuál es su propósito?
      R/: Un verbo PUT se utiliza para actualizar un recurso existente en el servidor con nuevos datos.
    
    - ¿Cuándo se utiliza un verbo PUT?
      R/: Se utiliza un verbo PUT cuando se necesita actualizar completamente un recurso existente, como modificar la información de un usuario.
    
    - ¿En qué se diferencia un verbo PUT de los otros verbos HTTP como POST, GET y DELETE?
      R/: - PUT: Actualiza recursos existentes.
          - POST: Crea nuevos recursos.
          - GET: Obtiene recursos.
          - DELETE: Elimina recursos.

10. En lo que respecta al verbo DELETE responde:
    - ¿Qué es un verbo DELETE y cuál es su propósito?
      R/: Un verbo DELETE se utiliza para eliminar un recurso existente en el servidor.

    - ¿Cuándo se utiliza un verbo DELETE?
      R/: Se utiliza un verbo DELETE cuando se necesita eliminar un recurso existente, como borrar un usuario de una base de datos.

    - ¿En qué se diferencia un verbo DELETE de los otros verbos HTTP como POST, GET y PUT?
      R/: - DELETE: Elimina recursos.
          - POST: Crea recursos.
          - GET: Obtiene recursos.
          - PUT: Actualiza recursos.

12. ¿Qué es un status code y cuáles son los más comunes?
  R/: Un status code es un código numérico en la respuesta HTTP que indica el resultado de la solicitud. Los más comunes son:

  - 200 OK: Solicitud exitosa.
  - 201 Created: Recurso creado exitosamente.
  - 400 Bad Request: Solicitud incorrecta.
  - 401 Unauthorized: No autorizado.
  - 404 Not Found: Recurso no encontrado.
  - 500 Internal Server Error: Error interno del servidor.

13. ¿Cuales son los status code mas comunes para el verbo POST?
  R/: - 201 Created: Recurso creado exitosamente.
      - 400 Bad Request: Solicitud incorrecta.
      - 401 Unauthorized: No autorizado.
      - 500 Internal Server Error: Error interno del servidor.

14. ¿Cuales son los status code mas comunes para el verbo GET?
  R/: - 200 OK: Solicitud exitosa.
      - 400 Bad Request: Solicitud incorrecta.
      - 401 Unauthorized: No autorizado.
      - 404 Not Found: Recurso no encontrado.
      - 500 Internal Server Error: Error interno del servidor.

15. ¿Cuales son los status code mas comunes para el verbo PUT?
  R/: - 200 OK: Actualización exitosa.
      - 204 No Content: Actualización exitosa sin contenido de respuesta.
      - 400 Bad Request: Solicitud incorrecta.
      - 401 Unauthorized: No autorizado.
      - 404 Not Found: Recurso no encontrado.
      - 500 Internal Server Error: Error interno del servidor.

16. ¿Cuales son los status code mas comunes para el verbo DELETE?
    R/: - 200 OK: Eliminación exitosa.
        - 204 No Content: Eliminación exitosa sin contenido de respuesta.
        - 400 Bad Request: Solicitud incorrecta.
        - 401 Unauthorized: No autorizado.
        - 404 Not Found: Recurso no encontrado.
        - 500 Internal Server Error: Error interno del servidor.
