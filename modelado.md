# Modelado de Datos

## Modelado de Datos en MySQL

MySQL utiliza un modelo de datos relacional, lo que significa que la información se almacena en tablas que pueden estar relacionadas entre sí. Estas relaciones se definen a través de claves primarias y foráneas.

- **Estructura Tabular:** Los datos se organizan en filas y columnas, con cada tabla representando un tipo de entidad.
- **Integridad Referencial:** Se mantiene mediante el uso de claves foráneas que enlazan registros de diferentes tablas.
- **Normalización:** El proceso de estructurar una base de datos relacional para reducir la redundancia y mejorar la integridad de los datos. Esto incluye técnicas como dividir grandes tablas en otras más pequeñas y relacionadas, y establecer relaciones entre ellas.

El diseño de la base de datos sigue las formas normales para asegurar la eficiencia y la coherencia de los datos.

## Modelado de Datos en MongoDB

MongoDB es un ejemplo de una base de datos NoSQL orientada a documentos. A diferencia de MySQL, no requiere una estructura de tabla fija y permite almacenar documentos JSON con esquemas dinámicos.

- **Esquemas Flexibles:** Los documentos en una colección pueden tener diferentes campos. Esto permite la evolución del esquema de datos sin necesidad de modificar toda la base de datos.
- **Documentos Anidados:** MongoDB puede almacenar arrays y subdocumentos, lo que facilita la representación de datos jerárquicos y reduce la necesidad de joins.
- **ID de Objeto:** Cada documento tiene un campo `_id` que actúa como clave primaria.

MongoDB es particularmente útil para casos de uso que requieren una gran flexibilidad y velocidad con esquemas que pueden cambiar con el tiempo.

---

# Modelado de Datos

La elección entre sistemas de bases de datos como MySQL y MongoDB a menudo depende de las necesidades específicas del proyecto, el tipo de datos y el patrón de acceso a los mismos. En este documento, contrastamos el modelado de datos para una aplicación llamada Chefdu, una red social para chefs.

## Ejemplo de Modelado de Datos para MySQL

Las claves primarias son identificadores únicos para cada fila en una tabla de una base de datos. Son como el número de seguro social para las personas; cada uno debe ser único para que la información no se confunda o mezcle. En nuestras tablas, usamos `id` como clave primaria para identificar de forma única a cada persona, artículo, comentario, etiqueta o categoría.

### Tablas y sus relaciones

```sql
CREATE TABLE IF NOT EXISTS person (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    home_phone VARCHAR(20),
    work_phone VARCHAR(20)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS article (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    date DATETIME NOT NULL,
    text TEXT NOT NULL,
    author_id INT,
    FOREIGN KEY (author_id) REFERENCES person(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS comment (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    URL VARCHAR(255),
    text TEXT NOT NULL,
    article_id INT,
    FOREIGN KEY (article_id) REFERENCES article(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS tag (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    url VARCHAR(255) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS category (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    url VARCHAR(255) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS article_tag (
    article_id INT,
    tag_id INT,
    PRIMARY KEY (article_id, tag_id),
    FOREIGN KEY (article_id) REFERENCES article(id),
    FOREIGN KEY (tag_id) REFERENCES tag(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS article_category (
    article_id INT,
    category_id INT,
    PRIMARY KEY (article_id, category_id),
    FOREIGN KEY (article_id) REFERENCES article(id),
    FOREIGN KEY (category_id) REFERENCES category(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

![sql](/assets/mysql.png)

Este esquema refleja un modelo normalizado que cumple con la tercera forma normal, ya que cada tabla tiene una clave primaria única, todos los atributos dependen de la clave primaria, y no hay dependencias transitivas entre atributos no clave.

## Modelado de Datos en MongoDB

En MongoDB, el esquema es más flexible. No tenemos que definir todas las columnas por adelantado y cada "documento" puede tener una estructura diferente. Sin embargo, podemos usar validaciones de esquema para asegurarnos de que los datos tengan el formato correcto.

Esquema para la colección `articles`

Esta colección almacenará artículos junto con sus etiquetas, categorías y comentarios. Cada artículo tiene un título, una fecha y un texto. Las etiquetas y categorías se almacenan en listas, lo que permite que cada artículo tenga varias de ellas.

![mongo](/assets/mongodb.png)

```javascript
db.createCollection("articles", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["title", "text", "date"],
      properties: {
        title: {
          bsonType: "string",
          description: "must be a string and is required",
        },
        date: {
          bsonType: "date",
          description: "must be a date and is required",
        },
        text: {
          bsonType: "string",
          description: "must be a string and is required",
        },
        tags: {
          bsonType: "array",
          description: "must be an array and is optional",
          items: {
            bsonType: "object",
            required: ["name", "url"],
            properties: {
              name: {
                bsonType: "string",
                description: "must be a string and is required",
              },
              url: {
                bsonType: "string",
                description: "must be a string and is required",
              },
            },
          },
        },
        // Omitiendo la definición detallada de 'categories' y 'comments' por brevedad.
      },
    },
  },
});
```
