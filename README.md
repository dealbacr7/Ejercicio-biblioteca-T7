Reparto de roles

👤 Persona 1: Arquitecto/a e infraestructura (Singleton)  JAVI

Configuracion (Singleton): máximo de préstamos por usuario, días de préstamo, número de copias online (5)…
GestorBD (Singleton): registro central de préstamos y reservas. Debe ser thread-safe, con un threading.Lock en __new__ (doble comprobación) y otro lock en cada operación de lectura o escritura.
Define al principio los contratos comunes: las clases abstractas y las firmas de los métodos (ver más abajo).
Hace el esquema de arquitectura e integra el main final.

👤 Persona 2: Catálogo de ítems (Factory Method)  SERGIO

Clase abstracta ItemPrestamo y sus productos concretos:
LibroFisico y Revista: se pueden prestar.
MaterialConsulta: se consulta bajo petición y no puede salir de la biblioteca. Si alguien intenta prestarlo, se lanza una excepción o se ofrece solo consulta en sala.
LibroOnline: tiene un máximo de 5 accesos simultáneos, controlados con threading.Semaphore(5) o un contador protegido con lock.
Creadores: la clase abstracta CreadorItem con crear_item(), y sus subclases CreadorLibroFisico, CreadorRevista, CreadorConsulta y CreadorOnline.

👤 Persona 3: Lógica del préstamo (State + Command)  RAFA

State: EstadoPrestamo (abstracta) con prestar(), devolver() y reservar(). Sus estados concretos son Disponible, Prestado y Reservado, y cada uno define qué transiciones permite y cuáles dan error.
Command: la interfaz Comando con ejecutar() y deshacer(). Los comandos concretos son PrestarCommand, DevolverCommand y ReservarCommand.
Invocador TerminalBiblioteca, que ejecuta los comandos y guarda un historial para poder deshacer.
Cada comando llama al ítem, y el ítem delega la acción en su estado actual.

👤 Persona 4: Notificaciones, concurrencia y UML (Observer + hilos)  CARLOS

Observer: el ítem es el sujeto (suscribir, desuscribir, notificar) y Usuario es el observador (update). Cuando el ítem vuelve a Disponible, se avisa a quienes estaban esperando.
Simulación multihilo: varios usuarios intentan coger el mismo libro a la vez, usando threading.Thread o ThreadPoolExecutor. La prueba debe comprobar que solo uno lo consigue con un libro físico, y que como máximo 5 lo consiguen con un libro online.
Pruebas de condiciones de carrera.
Diagrama UML de clases del proyecto completo.



Cómo encajan las piezas
Usuario (Observer) ──► TerminalBiblioteca (Invoker)
                            │ ejecuta
                            ▼
              PrestarCommand / DevolverCommand / ReservarCommand
                            │ llama a
                            ▼
   CreadorItem ──crea──► ItemPrestamo (Subject) ──delegа──► EstadoPrestamo
   (Factory Method)         │                                (Disponible/Prestado/Reservado)
                            │ registra (con Lock)
                            ▼
                    GestorBD + Configuracion (Singleton)




Para no bloquearse entre vosotros
Día 1, todos juntos: acordáis los contratos, que son ItemPrestamo, EstadoPrestamo, Comando, Observer y los métodos de GestorBD. Con eso cada persona puede programar su parte usando mocks de las demás.
Decisiones que conviene cerrar pronto:
En LibroOnline, el estado Prestado solo se activa cuando están ocupadas las 5 copias.
MaterialConsulta nunca pasa a Prestado.
Quién hace la llamada a notificar(): lo más limpio es que la haga el ítem cuando cambia de estado.
Dónde va el lock: puede ir en el ítem, en el GestorBD o en ambos. Recomiendo un lock por ítem, porque así dos usuarios que piden libros distintos no se bloquean entre sí.
Integración: la Persona 1 une las partes y la Persona 4 lanza la simulación con hilos sobre el sistema completo.
