## Planificación del Proyecto

### **Objetivo:**
Crear una aplicación en Java que gestione y muestre frases icónicas de películas famosas, junto con la información de la película (título, año de lanzamiento, director) y el personaje que dice la frase.

### **Funcionalidades:**
1. **Gestión de Frases:**
   - Guardar frases con sus datos relacionados: película y personaje.
   - Mostrar todas las frases almacenadas.

2. **Búsqueda Avanzada:**
   - Filtrar frases por título de la película.
   - Buscar por el personaje que dice la frase.

3. **Frases Aleatorias:**
   - Mostrar una frase aleatoria cuando el usuario lo solicite.

4. **Persistencia de Datos:**
   - Almacenar frases en una base de datos para que sean persistentes.

5. **Conexión con una API Pública (Opcional):**
   - Usar una API de películas (como OMDb API) para enriquecer los datos de las películas relacionadas con las frases.

---

## Arquitectura del Proyecto

- **Modelo (Model):**
  - Clases que representan las entidades: *Frase*, *Película*, *Personaje*.

- **Repositorio (Repository):**
  - Interacción con la base de datos utilizando **Spring Data JPA**.

- **Servicio (Service):**
  - Lógica de negocio para gestionar frases, películas y personajes.

- **Controlador (Controller):**
  - Endpoints REST para acceder a las frases y datos relacionados.

---

## Desarrollo Paso a Paso

### 1. **Configuración Inicial**

1. Crea un nuevo proyecto Spring Boot en **IntelliJ** o **Eclipse**.
2. Añade las dependencias necesarias en el archivo `pom.xml` para Spring Boot, Spring Data JPA y la base de datos (e.g., H2 o MySQL):
   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-jpa</artifactId>
   </dependency>
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   <dependency>
       <groupId>com.h2database</groupId>
       <artifactId>h2</artifactId>
       <scope>runtime</scope>
   </dependency>
   ```

3. Configura el archivo `application.properties`:
   ```properties
   spring.datasource.url=jdbc:h2:mem:screenmatch
   spring.datasource.driverClassName=org.h2.Driver
   spring.datasource.username=sa
   spring.datasource.password=password
   spring.jpa.hibernate.ddl-auto=update
   ```

---

### 2. **Modelado de Datos**

Crea las entidades principales.

#### Clase `Frase`:
```java
@Entity
public class Frase {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String texto;

    @ManyToOne
    private Pelicula pelicula;

    @ManyToOne
    private Personaje personaje;

    // Getters, Setters, Constructores
}
```

#### Clase `Pelicula`:
```java
@Entity
public class Pelicula {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String titulo;
    private int anio;
    private String director;

    @OneToMany(mappedBy = "pelicula")
    private List<Frase> frases;

    // Getters, Setters, Constructores
}
```

#### Clase `Personaje`:
```java
@Entity
public class Personaje {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nombre;

    @OneToMany(mappedBy = "personaje")
    private List<Frase> frases;

    // Getters, Setters, Constructores
}
```

---

### 3. **Repositorio y Servicio**

#### Repositorios:
Crea repositorios para cada entidad.
```java
public interface FraseRepository extends JpaRepository<Frase, Long> {
    List<Frase> findByPeliculaTitulo(String titulo);
    List<Frase> findByPersonajeNombre(String nombre);
}
```

#### Servicio:
Implementa la lógica de negocio.
```java
@Service
public class FraseService {
    @Autowired
    private FraseRepository fraseRepository;

    public List<Frase> obtenerTodasLasFrases() {
        return fraseRepository.findAll();
    }

    public List<Frase> buscarPorPelicula(String titulo) {
        return fraseRepository.findByPeliculaTitulo(titulo);
    }

    public List<Frase> buscarPorPersonaje(String nombre) {
        return fraseRepository.findByPersonajeNombre(nombre);
    }

    public Frase obtenerFraseAleatoria() {
        List<Frase> frases = fraseRepository.findAll();
        return frases.get(new Random().nextInt(frases.size()));
    }
}
```

---

### 4. **Controlador REST**

Crea un controlador para manejar las solicitudes HTTP.
```java
@RestController
@RequestMapping("/frases")
public class FraseController {
    @Autowired
    private FraseService fraseService;

    @GetMapping
    public List<Frase> obtenerTodasLasFrases() {
        return fraseService.obtenerTodasLasFrases();
    }

    @GetMapping("/pelicula/{titulo}")
    public List<Frase> buscarPorPelicula(@PathVariable String titulo) {
        return fraseService.buscarPorPelicula(titulo);
    }

    @GetMapping("/personaje/{nombre}")
    public List<Frase> buscarPorPersonaje(@PathVariable String nombre) {
        return fraseService.buscarPorPersonaje(nombre);
    }

    @GetMapping("/aleatoria")
    public Frase obtenerFraseAleatoria() {
        return fraseService.obtenerFraseAleatoria();
    }
}
```

---

### 5. **Carga de Datos Inicial**

Crea un archivo `data.sql` para precargar frases, películas y personajes.
```sql
INSERT INTO pelicula (id, titulo, anio, director) VALUES (1, 'El Padrino', 1972, 'Francis Ford Coppola');
INSERT INTO personaje (id, nombre) VALUES (1, 'Vito Corleone');
INSERT INTO frase (id, texto, pelicula_id, personaje_id) VALUES (1, 'Le haré una oferta que no podrá rechazar.', 1, 1);
```

---

### 6. **Ejemplo de Uso**

1. **Obtener todas las frases**:  
   `GET http://localhost:8080/frases`

2. **Buscar frases por película**:  
   `GET http://localhost:8080/frases/pelicula/El%20Padrino`

3. **Buscar frases por personaje**:  
   `GET http://localhost:8080/frases/personaje/Vito%20Corleone`

4. **Obtener una frase aleatoria**:  
   `GET http://localhost:8080/frases/aleatoria`

---

### 7. **Mejoras Futuras**

- **Autenticación**: Implementar autenticación para proteger la API.  
- **Front-End**: Desarrollar una interfaz para mostrar las frases de manera visual.  
- **Integración con APIs**: Usar la API de OMDb para enriquecer la información de las películas.  
