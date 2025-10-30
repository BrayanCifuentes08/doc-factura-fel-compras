# DOCUMENTACIÓN DE APLICACIÓN FLUTTER
El proyecto está organizado siguiendo una arquitectura modular que facilita la escalabilidad, mantenibilidad y separación de responsabilidades.
A continuación se describe la estructura principal del sistema asignacion_contable_sat:

## 🧱 Estructura del Proyecto Flutter

El proyecto está organizado siguiendo una arquitectura modular que facilita la escalabilidad, mantenibilidad y separación de responsabilidades.
A continuación se describe la estructura principal del sistema asignacion_contable_sat:

### 📁 lib/

Contiene todo el código fuente del proyecto Flutter, incluyendo controladores, modelos, servicios, notifiers y pantallas de la aplicación.

### 📂 common/

Carpeta destinada a componentes y clases de uso común dentro de la aplicación.
Incluye notifiers, configuraciones compartidas y utilidades globales.
not_factura_fel_compras.dart → Contiene el Notifier que gestiona el estado y la lógica de negocio relacionada con las facturas FEL de compras.

### 📂 models/

Define las estructuras de datos utilizadas en la aplicación, reflejando las entidades del backend o base de datos.
factura_fel_compras_m.dart → Modelo de datos para representar las facturas FEL de compras, con sus campos y métodos de serialización (fromJson / toJson).

### 📂 services/

Contiene la lógica de conexión con los endpoints del backend y manejo de peticiones HTTP.
compras_sat_service.dart → Servicio responsable de realizar las solicitudes HTTP al API, incluyendo migración, actualización y consulta de datos relacionados con compras FEL.

### 📂 widgets/

Incluye las pantallas, layouts y componentes visuales reutilizables del sistema.

detalle_procesados.dart → Pantalla para mostrar el detalle de los registros procesados (ejecutados y no ejecutados).
layout.dart → Define la estructura visual base del sistema, incluyendo organización de vistas, temas y disposición general.
traslado_datos_sat.dart → Pantalla principal para realizar el traslado y migración de datos desde archivos Excel hacia la base de datos FacturaFELCompras.

⚙️ Archivos de Configuración del Proyecto

pubspec.yaml → Define las dependencias del proyecto (paquetes, SDK de Flutter y assets).


## ⚙️ Configuración del Proyecto

Para poder ejecutar la aplicación asignacion_contable_sat, se deben seguir los siguientes pasos:

**1. Requisitos**

- Tener instalado Flutter SDK versión >=1.17.0 y compatible con Dart ^3.5.4.
- Tener un editor de código compatible, como VS Code o Android Studio.
- Configurar el emulador o dispositivo físico para pruebas.
  
**2. Instalación de dependencias**

El proyecto utiliza pubspec.yaml para gestionar dependencias. Para instalar todas las librerías necesarias, ejecutar:

```bash
flutter pub get
```
Esto instalará paquetes como:

- flutter_localizations → Para soporte de múltiples idiomas.
- intl e intl_utils → Para manejo de internacionalización.
- flutter_speed_dial → Para botones de acción flotantes.
- material_design_icons_flutter → Para íconos de Material Design.
- flutter_initicon → Para generar avatares con iniciales.
- file_picker → Para selección de archivos (Excel).
- font_awesome_flutter → Para íconos adicionales.
  
Dependencias locales business y core vía path relativo.

**3. Generación de archivos de internacionalización**

El proyecto tiene habilitado Flutter Intl. Para generar los archivos necesarios a partir de los .arb:
```bash
flutter pub run intl_utils:generate
```

**4. Ejecución de la aplicación**

Para correr la aplicación en modo desarrollo:
```bash
flutter run
```

Si se desea especificar un dispositivo:
```bash
flutter run -d <device_id>
```

**5. Información adicional de configuración**

El archivo pubspec.yaml define la versión de la app (0.0.1) y que no se publicará en pub.dev (publish_to: 'none').
La sección flutter: contiene generate: true para generar automáticamente ciertos archivos de configuración interna.
El proyecto utiliza notifiers, modelos, servicios y widgets organizados en carpetas modulares para mantener la estructura limpia y escalable.

## Uso de la Aplicación

La aplicación asignacion_contable_sat está diseñada para la gestión de facturas FEL, migración de datos desde Excel y actualización de cuentas contables mediante APIs REST. A continuación se describen los componentes principales y el flujo de uso.

