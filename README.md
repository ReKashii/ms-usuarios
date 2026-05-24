### 📂 Estructura de Paquetes y Código Creado                                                                                                                                                     
                                                                                                                                                                                                    
  Los archivos fuentes se han estructurado en el directorio  ms-usuarios/src/main/java/cl/bookpointchile/usuarios :                                                                                 
                                                                                                                                                                                                    
    ms-usuarios/                                                                                                                                                                                    
    ├── pom.xml                                     # Dependencias del proyecto (Lombok, Validation, JPA)                                                                                           
    └── src/                                                                                                                                                                                        
        └── main/                                                                                                                                                                                   
            ├── java/                                                                                                                                                                               
            │   └── cl/                                                                                                                                                                             
            │       └── bookpointchile/                                                                                                                                                             
            │           └── usuarios/                                                                                                                                                               
            │               ├── config/                                                                                                                                                             
            │               │   └── DataInitializer.java  # Sembrador automático de roles y administrador base                                                                                      
            │               ├── controller/                                                                                                                                                         
            │               │   └── UsuarioController.java # Controladores REST expuestos                                                                                                           
            │               ├── dto/                                                                                                                                                                
            │               │   ├── UsuarioRegistroRequestDTO.java # JSR 380 para creación de cuentas                                                                                               
            │               │   ├── ActualizarRolRequestDTO.java   # JSR 380 para cambio de perfil                                                                                                  
            │               │   └── UsuarioResponseDTO.java        # DTO de salida CSR                                                                                                              
            │               ├── exception/                                                                                                                                                          
            │               │   ├── ResourceNotFoundException.java                                                                                                                                  
            │               │   ├── EmailYaRegistradoException.java # HTTP 409 Conflict                                                                                                             
            │               │   ├── UsuarioNoEncontradoException.java # HTTP 404 Not Found                                                                                                          
            │               │   ├── ErrorResponse.java             # JSON uniforme de error                                                                                                         
            │               │   └── GlobalExceptionHandler.java    # Interceptor global @RestControllerAdvice                                                                                       
            │               ├── model/                                                                                                                                                              
            │               │   ├── Rol.java                       # Roles base de negocio                                                                                                          
            │               │   └── Usuario.java                   # Entidad transaccional de usuarios                                                                                              
            │               ├── repository/                                                                                                                                                         
            │               │   ├── RolRepository.java                                                                                                                                              
            │               │   └── UsuarioRepository.java         # Acceso a datos e índices de unicidad                                                                                           
            │               ├── service/                                                                                                                                                            
            │               │   ├── UsuarioService.java                                                                                                                                             
            │               │   └── UsuarioServiceImpl.java        # Reglas de negocio e inicio de sesión                                                                                           
            │               └── UsuariosApplication.java           # Bootstrap de Spring Boot                                                                                                       
            └── resources/                                                                                                                                                                          
                └── application.properties                  # Configuración de puerto, MySQL y logs                                                                                                 
  ──────                                                                                                                                                                                            
  ### 🛠️ Decisiones de Desarrollo y Diseño                                                                                                                                                         
                                                                                                                                                                                                    
  1. Integridad de Datos en Persistencia:                                                                                                                                                           
      • Usuario.java incluye restricciones de unicidad explícitas ( @UniqueConstraint ) tanto en la columna de  email  como en la de  rut . Esto evita de forma estricta registros duplicados en  
      el motor de base de datos MySQL.                                                                                                                                                              
      • La relación con Rol.java se define como un  @ManyToOne(fetch = FetchType.EAGER) , cargando el rol de manera directa puesto que siempre es necesario para el control de accesos del    
      sistema.                                                                                                                                                                                      
  2. Validaciones Beans Estrictas (JSR 380):                                                                                                                                                        
      • UsuarioRegistroRequestDTO.java valida que la contraseña tenga un tamaño mínimo ( @Size(min=8) ), que el formato del correo sea el correcto ( @Email ), y que los nombres o RUT no estén vacíos
( @NotBlank ).
  3. Arquitectura y Excepciones de Negocio Uniformes:                                                                                                                                               
      • La validación se gestiona de forma centralizada. Si el usuario intenta registrar un correo o un RUT que ya existe, UsuarioServiceImpl.java interrumpe el flujo, imprime una alerta usando 
@Slf4j  ( 
      log.warn ) y arroja una excepción  EmailYaRegistradoException .                                                                                                                               
      • GlobalExceptionHandler.java captura este evento y retorna un estado HTTP 409 Conflict con un payload JSON estructurado, ideal para la integración CSR.                                                  
  4. Flujos Inteligentes por Defecto:                                                                                                                                                               
      • Al registrar un usuario, si el parámetro  rolId  se envía nulo, el servicio asume de forma automática el rol de "Cliente Web". Esto permite que los flujos de registro público en la web no 
      requieran especificar un perfil, manteniendo la seguridad y la simplicidad técnica.                                                                                                           
  5. Carga Automática de Roles y Pruebas (Data Seeder):                                                                                                                                             
      • DataInitializer.java inyecta de forma automática los 4 perfiles obligatorios al arrancar el microservicio:                                                                                       
          1. "Administrador del Sistema"                                                                                                                                                            
          2. "Jefe de Sucursal"                                                                                                                                                                     
          3. "Asistente de Ventas"                                                                                                                                                                  
          4. "Cliente Web"                                                                                                                                                                          
      • Además, si no hay usuarios registrados, crea de forma inmediata una cuenta de prueba para el evaluador:                                                                                     
          • Email:  admin@bookpoint.cl                                                                                                                                                              
          • Contraseña:  admin1234                                                                                                                                                                  
          • Rol: Administrador del Sistema                                                                                                                                                          
                                                                                                                                                                                                    
                                                                                                                                                                                                    
  ──────                                                                                                                                                                                            
  ### ⚙️ Propiedades del Entorno                                                                                                                                                                     
                                                                                                                                                                                                    
  En application.properties se han ajustado las siguientes variables:                                                                                                                                      
                                                                                                                                                                                                    
  • Puerto:  server.port=8083  (independiente de ms-ventas  8081  y ms-inventario  8082 ).                                                                                                          
  • Base de datos: MySQL esquema  bookpoint_usuarios .                                                                                                                                              
  • Logging: Activado a nivel  INFO  para el rastreo en consola de creaciones y modificaciones de permisos.                                                                                         
  ──────                                                                                                                                                                                            
  ### 🔍 Endpoints REST Expuestos                                                                                                                                                                   
                                                                                                                                                                                                    
  •  POST /api/usuarios/registro : Registro de usuarios del sistema (Clientes Web o perfiles administrativos).                                                                                      
  •  GET /api/usuarios/{id} : Obtiene la información detallada de una cuenta mediante su identificador.                                                                                             
  •  PUT /api/usuarios/{id}/rol : Endpoint administrativo exclusivo del Administrador del Sistema para cambiar o promover a un usuario de perfil (ej. promover a un Asistente a Jefe de Sucursal).
