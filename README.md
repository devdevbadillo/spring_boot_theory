# Spring Boot

## Tabla de Contenido

- [¿Qué es Spring Boot?](#que-es-spring-boot)
  - [¿Qué es la Inversión de Control (IoC)?](#que-es-la-inversion-de-control)
    - [Inyección de dependecias](#inyeccion-de-dependecias)
    - [Localizador de Servicios (Service Locator)](#service-locator)
  - [ApplicationContext](#applicacion-context)
    - [@Bean vs @Component](#bean-vs-component)
    - [Las anotaciones @RestController, @Service y @Repository](#controller-service-repository)
    - [La anotación @Configuration](#anotacion-configuration)
    - [Ejemplo del funcionamiento de ApplicacionContext](#ejemplo-application-context)



<a id="que-es-spring-boot"></a>
## ¿Qué es Spring Boot?


<a id="que-es-la-inversion-de-control"></a>
### ¿Qué es la Inversión de Control (IoC)?

Es un principio de diseño de software en el que el flujo de control de un programa se invierte en comparación con la programación tradicional. En la programación tradicional, el código de la aplicación es el que controla el flujo y decide cuándo llamar a bibliotecas o frameworks. Con IoC, **este control se delega a un contenedor externo.**

En esencia, en lugar de que tus objetos controlen la creación y gestión de sus dependencias (otros objetos con los que necesitan interactuar), un contenedor IoC **se encarga de instanciar, configurar y ensamblar estos objetos.**

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

<a id="service-locator"></a>

#### Localizador de Servicios (Service Locator)
Esté patrón de diseño propone tener un registro centralizado **("service locator")** que conoce todas las dependencias disponibles en la aplicación. Cuando un objeto necesita una dependencia, le pide explícitamente al service locator una instancia de esa dependencia.

<a id="applicacion-context"></a>
### ApplicationContext


<a id="bean-vs-component"></a>
#### @Bean vs @Component


<a id="controller-service-repository"></a>
#### Las anotaciones @RestController, @Service y @Repository


<a id="anotacion-configuration"></a>
#### La anotación @Configuration

<a id="ejemplo-application-context"></a>
#### Ejemplo del funcionamiento de ApplicacionContext