### Flujo de Trabajo

- Ingreso a la aplicación y navegación al módulo: Primero, el usuario inicia sesión y se cargan los datos de empresa, estación de trabajo y usuario. Después se llega al layout         principal de la pantalla de inicio, donde en el drawer se seleccionará la app Factura FEL Compras, que tiene como user display FacturaFelCompras. Esta acción redirige al layout del módulo de FacturaFelCompras.
- Migración de datos: Mediante la pantalla TrasladoDatosSat, se seleccionan archivos Excel que se migran y validan.
- Procesamiento de facturas: Se realiza la actualización de las facturas en la base de datos a través de los servicios HTTP definidos en compras_sat_service.dart.
- Visualización de resultados y asignación de cuentas: Se pueden revisar los registros ejecutados y no ejecutados en el widget detalle_procesados.dart.
  Además, se puede asignar una cuenta contable a cada factura, registrándose en cuentaContableCuentaCorrentista y actualizando campo cuenta en FacturaFelCompras.

### Notifier y Gestión de Estado (not_factura_fel_compras.dart)

La aplicación utiliza Provider y ChangeNotifier para manejar el estado de los datos de manera reactiva. En particular, FacturaFELComprasNotifier gestiona todas las operaciones relacionadas con las facturas FEL de compras.

**1. Propiedades principales**

- `compras` → Lista principal de todas las compras cargadas desde Excel o backend.
- `comprasEjecutadas` → Lista de compras que fueron procesadas correctamente.
- `comprasNoEjecutadas` → Lista de compras que no se pudieron procesar.
- `comprasSeleccionadas` → Conjunto de IDs de compras seleccionadas mediante checkboxes.
- `searchCompraController` → Controlador de texto para búsqueda de compras por NIT.
- `focusSearchCompra` → FocusNode para el campo de búsqueda.
- `_cargandoCompras` y `_cargandoComprasDesc` → Indicadores de carga para mostrar spinners o estados de espera.
- Variables de totales (`sumaIVA`, `sumaGranTotal`, `sumaImpuestoPetroleo`, etc.) → Acumulan valores de todos los registros procesados.

**2. Métodos principales**

- `calcularTotalesEjecutadas()` → Calcula automáticamente los totales de todas las compras ejecutadas, incluyendo impuestos, tasas y otros valores específicos como TurismoHospedaje o TimbrePrensa.
- `clearCompras()` → Limpia la lista de compras y las compras seleccionadas, además de remover el foco del campo de búsqueda.
- `clearData()` → Reinicia todos los totales, limpia las listas de compras y resetea los controladores de búsqueda y el foco.

**3. Notificación de cambios**

Todas las propiedades importantes usan setters con notifyListeners(), lo que permite que cualquier widget que escuche este notifier se actualice automáticamente cuando cambian los datos. Esto asegura que la interfaz refleje siempre el estado actual de las compras y sus totales.

**4. Flujo de uso dentro de la app**

- Las compras se cargan en compras desde un archivo Excel o backend.
- El usuario puede seleccionar compras, buscar por NIT o filtrar registros.
- Al procesar las compras, se separan en comprasEjecutadas y comprasNoEjecutadas.
- Los totales se calculan automáticamente usando calcularTotalesEjecutadas().
- Se utiliza `clearCompras()` o `clearData()` para limpiar la información o reiniciar el proceso y preparar la app para uno nuevo.


### 📦 Modelos de Datos (factura_fel_compras_m.dart)

El proyecto utiliza modelos para representar las entidades del backend y estructurar la información que se maneja en la aplicación. En particular, FacturaFelComprasM representa una factura FEL de compras.

**1. Propiedades principales**

El modelo incluye los siguientes campos:

- `empresa` → Identificador de la empresa.
- `nit` → Número de identificación tributaria del emisor.
- `nombreEmisor` → Nombre del emisor de la factura.
- `serie` y `noDocto` → Serie y número de documento de la factura.
- `fechaDocumento` → Fecha de emisión de la factura.
- `granTotal` → Monto total de la factura.
- `iva` → Impuesto al valor agregado de la factura.
- `marcaAnulado` y `fechaAnulacion` → Indican si la factura fue anulada y cuándo.
- Impuestos y tasas adicionales: impuestoPetroleo, turismoHospedaje, turismoPasajes, timbrePrensa, bomberos, tasaMunicipal, bebidasAlcoholicas, bebidasNoAlcoholicas, tabaco, cemento, tarifaPortuaria.
- `cuenta` e `id_Cuenta` → Información de la cuenta contable asignada a la factura.
- `userName` → Usuario que procesó la factura.
- `fechaHora` → Fecha y hora de registro en la aplicación.

