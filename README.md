# Spring Boot

## Tabla de Contenido

- [¿Qué es Spring Boot?](#que-es-spring-boot)
  - [¿Qué es la Inversión de Control (IoC)?](#que-es-la-inversion-de-control)
    - [Inyección de dependecias](#inyeccion-de-dependecias)
  - [ApplicationContext](#applicacion-context)
    - [@Bean vs @Component](#bean-vs-component)
    - [Scope (Ámbito)](#scope-de-un-bean)
    - [Las anotaciones @RestController, @Service y @Repository](#controller-service-repository)
    - [Ejemplo del funcionamiento de ApplicacionContext](#ejemplo-application-context)



<a id="que-es-spring-boot"></a>
## ¿Qué es Spring Boot?


<a id="que-es-la-inversion-de-control"></a>
### ¿Qué es la Inversión de Control (IoC)?

Es un principio de diseño de software en el que el flujo de control de un programa se invierte en comparación con la programación tradicional. En la programación tradicional, el código de la aplicación es el que controla el flujo y decide cuándo llamar a bibliotecas o frameworks. Con IoC, **este control se delega a un contenedor externo.**

En esencia, en lugar de que tus objetos controlen la creación y gestión de sus dependencias (otros objetos con los que necesitan interactuar), un contenedor IoC **se encarga de instanciar, configurar y ensamblar estos objetos.**

>[!NOTE]
> El problema que resuelve IoC:
>
> - Si una clase A crea directamente una instancia de una clase B, A depende concretamente de la implementación de B. Cualquier cambio en B puede requerir modificaciones en A. Esto dificulta la mantenibilidad, la prueba unitaria y la extensibilidad del código.

<a id="inyeccion-de-dependecias"></a>
#### Inyección de dependecias

Esté es un patrón de diseño, en específico es una **implementación** del principio de IoC, donde las dependencias de un objeto son "inyectadas" por un agente externo (el contenedor IoC), en lugar de que el objeto mismo las cree o las busque activamente.

Esto se puede hacer a través de:
1. Inyección por constructor: Las dependencias se pasan como argumentos al constructor de la clase.

```
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.repository = userRepository;
    }

    /*
      ...
    */
}
```

2. Inyección por interfaz: La dependencia se define a través de una interfaz que el contenedor implementa.

<a id="applicacion-context"></a>
### ApplicationContext

Dentro de Spring Boot existe una **implementación** del principio IoC a través de su **conetdor IoC**, conocido cómo el ApplicationContext. 

El **ApplicationContext** es una interfaz dentro de Spring y es encargada de instanciar, configurar y ensamblar los **beans de Spring** , así como de gestionar sus ciclos de vida.

> [!NOTE]
> Para ensamblar beans, el contenedor utiliza metadatos de configuración. Estos metadatos se representan en XML, anotaciones Java o código Java.

<a id="bean-vs-component"></a>
#### @Bean vs @Component

Antes de ver las diferencias respecto a estas anotaciones, es importante responder la pregunta: **¿Qués un Bean en Spring?**

> Respuesta: Un bean dentro de Spring es un objeto que es instanciado, ensamblado y gestionado por el ApplicationContext de Spring.

Ahora bien, ya sabemos qué es un Bean en Spring, pero, ¿Cómo podemos crearlos? Una de las formas para crear estos Beans es a tráves de **anotaciones Java** (@Bean y @Component)

> @Component
> Es una anotación a nivel de clase que indica que la clase Java es un "componente" de Spring.
> Cuando Spring escanea tu aplicación, **registra automáticamente** estas clases como beans en el contenedor de Spring.

```
@Component
public class CredentialEntityMapper {
    private final PasswordEncoder passwordEncoder;

    public CredentialEntityMapper(PasswordEncoder passwordEncoder) {
        this.passwordEncoder = passwordEncoder;
    }

    public Credential toCredentialEntity(SignUpRequest signUpRequest){
        /* ... */
    }

    public String encodePassword(String password){
        return passwordEncoder.encode(password);
    }
}
```

> @Bean
> Es una anotación a nivel de método que se utiliza dentro de clases de configuración (generalmente anotadas con @Configuration). Este método retorna un **objeto que debe ser registrado** como un bean en el contenedor de Spring.

```
@Configuration
public class AuthMvcConfig {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

<a id="scope-de-un-bean"></a>
#### Scope (Ámbito)
Cómo se ha mencionado anteriormente, el **ApplicationContext** de Spring es el encargado de la gestión del ciclo de vida y la configuración de los beans. El concepto de "scope" define cómo se crean y comparten las instancias de estos beans dentro del contenedor.

> 1. Scope Singleton.

Cuando defines un bean con scope singleton, el ApplicationContext crea una única instancia de ese bean. Es decir, todas las dependencias y componentes que necesiten este bean compartirán la misma instancia.

> [!NOTE]
> Spring gestiona completamente el ciclo de vida del bean singleton, desde su creación hasta su destrucción cuando el ApplicationContext se cierra.

> Ejemplo de su definición:

```
@Component
@Scope("singleton") // O simplemente @Component, ya que singleton es el valor por defecto
public class MiObjetoCompartido {
    // ...
}
```

> 2. Scope Prototype.

En contraste con el scope singleton, el scope prototype indica que el ApplicationContext **creará una nueva instancia del bean cada vez que se solicite.** Esto significa que cada vez que una dependencia necesite un bean con scope prototype, entonces, se generará un objeto fresco.

> [!NOTE]
> - Spring crea la instancia del bean prototype y la proporciona al solicitante. Sin embargo, la responsabilidad de gestionar el ciclo de vida de esa instancia (especialmente su destrucción) recae en el cliente.
> - Spring no invoca los métodos de destrucción (como @PreDestroy o DisposableBean.destroy()) para los beans prototype.

> Ejemplo de su definición

```
@Component
@Scope("prototype")
public class MiObjetoTemporal {
    // ... estado y métodos específicos de la petición ...
}
```

<a id="controller-service-repository"></a>
#### Las anotaciones @RestController, @Service y @Repository

Dentro de Spring se tiene existe el concepto de **Estereótipo**. Bajo este concepto se busca clasificar una clase según su **función o responsabilidad** en la aplicación.

> [!IMPORTANT]
> Todos los estereotipos son tipos especializados de @Component.  lo que significa que:
> 1. Permiten que las clases sean detectadas durante el escaneo de componentes
> 2. Habilitan la autoconfiguración (instanciación y ensamblaje) de beans en el contexto de Spring
> 3. Mantienen todas las funcionalidades de @Component (Scope y  gestión de su ciclos de vida por parte del ApplicationContext)

Los tres principales esterótipos dentro de Spring son:

> @Service
> 
> - Identifica clases en la capa de servicio, que **contiene la lógica de negocio** principal
> - Semánticamente indica que la clase provee operaciones de negocio

```
@Service
public class UserService {
    // Lógica de negocio
}
```

>  @Repository
>
> - Marca clases en la capa de persistencia, que manejan el acceso y manipulación de datos
> - Actúa como un marcador para DAOs (Data Access Objects)
> - Habilita la traducción automática de excepciones específicas de la persistencia a DataAccessException de Spring

```
@Repository
public class UserRepository {
    // Operaciones de acceso a datos
}
```

> @RestController
>
> - Combina @Controller con @ResponseBody, simplificando la creación de APIs RESTful
> - Se integra con el sistema de mapeo de solicitudes de Spring MVC

```
@RestController  
public class UserApiController {
    // RESTful API endpoints
}
```

Así mismo, están otros estereotipos adicionales, uno de ellos es el **@RestControllerAdvice** el cuál permite la gestión de manera centralizada de las excepciones lanzadas por los controladores registrados dentro de la aplicación.


<a id="ejemplo-application-context"></a>
#### Ejemplo del funcionamiento de ApplicacionContext

Supongamos que tenemos las siguientes clases definidas dentro de nuestra aplicación:

```
package com.example.demo.repository;

public interface RepositorioB {
    void guardar(String dato);
}
```
```
package com.example.demo.repository;

import org.springframework.stereotype.Repository;

@Repository
public class RepositorioBImpl implements RepositorioB {
    @Override
    public void guardar(String dato) {
        /* Proceso de acceso a datos... (DAO)*/
    }
}
```
```
package com.example.demo.service;

public interface ServicioA {
    void realizarAccion(String valor);
}
```
```
package com.example.demo.service;

import com.example.demo.repository.RepositorioB;
import org.springframework.stereotype.Service;

@Service
public class ServicioAImpl implements ServicioA {

    private final RepositorioB repositorioB;

    // Inyección de dependencia por constructor
    public ServicioAImpl(RepositorioB repositorioB) {
        this.repositorioB = repositorioB;
    }

    @Override
    public void realizarAccion(String valor) {
        /* Lógica de negocio...*/
        repositorioB.guardar(valor);
    }
}
```
```
package com.example.demo.controller;

import com.example.demo.service.ServicioA;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MiRestController {

    private final ServicioA servicioA;

    // Inyección de dependencia por constructor
    public MiRestController(ServicioA servicioA) {
        this.servicioA = servicioA;
    }

    @GetMapping("/procesar")
    public String procesarDato(@RequestParam String dato) {
        servicioA.realizarAccion(dato);
        return "Dato procesado!";
    }
}
```

¿Cómo funciona el ApplicationContext con estas tres capas?

> 1. Detección de Beans: Cuando la aplicación Spring Boot se inicia, el ApplicationContext (generalmente un AnnotationConfigApplicationContext) realiza un escaneo de componentes.
> Por defecto, Spring busca clases anotadas con estereotipos como @Service, @Repository, y @RestController dentro del paquete de la clase principal de la aplicación y sus subpaquetes.

> 2. Registro de Beans: El ApplicationContext identifica las clases anotadas y las registra como beans dentro de su contenedor. Esto significa que el ApplicationContext ahora sabe de la existencia de RepositorioBImpl, ServicioAImpl, y MiRestController, y gestionará sus instancias.

> 3. Resolución e Inyección de Dependencias: Aquí es donde el principio de "no depender de implementaciones sino de abstracciones" entra en juego:
> - ServicioAImpl depende de RepositorioB: El constructor de ServicioAImpl declara una dependencia de tipo RepositorioB. El ApplicationContext busca en sus beans registrados alguna clase que implemente la interfaz RepositorioB. Encuentra RepositorioBImpl y automáticamente crea una instancia de RepositorioBImpl y la inyecta en el constructor de ServicioAImpl. ServicioAImpl no necesita saber la implementación concreta, solo que necesita algo que cumpla con el contrato de RepositorioB.
> 
> - MiRestController depende de ServicioA: De manera similar, el constructor de MiRestController declara una dependencia de tipo ServicioA. El ApplicationContext busca una implementación de ServicioA y encuentra ServicioAImpl. Crea una instancia de ServicioAImpl y la inyecta en el constructor de MiRestController. MiRestController interactúa con la lógica de negocio a través de la abstracción ServicioA, sin conocer los detalles de ServicioAImpl.



