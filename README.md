# Spring Boot

## Tabla de Contenido

- [¿Qué es Spring Boot?](#que-es-spring-boot)
  - [¿Qué es la Inversión de Control (IoC)?](#que-es-la-inversion-de-control)
    - [Inyección de dependecias](#inyeccion-de-dependecias)
  - [ApplicationContext](#applicacion-context)
    - [@Bean vs @Component](#bean-vs-component)
    - [Scope (Ámbito)](#scope-de-un-bean)
    - [Las anotaciones @RestController, @Service y @Repository](#controller-service-repository)
    - [La anotación @Configuration](#anotacion-configuration)
    - [Ejemplo del funcionamiento de ApplicacionContext](#ejemplo-application-context)
    - [La anotación @SpringBootApplication](#anotacion-springboot-application)
  - [La autoconfiguración de Spring Boot](#autoconfiguracion-de-spring-boot)
    - [¿Qué son los spring-boot-starter-*?](#spring-boot-starter)
    - [La configuración de spring-boot-starter-web](#starter-web)
    - [La configuración de spring-boot-starter-data-jpa](#stater-data)
    - [La configuración de spring-boot-starter-security](#stater-security)
  - [Aplicaciones autónomas](#aplicaciones-autonomas)
    - [Artefactos: JAR vs WAR](#jar-vs-war)
    - [Servidores embebidos](#servidores-embebidos)
- [Configuración de aplicaciones](#configuracion-de-applicaciones)
  - [Archivos .properties y YAML](#properties-y-yaml)
    - [La anotación @Value](#la-anotacion-value)
    - [La anotación @ConfigurationProperties](#la-anotacion-configuration-properties)
    - [Propiedades de línea de comandos](#propiedades-cli)
    - [Propiedades Java del sistema](#propiedades-java)
  - [Perfiles dentro de Spring Boot](#perfiles-en-spring)
    - [Configuración de Beans de acuerdo al perfil activo](#configuracion-beans-por-perfil)
  - [Configuración externa y centralizada](#configuracion-externa-y-centralizada)
    
<a id="que-es-spring-boot"></a>
## ¿Qué es Spring Boot?

Spring Boot es una herramienta que simplifica la creación de aplicaciones Java basadas en Spring. Ofrece una configuración predeterminada y reduce la necesidad de configuración manual, permitiendo a los desarrolladores enfocarse en la lógica de negocio.

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

<a id="anotacion-configuration"></a>
#### La anotación @Configuration

Gracias a esta anotación a nivel de clase ayuda a definir una **fuente de definiciones de beans**. Debido que, Spring identifica las clases anotadas con @Configuration y las procesa para descubrir y registrar los beans dentro del contenedor IoC.

> Dependencias dentro de la definición con @Bean
>
> - Dentro de un método que sea marcado con @Bean, puede invocar otros métodos @Bean dentro de la misma clase @Configuration. Spring se encargará de invocar el método dependiente y proveer la instancia del bean necesario.
> - También se pueden declarar dependencias como parámetros en los métodos marcados con @Bean. Spring buscará un bean del tipo especificado en el contexto y lo inyectará automáticamente.

En el código siguiente tenemos la definición de una clase que define Beans dentro de Spring. Dentro de ella define un Bean de tipo **ProcesadorMensajes** que a su vez define una dependencia **ServicioMensajes** de tal manera que el contexto de Spring **(ApplicationContext)** buscará una instancia de está clase y se la inyectará a este Bean.

```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
    @Bean
    public ProcesadorMensajes procesador(ServicioMensajes servicio) {
        return new ProcesadorMensajes(servicio);
    }
}
```
> [!IMPORTANT]
> Se ha mencionado que para la creación de Beans dentro de Spring se puede realizar a través de anotaciones y código de java.
>
> 1. Los beans cómo por ejemplo, @Component, @Service, etc. Son ejemplos de Beans creados por **anotaciones**
> 2. Los beans creados dentro de una clase @Configuration son ejemplos de Beans creados por **código de java**


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


<a id="anotacion-springboot-application"></a>
#### La anotación @SpringBootApplication


<a id="autoconfiguracion-de-spring-boot"></a>
### La autoconfiguración de Spring Boot

La autoconfiguración es el corazón de la magia de Spring Boot. Su objetivo principal es reducir significativamente la cantidad de configuración manual que se necesita escribir para poner en marcha una aplicación Spring. 

En esencia, Spring Boot **examina las dependencias** que vas añadiendo a tu proyecto (principalmente a través del archivo pom.xml o build.gradle) y, basándose en esas dependencias, **configura automáticamente** una gran cantidad de beans de Spring con valores predeterminados.

<a id="spring-boot-starter"></a>
#### ¿Qué son los spring-boot-starter-*?

Los spring-boot-starter-* son un conjunto de dependencias convenientes que **agrupan todas las bibliotecas** relacionadas que típicamente se utilizan juntas **para una funcionalidad específica**. 

> Beneficios
> 
> - Spring Boot gestiona las versiones compatibles de las dependencias transitivas. Esto reduce los problemas de compatibilidad entre diferentes bibliotecas.
> - Incluyen la autoconfiguración necesaria para que las dependencias que agrupan funcionen de forma predeterminada con poca o ninguna configuración adicional.

<a id="starter-web"></a>
#### La configuración de spring-boot-starter-web

Cuando se incluye la dependencia spring-boot-starter-web (a través del del archivo pom.xml o build.gradle) en tu proyecto, Spring Boot automáticamente configura una serie de beans esenciales para construir aplicaciones web.

> ¿Qué configura este starter por defecto?
>
> - Spring MVC: El framework Spring MVC se configura automáticamente, incluyendo los componentes necesarios para manejar las peticiones HTTP, como **DispatcherServlet, HandlerMapping, HandlerAdapter, etc**
> - Configura la implementación (por defecto Jackson) de soporte para serializar y deserializar objetos Java a JSON.
> - Servidor embebido: Por defecto, Spring Boot incluye y configura un servidor Tomcat embebido. Esto significa que no necesitas desplegar tu aplicación en un servidor web externo para la ejecución de la aplicación. 

> [!NOTE]
> Un starter de vital importancia es el **spring-boot-starter-validation**
> - Este starter de Spring Boot, configura automáticamente un validador basado en la especificación Bean Validation (normalmente Hibernate Validator)
> - Si deseas tener más información de **Spring Web MVC**, visita mi repositorio: <a href="https://github.com/devdevbadillo/spring_web_mvc_theory">Ir</a>

<a id="stater-data"></a>
#### La configuración de spring-boot-starter-data-jpa

Esté starter tiene cómo objetivo el simplificar el trabajo con bases de datos relacionales en aplicaciones Spring **utilizando la Java Persistence API (JPA)** a través de la implementación de Hibernate.

> ¿Qué configura este starter por defecto?
>
> - Spring Boot intenta autoconfigurar una fuente de datos (DataSource) basada en la información de conexión a la base de datos que sea proporcionada en los **archivos .propiedades o YAML**.
> - Se configura un **EntityManagerFactory**, que es el punto de entrada principal para las operaciones de persistencia en JPA. Hibernate actúa como el proveedor de persistencia subyacente.
> - Simplifica la implementación de la capa de acceso a datos proporcionando una forma de definir interfaces de repositorio que se implementan automáticamente en tiempo de ejecución.

<a id="stater-security"></a>
#### La configuración de spring-boot-starter-security

El starter de **spring-boot-starter-security** proporciona las dependencias necesarias para integrar la seguridad dentro de la aplicación de Spring

> ¿Qué configura este starter por defecto?
> 
> - SecurityFilterChain: Spring Security utiliza una cadena de filtros (FilterChain) para interceptar y procesar las peticiones HTTP. Spring Boot autoconfigura una cadena de filtros básica que aplica la autenticación y la autorización.
> - Al incluir este starter sin ninguna configuración adicional, Spring Boot aplica una seguridad básica basada en **"HTTP Basic Authentication"**. Esto significa que cuando intentas acceder a cualquier endpoint de tu aplicación, el navegador te pedirá un nombre de usuario y una contraseña


<a id="aplicaciones-autonomas"></a>
### Aplicaciones autónomas

El concepto de "aplicaciones autónomas" (souper-jar o fat jar) es una de las características distintivas dentro del framework de Spring Boot. Se refiere a la capacidad de empaquetar una aplicación Spring Boot completa, incluyendo todas sus dependencias (como el servidor web embebido, las bibliotecas de la aplicación, las dependencias de Spring, etc.), en un único archivo ejecutable. Este archivo suele ser un **JAR (Java Archive)**.

> Beneficios de las aplicaciones autónomas
>
> - Se ha mencionado que el archivo **JAR** contiene todo lo necesario para ejecutar la aplicación, esto permite la portablilidad de la aplicación frente a diferentes entornos (diferentes sistemas operativos, proveedores de nube, etc.) siempre y cuando se tenga una Java Virtual Machine (JVM) compatible.
> - Uno de los beneficios más importantes es la ejecución de la aplicación, pues, solo basta con ejecutar el comando: **java -jar <nombre_del_archivo>.jar**. Esto es genial, debido que facilita la integración con herramientas como Docker y Kubernetes.

Ya sabemos qué es una aplicación autónoma y que beneficios nos provee está característica, pero, ¿Cómo crea dichas aplicaciones el framework de Spring Boot?

> Respuesta: Spring Boot utiliza el plugin spring-boot-maven-plugin (para Maven) o org.springframework.boot (para Gradle) para empaquetar la aplicación como un JAR ejecutable. Este plugin reestructura el JAR estándar para que pueda ser ejecutado. En esencia, añade un "loader" especial que sabe cómo encontrar y cargar las clases y dependencias empaquetadas dentro del JAR.

<a id="jar-vs-war"></a>
#### Artefactos: JAR vs WAR

En el mundo de las aplicaciones Java, los formatos de empaquetado más comunes para desplegar aplicaciones son JAR (Java Archive) y WAR (Web Application Archive).

> JAR ( Java Archive)

Originalmente, los JARs se utilizaban principalmente para empaquetar bibliotecas de código Java (APIs) para que pudieran ser incluidas como dependencias en otros proyectos Java, pero, con la llegada de Spring Boot y los servidores embebidos, el JAR se ha convertido en un formato viable para empaquetar y desplegar aplicaciones web completas. 

>  WAR

Un archivo WAR es un formato de empaquetado específico para aplicaciones web Java que siguen la especificación de Servlets y JavaServer Pages (JSP)

Esté archivo se basa en una estructura de directorios bien definida que un servidor de aplicaciones Java EE (como Tomcat, Jetty, WildFly, etc.) espera para **poder desplegar y ejecutar la aplicación web**. Esta estructura incluye directorios como WEB-INF (que contiene la configuración de la aplicación web, clases compiladas, bibliotecas JAR privadas), META-INF (que contiene metadatos del archivo WAR), y otros directorios para recursos web (HTML, CSS, JavaScript, imágenes, JSPs, etc.).

> ¿Cuándo elegir JAR o WAR con Spring Boot?
>
> - JAR (autónomo): Generalmente es la opción preferida para microservicios, aplicaciones que se desplegarán en contenedores (Docker), o cuando se busca simplicidad en el despliegue y no se requiere un servidor de aplicaciones Java EE específico.
> - WAR (para servidor externo): Se elige cuando hay un requisito de desplegar la aplicación en un servidor de aplicaciones Java EE existente que es gestionado por la organización o cuando se necesitan características específicas proporcionadas por ese servidor de aplicaciones

<a id="servidores-embebidos"></a>
#### Servidores embebidos

Un servidor embebido es un servidor web que se **empaqueta directamente dentro de la aplicación**. En el contexto de Spring Boot, esto significa que en lugar de depender de un servidor web externo (como un Tomcat, Jetty o WildFly instalado por separado), la aplicación **incluye una instancia del servidor** web dentro de su archivo ejecutable (JAR).

Spring Boot configurará automáticamente un servidor por defecto, simplemente incluyendo el starter dentro de las dependencias en el proyecto (dentro del pom.xml o build.grandle).


| Starter                           	      |  Descripción                                          	|
|------------------------------------------	|------------------------------------------------------		|
| `spring-boot-starter-web`                 | El servidor configurado por **defecto** es Tomcat. Esté es  un servidor Servlet popular y maduro.		                                                                        |                         
| `spring-boot-starter-jetty`               | Jetty es otro servidor Servlet ligero. Esté puede utilizar siempre y cuando se excluya la dependencia de **spring-boot-starter-web**                                          |
| `spring-boot-starter-undertow` 		        | Un servidor web flexible y de alto rendimiento desarrollado por Red Hat. Esté puede utilizar siempre y cuando se excluya la dependencia de **spring-boot-starter-web**	      |


> [!NOTE]
> ¿Qué es un servidor Servlet?
> Un servidor Servlet en Java es un programa que se ejecuta en un servidor web y que está diseñado para **manejar las peticiones de los clientes** (generalmente navegadores web) y generar respuestas dinámicas.
>
> El proceso que se lleva acabo es:
> 1. Recibe la petición: El Servlet recibe la petición del cliente.
> 2. Procesa la petición: El Servlet puede acceder a bases de datos, realizar cálculos, interactuar con otros servicios, etc., para preparar la información necesaria para la respuesta.
> 3. Prepara la respuesta: El Servlet genera una respuesta dinámica, puede ser XML, JSON, imágenes, etc.
> 4. Entrega la respuesta: El Servlet envía esta respuesta de vuelta al cliente a través del servidor web.


<a id="configuracion-de-applicaciones"></a>
## Configuración de aplicaciones

La configuración de aplicaciones en Spring Boot se centra en cómo **externalizar** los parámetros de tu aplicación para que puedas modificar su comportamiento sin necesidad de recompilar el código. Esto es crucial para personalizar la aplicación según las necesidades.

Spring Boot ofrece varias maneras de configurar tu aplicación, siendo los archivos .properties y YAML (.yml o .yaml) los más comunes.

<a id="properties-y-yaml"></a>
### Archivos .properties y YAML

> Archivos .properties

Son archivos de texto plano donde las propiedades se **definen en pares clave-valor**, separados por el signo igual (=).

> Ejemplo ilustrativo:

```
server.port=8080
spring.datasource.url=jdbc:h2:mem:mydb
spring.datasource.username=sa
spring.datasource.password=
myapp.message=¡Hola desde properties!
```

> Archivos YAML

YAML es un **formato de serialización** de datos legible por humanos. Utiliza la indentación para definir la estructura jerárquica y es más expresivo que .properties para configuraciones complejas.

> Ejemplo ilustrativo:
```
server:
  port: 8080
spring:
  datasource:
    url: jdbc:h2:mem:mydb
    username: sa
    password: ""
myapp:
  message: ¡Hola desde YAML!
```

Spring Boot cargará automáticamente los archivos application.properties o application.yml (o application.yaml) que encuentre en el classpath de la aplicación (normalmente en la carpeta src/main/resources). 

> [!NOTE]
> Si ambos tipos de archivos están presentes, las propiedades de YAML tienen prioridad


<a id="la-anotacion-value"></a>
#### La anotación @Value

La anotación @Value es una herramienta poderosa para **inyectar valores de configuración** (valores definidos en los archivos .properties o YAML) directamente a los Beans definidos dentro de la aplicación de Spring.

> Uno de los usos principales es para obtener secretos definidos en las properties y ser usados para realizar una función que puede ser compartida en diferentes capas de la aplicación.

```
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class MyUtil {

    @Value("${jwt.key}")
    private String secret;

    public void generateToken() {
      /* ...Bussiness logic... */
    }
```

> [!NOTE]
>
> Se puede especificar un valor por defecto usando el operador :${defaultValue}. Si la propiedad no se encuentra en los archivos de configuración, se utilizará el valor por defecto.
> Ejemplo:
> - @Value("${myapp.message:Hola mundo}")

<a id="la-anotacion-configuration-properties"></a>
#### La anotación @ConfigurationProperties

La anotación @ConfigurationProperties ofrece una forma más estructurada y robusta de trabajar con grupos de propiedades relacionadas. En lugar de inyectar propiedades individuales con @Value, puedes **mapear un prefijo de propiedades** a un objeto Java.


Para ejemplificar el uso de está anotación supongamos que dentro de nuestro archivo de configuración se tienen las siguientes propiedades:
  
```
myapp:
  message: Mensaje desde ConfigurationProperties
  environment: production    
```

Notemos que el prefijo de estas configuraciones es **myapp**, entonces, en el siguiente código tenemos ese mapeo por medio del prefijo hacía una instancia de la clase MyAppProperties que tiene los atributos de **message y environment**
que a su vez los valores de estos atributos son los definidos en el archivo de configuración.

```
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties("myapp")
public class MyAppProperties {

    private String message;
    private String environment;

    public String getMessage() {
        return message;
    }

    public String getEnvironment() {
        return environment;
    }
}
```

<a id="propiedades-cli"></a>
#### Propiedades de línea de comandos

Spring Boot permite pasar argumentos directamente a la aplicación cuando es ejecutada desde la línea de comandos. Estos argumentos se convierten **automáticamente** en propiedades de Spring y **tienen una mayor precedencia** sobre otras fuentes de configuración (como los archivos application.properties o YAML).

> Ejemplo:
>
> java -jar <nombre_del_ejecutable_de_la_aplicacion>.jar --server.port=3000

En este caso, la propiedad server.port se establecerá en 3000, y esto **sobrescribirá** cualquier valor que pueda haber sido definido para la propiedad **server.port**.

> [!NOTE]
>
> Estas propiedades se vuelven parte del entorno de Spring y puedes acceder a ellas de la misma manera que accedes a cualquier otra propiedad de configuración, utilizando @Value o @ConfigurationProperties.

<a id="propiedades-java"></a>
#### Propiedades Java del sistema

Las propiedades Java del sistema son pares clave-valor que se establecen en la máquina virtual Java (JVM) durante su inicio. Spring Boot también integra estas propiedades en su mecanismo de configuración, aunque con una **precedencia menor** que las propiedades de línea de comandos pero **mayor** que las propiedades de los archivos de configuración.

> Ejemplo: 
> - Las propiedades del sistema se establecen utilizando la opción -D al ejecutar la JVM:
> java -Dserver.port=8085 -jar mi-aplicacion.jar

> [!NOTE]
> 
> Al igual que las propiedades de línea de comandos, puedes acceder a las propiedades del sistema en tus beans de Spring utilizando @Value o @ConfigurationProperties

<a id="perfiles-en-spring"></a>
### Perfiles dentro de Spring Boot

Un perfil en Spring Boot representa un **conjunto de configuraciones** que se activan bajo ciertas condiciones. Esto te permite definir diferentes beans, propiedades y configuraciones específicas para cada entorno, manteniendo el código base de la aplicación de una manera limpia y organizada.

> Definición de perfiles a través de los archivos de configuración

La forma más común de definir configuraciones específicas para un perfil es creando archivos de configuración con el siguiente patrón de nomenclatura: application-{profile}.properties o application-{profile}.yml (o .yaml).


| Perfil                           	        |  Descripción                                          	|
|------------------------------------------	|------------------------------------------------------		|
| `application.properties`                  | Contiene la configuración por defecto que se aplica en todos los perfiles.            |                         
| `application-dev.properties`              | Contiene configuraciones específicas para el perfil de desarrollo.                    |
| `application-prod.yaml` 		              | Contiene configuraciones específicas para el perfil de producción.                    |

Supongamos que se tienen los siguientes archivos de configuración dentro del classpath src/main/resources:

```
# application.properties:
server.port=8080
spring.datasource.url=jdbc:h2:mem:defaultdb
spring.datasource.username=sa
spring.datasource.password=
myapp.environment=default

# application-dev.properties:
spring.datasource.url=jdbc:h2:mem:devdb
spring.datasource.username=devuser
spring.datasource.password=devpass
myapp.environment=development

# application-prod.properties:
spring.datasource.url=jdbc:mysql://prod.example.com:3306/proddb
spring.datasource.username=produser
spring.datasource.password=prodsecret
myapp.environment=production

```

Entonces, ¿Cómo puede Spring tomar las configuraciones de acuerdo al perfil activo al momento del arranque de la aplicación?

Esto se hace a través del archivo de configuración por defecto (application.properties) con la declaración siguiente:

```
spring:
  profiles:
    active: dev
```

> [!NOTE]
> En esté caso, la aplicación utilizará la URL, el usuario y la contraseña de la base de datos definidos en application-dev.properties, y el valor de myapp.environment será **development**


<a id="configuracion-beans-por-perfil"></a>
#### Configuración de Beans de acuerdo al perfil activo

Dentro de Spring se pueden definir beans que solo se **instancien** cuando un determinado perfil esté activo utilizando la **anotación @Profile**.

> Ejemplo de su implementación:

```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration
public class DatabaseConfig {

    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        // Configuración de la fuente de datos para desarrollo (por ejemplo, H2 en memoria)
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("org.h2.Driver");
        dataSource.setUrl("jdbc:h2:mem:devdb");
        dataSource.setUsername("devuser");
        dataSource.setPassword("devpass");
        return dataSource;
    }

    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        // Configuración de la fuente de datos para producción (por ejemplo, MySQL)
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://prod.example.com:3306/proddb");
        dataSource.setUsername("produser");
        dataSource.setPassword("prodsecret");
        return dataSource;
    }

    @Bean
    @Profile("!prod") // Bean que se instancia en todos los perfiles EXCEPTO 'prod'
    public MyDevelopmentTool developmentTool() {
        return new MyDevelopmentTool();
    }

    @Bean
    @Profile({"dev", "test"}) // Bean que se instancia en los perfiles 'dev' Y 'test'
    public DebugService debugService() {
        return new DebugService();
    }
}
```

> De esté ejemplo tenemos que:
> - El bean devDataSource solo se creará cuando el perfil dev esté activo.
> - El bean prodDataSource solo se creará cuando el perfil prod esté activo.
> - El bean developmentTool se creará en todos los perfiles excepto prod.
> - El bean debugService se creará cuando los perfiles dev o test (o ambos) estén activos.


<a id="configuracion-externa-y-centralizada"></a>
### Configuración externa y centralizada