**2. Serialización y deserialización**

El modelo incluye métodos para convertir objetos desde JSON y hacia JSON, facilitando la comunicación con APIs REST:
```dart
// Convertir de JSON a objeto
FacturaFelComprasM factura = FacturaFelComprasM.fromJson(jsonData);

// Convertir de objeto a JSON
Map<String, dynamic> jsonData = factura.toJson();
```

**3. Uso en la aplicación**

Se utiliza dentro de FacturaFELComprasNotifier para almacenar, filtrar y calcular totales de las facturas.
Cada registro representa una factura completa, incluyendo información de totales y cuentas contables asignadas.
Facilita la integración con las pantallas de la aplicación y la comunicación con los servicios REST del backend.

### 🌐 Servicios y Conexión con APIs (compras_sat_service.dart)

La aplicación utiliza la clase ComprasSatService para gestionar la comunicación con el backend a través de APIs REST, tanto para obtener datos como para migrar información desde archivos Excel.

**1. Obtener facturas FEL de compras**

El método `getFacturasFELCompras` permite obtener la lista completa de facturas desde el servidor.
```dart
List<FacturaFelComprasM> facturas = await ComprasSatService.getFacturasFELCompras(
  baseUrl: 'https://mi-servidor.com/',
  context: context,
);
```
**- Parámetros:**

  - `baseUrl` → URL base del backend.
  - `context` → Contexto de Flutter para obtener el token del usuario.

**- Funcionamiento:**

  - Se obtiene el token de autenticación mediante LoginService.
  - Se realiza una solicitud GET al endpoint /PaFacturaFELComprasSelCtrl.
  - Se decodifica el JSON y se mapea a una lista de FacturaFelComprasM.

**- Retorno:** Lista de facturas obtenidas del servidor.
**- Errores:** Lanza una excepción si la respuesta HTTP no es exitosa.

**2. Trasladar datos desde Excel**

El método trasladarDatos permite migrar datos de un archivo Excel al backend y devuelve las facturas que no pudieron ejecutarse.
```dart
List<FacturaFelComprasM> noEjecutadas = await ComprasSatService.trasladarDatos(
  baseUrl: 'https://mi-servidor.com',
  nombreHoja: 'Hoja1',
  archivoPath: '/ruta/archivo.xlsx',
  context: context,
);
```
**- Parámetros:**

  - `baseUrl` → URL base del backend.
  - `nombreHoja` → Nombre de la hoja del archivo Excel.
  - `archivoPath` → Ruta local del archivo Excel.
  - `context` → Contexto de Flutter para obtener token y datos de usuario.

**- Funcionamiento:**

  - Se obtiene el token, la empresa y el usuario mediante LoginService.
  - Se construye una solicitud multipart POST al endpoint /PaFacturaFELComprasICtrl con el archivo Excel.
  - Se envía la solicitud y se procesa la respuesta.
  - Se extraen las facturas no ejecutadas en caso de errores o inconsistencias.

**- Retorno:** Lista de facturas que no pudieron ser procesadas.
**- Errores:** Lanza una excepción si la respuesta HTTP no es exitosa.

**3. Integración con UI**

Los métodos de ComprasSatService se utilizan dentro de pantalla de TrasladoDatosSat para:

Cargar las facturas iniciales desde el backend.
Migrar los datos desde archivos Excel.
Actualizar automáticamente los estados de comprasEjecutadas y comprasNoEjecutadas.
Permitir al usuario visualizar resultados y realizar acciones sobre las facturas.

### Layout Principal (layout.dart)

El widget Layout es el contenedor principal de la aplicación. Gestiona:

- El drawer lateral con apartado de ajustes y navegación a pantalla de inicio (DrawerPersonalizado).
- La barra superior dinámica (CustomSliverAppBar) que muestra el título de la acción en curso.
- El body principal donde se carga la pantalla de traslado de datos (TrasladoDatosSat).
- El footer con información adicional de la sesión (Footer).
- El Floating Action Button central (HomeFAB) para acceso rápido al home.

