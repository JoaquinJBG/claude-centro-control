# Qué mirar en cada dimensión

Adapta cada lista a la tecnología y salta lo que no aplique (dilo en el inventario).

## Por componente

### 1. Calidad y mantenibilidad
Legibilidad; separación de responsabilidades; cohesión y acoplamiento; convenciones del lenguaje,
del framework y del proyecto; duplicación y lógica de negocio repetida; funciones demasiado
complejas; abstracciones innecesarias o patrones mal aplicados; código muerto, imports y
dependencias sin usar; tipado, contratos, interfaces y validaciones; gestión de errores y
excepciones; manejo de recursos y fugas; asincronía, concurrencia y condiciones de carrera; uso
incorrecto de librerías o APIs; deuda técnica que frene cambios futuros.

### 2. Bugs y errores funcionales
Errores lógicos; condiciones límite; nulos, vacíos, indefinidos o mal formados; entradas inválidas
y tipos inesperados; flujos incompletos y estados inconsistentes; validaciones; cálculos y
precisión; fechas, horarios y zonas horarias; paginación y grandes volúmenes; concurrencia; errores
silenciosos y excepciones mal gestionadas; reintentos, timeouts y fallos de servicios externos;
operaciones que se ejecutan a medias y dejan datos inconsistentes; diferencias entre lo esperado y
lo implementado.

### 3. Seguridad
Autenticación y autorización; control de acceso y permisos; sesiones, tokens y credenciales;
secretos expuestos; validación y sanitización de entradas; inyecciones (SQL, NoSQL, comandos);
XSS, CSRF, SSRF cuando proceda; endpoints sin protección; escalada de privilegios y acceso a
recursos de otros clientes o usuarios; CORS, cabeceras y políticas de contenido; datos personales y
sensibles; información en logs, respuestas y mensajes de error; ficheros, rutas, cargas y
descargas; dependencias vulnerables y cadena de suministro; configuraciones inseguras de desarrollo
o producción; límites de peticiones y abuso de recursos; permisos entre front, back, APIs y base de
datos; integraciones externas; secretos y variables de entorno en el despliegue. No te limites a lo
conocido: mira errores de diseño y combinaciones de funciones que abran un fallo de autorización.

### 4. Usabilidad y experiencia (si hay interfaz)
Navegación; coherencia de interacciones; facilidad de las tareas principales; formularios y
validaciones; mensajes de error y recuperación; estados de carga, vacío, éxito y fallo; acciones sin
feedback; comportamientos confusos; responsive y móvil; accesibilidad y teclado; etiquetas,
jerarquía visual y contraste; consistencia entre pantallas; rendimiento percibido; bloqueos de la
interfaz. Prueba los flujos principales como lo haría un usuario real cuando se pueda.

### 5. Rendimiento y eficiencia
Consultas ineficientes y N+1; operaciones redundantes; llamadas innecesarias a servicios externos;
renderizados excesivos; procesamiento innecesario; consumo de memoria o CPU; recursos pesados;
caché mal usada; falta de índices; consultas sin paginación; cuellos de botella; escalabilidad;
conexiones y recursos. Sin medición es Hipótesis: di qué prueba o medición haría falta.

### 6. Pruebas y fiabilidad
Unitarias, integración y end-to-end; cobertura de lo crítico; casos límite sin cubrir; pruebas
frágiles o dependientes del entorno; aserciones insuficientes; tests que no verifican el
comportamiento real (¿fallan sin el arreglo?); mocks y datos de prueba; falsos positivos; scripts de
lint, tipos y compilación; fiabilidad de los procesos de validación.

## Transversal

### A. Contradicciones entre componentes
Reglas de negocio implementadas distinto en módulos distintos; funciones que se contradicen;
documentación frente a implementación; configuraciones incompatibles; validaciones que uno acepta y
otro rechaza; estados representados de forma distinta; lógica duplicada que ha divergido;
responsabilidades que un componente asume y otro no cumple; contratos que no coinciden;
incompatibilidades de versiones o dependencias; flujos que funcionan aislados y fallan integrados.

### B. Contratos, tipos y formatos
Productor frente a consumidor; compatibilidad de esquemas; tipos y campos obligatorios; valores por
defecto; tratamiento de nulos; formatos de fechas, identificadores y cantidades; serialización;
versionado de APIs; compatibilidad entre front, back y persistencia; migraciones y cambios de
esquema; consistencia entre modelos, validaciones y almacenamiento.

### C. Arquitectura e integraciones
Dependencias circulares; acoplamiento excesivo; límites de responsabilidad mal definidos; flujos de
datos difíciles de mantener; puntos únicos de fallo; errores entre servicios; timeouts y reintentos
incompatibles; consistencia transaccional y fallos parciales; diferencias entre entornos;
dependencias innecesarias; integraciones que no gestionan sus errores.

### D. Trazabilidad de extremo a extremo
Traza cada flujo crítico de principio a fin y comprueba que las capas están alineadas:
interfaz → API → lógica → base de datos; autenticación → autorización → acceso al recurso; entrada
→ validación → transformación → persistencia; acción del usuario → estado → respuesta de la
interfaz; solicitud externa → procesamiento → respuesta y errores.

## Ejemplos según el stack

No son listas cerradas: úsalos solo si el stack del proyecto coincide, y deduce los equivalentes
si no.

- **Si es web con front y back separados** (React/Vue/Next, etc. + API): tipos del front frente al
  esquema real de la API; validación solo en cliente; permisos que el front oculta pero la API no
  comprueba; datos sensibles en el bundle o en variables públicas (`NEXT_PUBLIC_*`, `VITE_*`);
  renderizado servidor/cliente que diverge; caché de peticiones obsoleta tras una mutación.
- **Si usa ORM y base de datos relacional**: N+1; migraciones irreversibles o que bloquean tablas
  grandes; índices que faltan para los filtros reales; transacciones que no cubren la operación
  entera; consultas en crudo con interpolación de cadenas.
- **Si es multi-tenant o multi-cliente**: cada consulta filtra por tenant/usuario; IDs adivinables
  que dan acceso a recursos ajenos; jobs y colas que pierden el contexto del tenant.
- **Si es Python** (Django, FastAPI, Flask...): `DEBUG` y `SECRET_KEY` por entorno; `pickle`/`eval`
  sobre datos externos; dependencias sin fijar; tareas asíncronas mezcladas con código bloqueante.
- **Si es Node/TypeScript**: `any` y aserciones que esconden contratos rotos; promesas sin
  `await` ni `catch`; `strict` desactivado; lockfile coherente con el `package.json`.
- **Si es móvil o escritorio**: almacenamiento local de tokens; permisos del sistema; modo sin
  conexión y reintentos; actualizaciones con migraciones de datos locales.
- **Si hay infraestructura como código o CI/CD** (Docker, Kubernetes, Terraform, pipelines):
  secretos en el repo o en logs del pipeline; imágenes base sin fijar; contenedores como root;
  diferencias entre entornos que el código da por iguales; pasos de CI que no fallan cuando deben.
- **Si maneja dinero, cantidades o medidas físicas**: precisión decimal (nunca coma flotante para
  importes), redondeos, unidades y conversiones, zonas horarias en cortes diarios o mensuales.