**1. Notifiers y Estados**

La aplicación utiliza Provider para manejar el estado:

- `FacturaFELComprasNotifier` → Gestiona la información de facturas FEL de compras, incluyendo los datos migrados desde Excel y su validación.
- `CuentaContableNotifier` → Administra la información de cuentas contables mediante APIs.
- `ThemeNotifier` y `AppStyleNotifier` → Gestionan temas y estilos visuales de la aplicación.


**4. Servicios y APIs**

- `compras_sat_service.dart` → Se encarga de realizar las llamadas REST al backend para migración y actualización de datos.
- `UtilitiesService` → Proporciona utilidades generales como control de scroll, confirmaciones de salida y validaciones.

**5. Comportamiento de la Interfaz**

- El FAB se oculta automáticamente cuando el usuario desea ingresar texto en inputs.
- La barra superior es dinámica.
- El drawer lateral permite resetear notifiers y limpiar datos antes de realizar nuevas migraciones.

### TrasladoDatosSat (traslado_datos_sat.dart)

**Descripción:**
El widget TrasladoDatosSat permite al usuario trasladar datos desde archivos Excel hacia el backend, visualizar facturas FEL de compras, buscar por NIT y asignar cuentas contables a las facturas. También gestiona el estado de carga y muestra mensajes de éxito o error al usuario.

**1. Propiedades principales del widget**

`isBackgroundSet`, `imagePath`, `temaClaro`, `token`, `pUserName`, `pEmpresa`, `pEstacion_Trabajo`, `fechaSesion`, `fechaExpiracion`, `despEmpresa`, `despEstacion_Trabajo`, `userController`
→ Parámetros requeridos para inicializar el widget y mantener contexto de sesión, empresa y usuario.

**2. Variables de estado**

- `_cargandoTraslado` → indica si se está ejecutando un traslado de datos.
- `_cargandoHojas` → indica si se están cargando las hojas de Excel.
- `_mostrarAsignacion` → controla la visibilidad de la sección de asignación de cuentas.
- `_archivoSeleccionado` → almacena el archivo Excel seleccionado.
- `_nombresHojasSeleccionadas` → lista de hojas de Excel que se van a procesar.
- `_nombreHojaSeleccionada` → hoja actualmente seleccionada.
- `clientes`, `clienteSeleccionado` → manejo de cuentas correntistas.
- `cuentaContableNotifier`, `comprasFELNotifier`, `aplicacionesNotifier` → notifiers para actualizar la UI y manejar datos.

**3. Métodos principales**

**`_msgSeleccionarArchivo()`**
Muestra un diálogo de alerta si el usuario no ha seleccionado un archivo Excel antes de iniciar el traslado.

**`_trasladarDatos()`**
- Valida si se seleccionó un archivo.
- Itera sobre la hoja seleccionadas y llama a `ComprasSatService.trasladarDatos` para enviar los datos al backend.
- Actualiza el notifier comprasFELNotifier con las facturas no ejecutadas.
- Muestra mensajes de éxito o error por la hoja procesada.
- Refresca la lista de facturas ejecutadas con _verFacturaFelCompras().

**`_verFacturaFelCompras()`**
- Llama al servicio `ComprasSatService.getFacturasFELCompras` para obtener las facturas ejecutadas.
- Actualiza `comprasFELNotifier.comprasEjecutadas` y calcula los totales de los campos financieros.
- Maneja errores con diálogos de reintento.

**`_buscarFacturaFelCompras()`**
- Limpia datos previos y ejecuta búsqueda de facturas por NIT.
- Llama a `_BuscarCliente()` para asociar cuentas correntistas si aplica.
- Consulta el endpoint `PaFacturaFELComprasGCtrl` y actualiza `comprasFELNotifier.compras`.

**`_actualizarCuentaFacturaFelCompras()`**
- Inserta la relación entre cuenta contable y cuenta correntista usando _insertarCtaContableCtaCtista().
- Realiza la actualización de la cuenta contable de la factura en el backend mediante PaFacturaFELComprasUCtrl.
- Muestra mensajes de éxito y refresca la lista de facturas.

**`_BuscarCliente()`**
- Busca cuentas correntistas asociadas al NIT ingresado.
- Actualiza la lista clientes y selecciona automáticamente el primer cliente.
- Maneja errores mostrando un diálogo de reintento.

**`_insertarCtaContableCtaCtista()`**
- Inserta la relación entre la cuenta contable seleccionada y la cuenta correntista en el backend.
- Actualiza la UI para reflejar el estado de carga y muestra errores si ocurre algún fallo.

**4. Método `build`**
El método `build` es el núcleo visual del widget `TrasladoDatosSat`. Se encarga de construir la interfaz de usuario, mostrando tanto la sección de **traslado de datos desde Excel** como la sección de **asignación de cuentas contables**. A continuación, se describe cada parte:

**- Contexto y ThemeNotifier**
```dart
final themeNotifier = Provider.of<ThemeNotifier>(context);
```
- Obtiene el tema actual de la aplicación mediante Provider.
- Permite alternar colores y estilos entre tema claro y tema oscuro.

**- Contenedor principal y scroll**
```dart
  SliverToBoxAdapter(
  child: Padding(
    child: SingleChildScrollView(
      child: Container(
        constraints: BoxConstraints(minHeight: MediaQuery.of(context).size.height),
        decoration: BoxDecoration(...),
```
- SliverToBoxAdapter: Permite que este widget sea usado dentro de un CustomScrollView.
- SingleChildScrollView: Habilita el desplazamiento vertical.
- Container con BoxDecoration: Cambia el color según el tema y si se establece un background con imagen.
- Bordes redondeados y shadow para estética.
- Soporta imagen de fondo si widget.isBackgroundSet es true.

**Botones de acción principales**
```dart
Wrap(
  children: [
    // Botón "Migrar Excel"
    Tooltip(...),
    // Botón "Asignar Cuenta"
    Container(...),
  ],
)
```
- Migrar Excel: Vuelve a la sección de traslado de datos.
- Asignar Cuenta: Muestra la sección de asignación de cuenta contable.
- Uso de Wrap para mantener responsividad en varias resoluciones.
- Ambos botones utilizan estilos adaptativos según el tema.

**Interfaz de traslado de datos (Excel)**
```dart
if (!_mostrarAsignacion)
  ClipRRect(
    borderRadius: BorderRadius.circular(16.0),
    child: Container(
      decoration: BoxDecoration(...),
      child: ExpansionTile(
        title: Text('Seleccionar Archivo Excel'),
        children: [
          ExcelUploader(...),
          if (comprasFELNotifier.cargandoComprasDesc)
            LoadingComponent(color: Colors.blue),
          DetalleProcesados(...),
          TotalCard(...) // Totales de las facturas procesadas
        ],
      )
    )
  )
```

- ExpansionTile: Contenedor expandible que agrupa los componentes del traslado de datos.
- ExcelUploader: Widget personalizado para cargar archivos Excel, seleccionar hojas y ejecutar el traslado.
- DetalleProcesados: Muestra dos pestañas:
  - Ejecutados: Compras correctamente procesadas.
  - No Ejecutados: Compras que no pudieron procesarse.
- TotalCard: Muestra los totales agrupados por categoría (IVA, Turismo, Bomberos, etc.).
- Visual feedback con LoadingComponent durante la carga de datos.

**Sección de asignación de cuentas contables**
```dart
if (_mostrarAsignacion)
  Column(
    children: [
      TextField(...), // Buscar Facturas por NIT
      if (comprasFELNotifier.cargandoCompras)
        LoadingComponent(color: Colors.blue),
      TablaGenerica<FacturaFelComprasM>(...), // Tabla de facturas filtradas
      if (comprasFELNotifier.comprasSeleccionadas.isNotEmpty)
        CuentaContableDetalle(...), // Selección de cuenta contable
      if (cuentaContableNotifier.cuentaSeleccionada != null)
        ElevatedButton(...), // Botón Asignar
    ],
  )
```
- Permite buscar facturas FEL por NIT y seleccionarlas para asignación de cuenta contable.
- TextField con prefixIcon y suffixIcon para buscar y limpiar.
- TablaGenerica: Muestra las facturas filtradas y permite selección mediante checkboxes.
- CuentaContableDetalle: Permite elegir la cuenta contable para la asignación.
- Botón Asignar: Ejecuta _actualizarCuentaFacturaFelCompras luego de confirmación en mostrarDialogo.

**Comportamiento adaptativo**

- Todos los colores y estilos se adaptan dinámicamente al tema claro/oscuro.
- Los widgets de carga y mensajes muestran retroalimentación visual al usuario.
- Se maneja la selección múltiple de facturas y la actualización de notifiers (comprasFELNotifier y cuentaContableNotifier) para mantener el estado sincronizado.


### DetalleProcesados (detalle_procesados.dart)
El widget `DetalleProcesados` muestra los **detalles de los registros procesados** en dos pestañas: una para los registros **no ejecutados** y otra para los **ejecutados**. Incluye tablas dinámicas, interacción con el portapapeles y animaciones en la selección de pestañas.

**1. Propiedades principales**

- `tituloTab1` / `tituloTab2`: Títulos de las pestañas.
- `iconoTab1` / `iconoTab2`: Íconos asociados a cada pestaña.
- `colorTab1` / `colorTab2`: Colores de los títulos e íconos de cada pestaña.
- `itemsTab1` / `itemsTab2`: Listas de objetos `FacturaFelComprasM` que se mostrarán en cada pestaña.

Estas propiedades se pasan mediante el **constructor** del widget y permiten personalizar el contenido y estilo de las pestañas.

**2. Estado interno**

- `_tabController`: Controla la navegación y animación entre las dos pestañas.
- Se inicializa en `initState()` y se libera en `dispose()` para evitar fugas de memoria.

**3. Build Method**

El método `build` construye toda la interfaz de usuario:

**3.1 Proveedores utilizados**
```dart
final themeNotifier = Provider.of<ThemeNotifier>(context);
final comprasFELNotifier = Provider.of<FacturaFELComprasNotifier>(context);
final colorNotifier = Provider.of<AppStyleNotifier>(context);
```

- themeNotifier: Determina el tema claro u oscuro.
- comprasFELNotifier: Maneja el estado de las compras FEL.
- colorNotifier: Permite obtener colores personalizados de los botones y elementos UI.

**3.2 Colores adaptativos**
```dart
final Color subtitulo = themeNotifier.temaClaro ? Colors.grey[800]! : Colors.grey[400]!;
final Color fondo = themeNotifier.temaClaro ? Colors.grey[100]! : Color.fromARGB(47, 134, 134, 134);
```
- Colores de fondo y subtítulos cambian según el tema.

**3.3 AnimatedBuilder**

- Permite animar la transición de colores en los TabBar cuando se cambia de pestaña.
- animationValue determina el estado de animación actual.

**3.4 Estructura del contenedor principal**
Contenedor con altura fija de 590px y bordes redondeados.
Column que incluye:
Encabezado superior con ícono y título.
TabBar: Pestañas superiores con íconos y colores animados.
TabBarView: Contenido de cada pestaña.

**4. TabBar (Pestañas superiores)**
```dart
TabBar(
  controller: _tabController,
  indicatorColor: colorNotifier.getBtnColor(themeNotifier.temaClaro),
  labelColor: Colors.transparent,
  unselectedLabelColor: subtitulo,
  tabs: [...]
)
```

- Pestañas construidas con _buildColoredTab, que permite animación de color según la pestaña activa.
- Cada pestaña muestra el título y la cantidad de elementos (widget.itemsTabX.length).

**5. TabBarView (Contenido de las pestañas)**

**5.1 Pestaña 1 – No Ejecutados**

- Contiene un TablaGenerica con los datos de itemsTab1.
- No incluye botón de copiar NIT.

**5.2 Pestaña 2 – Ejecutados**

- Contiene un TablaGenerica con los datos de itemsTab2.
- Añade botón de copiar NIT para cada fila:
  - Copia el NIT al portapapeles.
  - Actualiza el searchCompraController de comprasFELNotifier.
  - Muestra un SnackBar confirmando la acción.
 
**6. Función _buildColoredTab**
```dart
Widget _buildColoredTab({ ... })
```
- Construye una pestaña con animación de color según su estado.
- Calcula la distancia entre la pestaña activa y la inactiva (animationValue - tabIndex).
- Interpola el color del texto entre el color activo y el inactivo.
- Devuelve un widget Tab con ícono y texto.

**7. Resumen de funcionalidades**

- Visualización tabular de registros ejecutados y no ejecutados.
- Interacción con portapapeles para copiar NIT en registros ejecutados.
- Animación en TabBar para destacar la pestaña activa.
- Adaptación a temas claros y oscuros.
- Uso de proveedores para:
  - Estado de compras FEL.
  - Tema de la aplicación.
  - Colores personalizados de botones y elementos.



