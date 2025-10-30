# Módulo SAT - Documentación

## Descripción General

El **Módulo SAT** es un módulo de Angular diseñado para gestionar la asignación de datos relacionados con el SAT (Sistema de Administración Tributaria). Este módulo implementa un sistema de layout centralizado que mantiene consistencia visual y funcional en todas sus vistas.

### Características Principales
- Layout responsivo con sidebar colapsable
- Sistema de drawer para configuraciones
- Barra de estado con información de usuario, estación de trabajo y empresa
- Routing anidado para gestión de vistas
- Protección de rutas con guards

---

## Arquitectura del Módulo

El módulo sigue una arquitectura basada en componentes con un layout principal que encapsula todas las vistas secundarias:

```
SatLayoutComponent (Contenedor Principal)
├── Sidebar (Navegación lateral)
├── Header (Encabezado del módulo)
├── Settings Drawer (Panel de configuración)
├── Router Outlet (Vistas dinámicas)
└── Footer/Barra de Estado
```

---

## Componentes Principales

### SatLayoutComponent

**Ubicación:** `sat-layout.component.ts`

**Descripción:**  
Componente contenedor principal del módulo SAT. Se encarga de mantener todo el contenido del módulo y se renderiza según el display seleccionado por el usuario.

#### Propiedades

| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `headerText` | `string` | Texto del encabezado del módulo |
| `usuario` | `string` | Nombre del usuario actual |
| `estacionTrabajo` | `string` | Descripción de la estación de trabajo |
| `empresa` | `string` | Nombre de la empresa |
| `drawerVisible` | `boolean` | Controla la visibilidad del drawer de configuraciones |

#### Ciclo de Vida

**ngOnInit():**
```typescript
ngOnInit() {
  // Suscripción a la visibilidad del drawer
  this.sharedService.getDrawerVisibility().subscribe((visible) => {
    this.drawerVisible = visible;
  });
  
  // Obtención de datos de usuario
  this.usuario = this.loginService.getUser();
  
  // Obtención de estación de trabajo
  const estacionTrabajo = this.loginService.getEstacion()
  this.estacionTrabajo = estacionTrabajo?.descripcion || '';
  
  // Obtención de empresa
  const empresa = this.loginService.getEmpresa();
  this.empresa = empresa?.nombre || '';
  
  // Suscripción al texto del header
  this.sharedService.currentHeaderText.subscribe(text => {
    this.headerText = text || '';
  });
}
```

#### Estructura del Template

El componente utiliza **CSS Grid** para crear un layout responsivo de altura completa:

```html
<div class="h-screen grid grid-rows-[1fr_auto]">
  <!-- Contenido principal con sidebar y router-outlet -->
  <div class="flex overflow-hidden">
    <!-- Navegación lateral -->
    <app-sidebar-common></app-sidebar-common>
    
    <!-- Área de contenido -->
    <div class="flex-1 overflow-y-auto p-4 xl:ml-80">
      <!-- Header del módulo -->
      <nav class="w-full">
        <app-header-common
          [titulo]="'MODULO SAT'"
          [mostrarSidebarBtn]="true"
        />
      </nav>
      
      <!-- Drawer de configuraciones -->
      <app-settings-drawer
        [visible]="drawerVisible"
        (closed)="sharedService.closeDrawer()"
      >
      </app-settings-drawer>
      
      <!-- Outlet para vistas hijas -->
      <router-outlet></router-outlet>
    </div>
  </div>
  
  <!-- Footer fijo -->
  <div class="xl:ml-80">
    <app-barra-estado
      [conAccion]="false"
      [usuario]="usuario"
      [estacionTrabajo]="estacionTrabajo"
      [empresa]="empresa"
    >
    </app-barra-estado>
  </div>
</div>
```
---

## Rutas y Navegación

### Configuración de Rutas

**Archivo:** `sat.routes.ts`

```typescript
export const SATRoutes: Routes = [
  {
    path: 'asignacion-datos-sat',
    component: SatLayoutComponent,
    title: 'Asignacion Datos Sat',
    canActivate: [AuthGuard, AppSelectionGuard],
    children: [
      { 
        path: 'proceso-sat', 
        component: TrasladoDatosSatComponent 
      },
      { 
        path: '', 
        redirectTo: 'proceso-sat', 
        pathMatch: 'full' 
      },
    ],
  },
];
```

### Descripción de Rutas

#### Ruta Principal: `/asignacion-datos-sat`

| Configuración | Valor | Descripción |
|---------------|-------|-------------|
| **Path** | `asignacion-datos-sat` | Ruta base del módulo |
| **Component** | `SatLayoutComponent` | Componente contenedor |
| **Title** | `Asignacion Datos Sat` | Título de la página |
| **Guards** | `[AuthGuard, AppSelectionGuard]` | Protección de autenticación y selección de aplicación |

#### Rutas Hijas

1. **Proceso SAT**
   - **Path:** `proceso-sat`
   - **Component:** `TrasladoDatosSatComponent`
   - **URL Completa:** `/asignacion-datos-sat/proceso-sat`

2. **Redirección por Defecto**
   - **Path:** `` (vacío)
   - **Redirect:** `proceso-sat`
   - **Comportamiento:** Redirige automáticamente a `proceso-sat` cuando se accede a la ruta base

### Guards de Protección

#### AuthGuard
Verifica que el usuario esté autenticado antes de acceder al módulo.

#### AppSelectionGuard
Valida que se haya seleccionado correctamente la aplicación antes de permitir el acceso.

### Flujo de Navegación

```
Usuario accede → Guards verifican → Layout carga → Redirección automática → Vista de proceso-sat
```

---

## Servicios Utilizados

### SharedService
- **Función:** Gestión de estado compartido entre componentes
- **Métodos utilizados:**
  - `getDrawerVisibility()`: Observable para la visibilidad del drawer
  - `closeDrawer()`: Cierra el drawer de configuraciones
  - `currentHeaderText`: Observable para el texto del header

### LoginService
- **Función:** Gestión de datos de autenticación y sesión
- **Métodos utilizados:**
  - `getUser()`: Obtiene el nombre del usuario actual
  - `getEstacion()`: Obtiene información de la estación de trabajo
  - `getEmpresa()`: Obtiene información de la empresa

---

## TrasladoDatosSatComponent

**Ubicación:** `traslado-datos-sat.component.ts`

**Descripción:**  
Componente principal para la carga y procesamiento de archivos Excel con facturas FEL (Factura Electrónica en Línea). Permite al usuario subir un archivo Excel, seleccionar la hoja a procesar, trasladar los datos al sistema y visualizar los resultados del procesamiento.

### Flujo de Operación

```
Usuario carga Excel → Sistema valida archivo → Extrae hojas disponibles → 
Usuario selecciona hoja → Traslada datos → Backend procesa → 
Sistema muestra resultados (guardados y faltantes)
```

### Propiedades Principales

#### Control de Archivos
| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `fileSeleccionado` | `File \| null` | Archivo Excel seleccionado por el usuario |
| `hojas` | `string[]` | Lista de hojas detectadas en el archivo Excel |
| `hojaSeleccionada` | `string` | Nombre de la hoja actualmente seleccionada |
| `draggingFile` | `boolean` | Estado de arrastre de archivo (drag and drop) |

#### Banderas de Carga
| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `cargandoHojas` | `boolean` | Indica si se están cargando las hojas del Excel |
| `cargandoTraslado` | `boolean` | Indica si se está procesando el traslado de datos |
| `cargandoRegistrosFacturaFELCompras` | `boolean` | Indica si se están obteniendo los registros procesados |

#### Datos de Facturas FEL
| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `facturasFelCompras` | `PaFacturaFELComprasM[]` | Registros de facturas guardados exitosamente |
| `dataRegistrosFaltantes` | `any[]` | Registros que no pudieron ser procesados |
| `comprasFELKeys` | `string[]` | Nombres de las columnas para la tabla de resultados |

#### Totales de Impuestos
El componente mantiene una estructura detallada de sumatorias por cada tipo de impuesto:

- **sumaGranTotal**: Total general de todas las facturas
- **sumaIVA**: Total del Impuesto al Valor Agregado
- **sumaImpuestoPetroleo**: Total de impuesto sobre petróleo
- **sumaTurismoHospedaje**: Total de impuesto turístico de hospedaje
- **sumaTurismoPasajes**: Total de impuesto turístico de pasajes
- **sumaTimbrePrensa**: Total de timbre de prensa
- **sumaBomberos**: Total de contribución de bomberos
- **sumaTasaMunicipal**: Total de tasa municipal
- **sumaBebidasAlcoholicas**: Total de impuesto sobre bebidas alcohólicas
- **sumaBebidasNoAlcoholicas**: Total de impuesto sobre bebidas no alcohólicas
- **sumaTabaco**: Total de impuesto sobre tabaco
- **sumaCemento**: Total de impuesto sobre cemento
- **sumaTarifaPortuaria**: Total de tarifa portuaria

#### Control de Vistas
| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `mostrarDetalleProcesados` | `boolean` | Controla la visibilidad de la tabla de resultados |
| `mostrandoAsignarCuenta` | `boolean` | Controla la visibilidad del módulo de asignación de cuentas |
| `nitCopiado` | `string \| null` | NIT seleccionado para asignación de cuenta contable |

#### Mensajes y Modales
| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `mostrarMensajeAdvertencia` | `boolean` | Muestra advertencia de archivo duplicado |
| `isVisibleModal` | `boolean` | Controla el modal de confirmación de traslado |
| `isVisibleExito` | `boolean` | Muestra mensaje de éxito |
| `mensajeExito` | `string` | Contenido del mensaje de éxito |
| `isVisibleAlerta` | `boolean` | Muestra mensaje de alerta/error |
| `mensajeAlerta` | `string` | Contenido del mensaje de alerta |


### Métodos Principales

#### 1. ngOnInit()
**Propósito:** Inicialización del componente
**Acciones:**
- Reinicia la selección de cuenta contable
- Obtiene el símbolo de moneda de la empresa

#### 2. cargarFile(event: any)
**Propósito:** Procesa la carga de un archivo Excel desde el input
**Flujo:**
- Valida que no exista un archivo previamente cargado
- Muestra advertencia si hay duplicado (se oculta automáticamente después de 7 segundos)
- Llama a `migrarExcelService.obtenerHojasExcel()` para extraer las hojas
- Gestiona estados de carga y errores

**Validaciones:**
- Solo permite un archivo a la vez
- Debe ser formato `.xlsx`

#### 3. onDragOver(event: DragEvent) y onDragLeave(event: DragEvent)
**Propósito:** Gestiona los eventos de arrastrar y soltar (drag and drop)
**Comportamiento:**
- Activa el estado visual `draggingFile` al arrastrar sobre el área
- Desactiva el estado al salir del área

#### 4. seleccionarHoja(sheet: string)
**Propósito:** Establece la hoja seleccionada del Excel
**Acciones:**
- Asigna el nombre de la hoja a `hojaSeleccionada`
- Cierra el dropdown de selección

#### 5. removerFile()
**Propósito:** Elimina el archivo cargado y reinicia el estado del componente
**Acciones:**
- Limpia el input de archivo
- Reinicia todas las propiedades relacionadas con archivos
- Oculta las tablas de resultados
- Limpia los datos de facturas y registros faltantes

#### 6. mostrarModalTraslado()
**Propósito:** Muestra el modal de confirmación antes de trasladar los datos

#### 7. trasladarDatosCompras()
**Propósito:** Envía el archivo Excel al backend para procesar las facturas
**Flujo:**
- Valida que exista un archivo seleccionado
- Prepara el modelo con: usuario, empresa, archivo y nombre de hoja
- Activa indicador de carga
- Reinicia arrays de datos
- Llama al servicio `comprasSATService.trasladarDatosComprasSAT()`
- Al recibir respuesta exitosa:
  - Extrae registros no ejecutados (faltantes)
  - Llama a `verFacturaFELCompras()` para obtener registros guardados
  - Muestra mensaje de éxito
  - Limpia la hoja seleccionada después de 2 segundos
- Gestiona errores mostrando modal de error

#### 8. verFacturaFELCompras()
**Propósito:** Obtiene los registros de facturas procesados correctamente desde el backend
**Flujo:**
- Activa indicador de carga
- Llama a `comprasSATService.getRegistrosFacturasFelCompras()`
- Procesa la respuesta:
  - Asigna las facturas a `facturasFelCompras`
  - Calcula sumatorias de todos los tipos de impuestos usando `reduce()`
  - Extrae las keys para las columnas de la tabla
  - Activa la visualización de resultados (`mostrarDetalleProcesados = true`)
- Gestiona errores y desactiva la visualización en caso de fallo

#### 9. tarjetasTotales (Getter)
**Propósito:** Genera un array de objetos con los totales de impuestos para visualización
**Retorna:** Array de objetos con:
- `label`: Nombre del impuesto
- `valor`: Total calculado
- `color`: Color de borde para la tarjeta (Tailwind CSS)

**Filtrado:** Solo retorna impuestos con valor mayor a 0

#### 10. recibirNit(nit: string)
**Propósito:** Recibe el NIT seleccionado desde el componente hijo `DetalleProcesados`
**Acciones:**
- Almacena el NIT en `nitCopiado`
- Activa automáticamente la vista de asignación de cuenta

#### 11. toggleHistorial() y ocultarHistorial()
**Propósito:** Controla la visibilidad del módulo de asignación de cuenta
**Uso:** Permite alternar entre la vista de resultados y la asignación de cuentas

### Características de la Interfaz

#### Sección de Carga de Archivo
- **Drag and Drop**: Área interactiva con efectos visuales al arrastrar archivos
- **Validación visual**: Muestra información del archivo seleccionado (nombre, tamaño)
- **Feedback inmediato**: Spinner mientras se cargan las hojas

#### Sección de Selección de Hoja
- **Dropdown personalizado**: Lista desplegable con todas las hojas del Excel
- **Confirmación visual**: Chip que muestra la hoja seleccionada
- **Estado deshabilitado**: El botón de traslado solo se activa con hoja seleccionada

#### Sección de Resultados
Se divide en dos componentes hijos:

**app-detalle-procesados**:
- Muestra las facturas guardadas exitosamente
- Muestra los registros que no pudieron procesarse
- Permite copiar NITs para asignación de cuentas
- Recibe: registros, columnas, símbolo de moneda, tarjetas de totales

**app-asignacion-datos-sat**:
- Permite asignar cuentas contables a proveedores
- Se activa al seleccionar un NIT desde la tabla de resultados
- Recibe: NIT seleccionado
- Emite evento para volver a la vista de resultados

#### Navegación Condicional
```
Vista Principal (Carga y Resultados)
        ↓ (Al seleccionar NIT)
Vista de Asignación de Cuenta
        ↓ (Botón volver)
Vista Principal
```

### Integración con Servicios Backend

#### Endpoints Utilizados
1. **obtenerHojasExcel**: Extrae los nombres de las hojas del archivo Excel
2. **trasladarDatosComprasSAT**: Procesa y guarda las facturas del Excel
3. **getRegistrosFacturasFelCompras**: Obtiene las facturas guardadas

### Manejo de Errores
- Errores de carga de hojas: Modal con opción de reintentar
- Errores de traslado: Modal informativo
- Errores al obtener registros: Modal y desactivación de vista de resultados
- Validación de archivo duplicado: Mensaje de advertencia temporal

---

## DetalleProcesadosComponent

**Ubicación:** `detalle-procesados.component.ts`

**Descripción:**  
Componente de visualización que muestra los resultados del traslado de datos mediante un sistema de tabs. Presenta dos tablas: una con los registros guardados exitosamente y otra con los registros que no pudieron procesarse. Incluye funcionalidad para copiar NITs y tarjetas de totales con sumatorias de impuestos.

### Propósito Principal
Proporcionar al usuario una vista clara y organizada de los resultados del procesamiento, permitiendo identificar rápidamente:
- Facturas procesadas correctamente
- Registros con errores o faltantes
- Totales de impuestos calculados
- Acceso rápido a NITs para asignación de cuentas

### Inputs (Propiedades Recibidas del Padre)

| Input | Tipo | Descripción |
|-------|------|-------------|
| `registrosGuardados` | `PaFacturaFELComprasM[]` | Array de facturas FEL guardadas exitosamente en el sistema |
| `registrosFaltantes` | `PaFacturaFELComprasM[]` | Array de registros que no pudieron ser procesados |
| `columns` | `string[]` | Nombres de las columnas a mostrar en las tablas |
| `simboloMoneda` | `string \| null` | Símbolo de la moneda de la empresa (ej: "Q", "$") |
| `tarjetasTotales` | `{label: string; valor: number; color?: string}[]` | Array de objetos con totales de impuestos para las tarjetas |

### Outputs (Eventos Emitidos al Padre)

| Output | Tipo | Descripción |
|--------|------|-------------|
| `nitCopiadoEvent` | `EventEmitter<string>` | Emite el NIT cuando el usuario hace clic en "Copiar" |

### Propiedades Internas

#### Control de Tabs
```typescript
private _activeTab: 'registrosGuardados' | 'registrosFaltantes' = 'registrosFaltantes';
```

**Implementación con Getter/Setter:**
- **Valor por defecto**: `'registrosFaltantes'` (muestra primero los registros con error)
- **Propósito del setter**: Detectar cambios cuando el usuario cambia de tab
- **Tipo**: Union type con solo dos valores posibles

### Métodos

#### 1. ngOnInit()
**Propósito:** Inicialización del componente

**Acciones:**
- Obtiene el símbolo de moneda de la empresa actual
- Asigna valor por defecto si no existe símbolo

#### 2. copiarNIT(nit: string)
**Propósito:** Copia el NIT al portapapeles y notifica al componente padre

**Parámetros:**
- `nit`: Número de Identificación Tributaria a copiar

**Flujo de Ejecución:**
1. Valida que el NIT no esté vacío
2. Utiliza la API del navegador `navigator.clipboard.writeText()`
3. Si la copia es exitosa:
   - Registra el éxito en consola
   - **Emite el evento** `nitCopiadoEvent` al componente padre
4. Si hay error, lo registra en consola

### Estructura de la Interfaz

#### Sistema de Tabs

**Tab 1: Ver Faltantes** (registrosFaltantes)

**Tab 2: Ver Guardados** (registrosGuardados)

#### Contenido del Tab "Registros Guardados"

**Tabla Principal:**
- **Componente**: `app-tabla-paginada`
- **Datos**: Facturas guardadas exitosamente
- **Columnas**: Dinámicas (recibidas por Input)
- **Paginación**: 100 registros por página
- **Columna Especial**: Template personalizado para copiar NIT

**Tarjetas de Totales:**
- **Ubicación**: Parte inferior derecha de la tabla
- **Layout**: Grid responsivo (1-2-4 columnas según tamaño de pantalla)
- **Componente**: `app-total-card`
- **Props pasadas**: label, valor, color, símbolo de moneda
- **Condición**: Solo se muestran si hay tarjetas disponibles

**Estado Vacío (No hay registros guardados):**
- Icono de check circular grande
- Mensaje: "No hay registros ejecutados disponibles"
- Diseño centrado verticalmente

#### Contenido del Tab "Registros Faltantes"

**Tabla Principal:**
- **Componente**: `app-tabla-paginada`
- **Datos**: Registros que fallaron al procesarse
- **Columnas**: Las mismas que la tabla de guardados
- **Paginación**: 100 registros por página
- **Sin columnas especiales**: No incluye el botón de copiar NIT

**Estado Vacío (No hay registros faltantes):**
- Icono de exclamación circular grande
- Mensaje: "No hay registros faltantes disponibles"
- Diseño centrado verticalmente

### Flujo de Comunicación con el Padre
```
TrasladoDatosSatComponent (Padre)
        ↓ (Inputs)
DetalleProcesadosComponent
  - registrosGuardados
  - registrosFaltantes
  - columns
  - tarjetasTotales
  - simboloMoneda
        ↓ (Usuario hace clic en "Copiar NIT")
copiarNIT() ejecuta
        ↓ (Output)
nitCopiadoEvent.emit(nit)
        ↓
TrasladoDatosSatComponent.recibirNit(nit)
        ↓
Activa vista de AsignacionDatosSat
```
## AsignacionDatosSatComponent

**Ubicación:** `asignacion-datos-sat.component.ts`

**Descripción:**  
Componente especializado en la asignación de cuentas contables a facturas FEL (Factura Electrónica en Línea) por NIT. Permite buscar facturas mediante NIT, seleccionar una cuenta contable apropiada y actualizar la base de datos con la asignación realizada. 

### Propósito Principal
Completar el flujo de procesamiento de facturas FEL al permitir la asignación manual de cuentas contables a proveedores específicos, asegurando que cada factura quede correctamente categorizada en el sistema contable.

### Flujo de Operación
```
Usuario ingresa NIT → Busca facturas FEL → Sistema muestra facturas encontradas → 
Usuario selecciona facturas → Elige cuenta contable → Confirma asignación → 
Sistema actualiza BD y crea relación cuenta contable-cuenta corriente
```
### Propiedades
#### Datos Principales
| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `campoNit` | `string` | NIT ingresado por el usuario o recibido desde otro componente |
| `comprasFEL` | `any[] | null` | Lista de facturas FEL de compras obtenidas desde el API |
| `comprasFELKeys` | `string[]` | Nombres de las columnas para la tabla de facturas |
| `compra` | `PaFacturaFELComprasM | null` | Factura actualmente seleccionada |
| `registrosSeleccionados` | `any[]` | Registros marcados como seleccionados en la tabla |

#### Cuenta Contable y Cliente
| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `itemCuentaContableSeleccionada` | `PaBscNomenclaturaContable2M | null` | Cuenta contable seleccionada por el usuario |
| `clienteSeleccionado` | `any` | Cliente actualmente seleccionado desde el servicio CxC |
| `clientes` | `PaBscCuentaCorrentistaMovilM[]` | Lista de clientes obtenidos desde el servicio CxC |

#### Banderas de Estado y Carga
| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `cargandoBusquedaNit` | `boolean` | Indica si se está realizando búsqueda por NIT |
| `cargandoActualizacion` | `boolean` | Indica si se está actualizando la cuenta contable |
| `cargando` | `boolean` | Estado general de carga |
| `mostrarSeccionCuentaContable` | `boolean` | Controla la visibilidad del bloque de selección de cuenta contable |
| `mostrarAdvertencia` | `boolean` |Controla la visibilidad de advertencias o mensajes informativos|
| `noDatosEncontrados` | `boolean` | Indica si no se encontraron datos en la búsqueda |

### Referencias a Componentes Hijos
| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `@ViewChild(CuentaContableDetalleComponent)` | `CuentaContableDetalleComponent` | Referencia al componente hijo de detalle de cuentas contables |
| `@ViewChild('inputNit')` | `ElementRef` | Referencia al campo de entrada de NIT para control manual del foco |
| `@ViewChild(MensajeAdvertenciaComponent)` | `MensajeAdvertenciaComponent` | Referencia al componente que muestra mensajes de advertencia |

### Inputs y Suscripciones
| Propiedad | Tipo | Descripción |
|-----------|------|-------------|
| `@Input() nitSeleccionado` | `string | null` | NIT recibido como input desde otro componente (opcional) |
| `private subscription` | `Subscription | undefined` | Suscripción activa al observable de búsqueda (se limpia al hacer una nueva) |


### Métodos Principales

#### 1. clearInputNit(ref: HTMLInputElement)
**Propósito:** Limpia todos los datos relacionados al NIT y resetea la vista
**Acciones:**
- Reinicia campoNit a string vacío
- Establece foco en el input de referencia
- Limpia arrays de facturas FEL y registros seleccionados
- Oculta la sección de cuenta contable
- Reinicia la cuenta contable seleccionada en el DataService

#### 2. onNitChange(value: string)
**Propósito:**  Maneja los cambios en el campo de NIT y limpia datos si se deja vacío
**Comportamiento:**
- Actualiza campoNit con el valor ingresado
- Si el valor está vacío o solo contiene espacios:
  - Limpia arrays de facturas FEL
  - Limpia registros seleccionados
  - Oculta sección de cuenta contable
  - Reinicia cuenta contable seleccionada
  
#### 3. buscarCliente()
**Propósito:**  Busca clientes en el servicio CxC basándose en el NIT ingresado
**Flujo:**
- Limpia estados anteriores y desuscribe suscripciones previas
- Prepara parámetros: empresa, usuario, estación de trabajo, aplicación
- Llama a cxcService.buscarClientes()
- Al recibir respuesta:
  - Asigna lista de clientes
  - Si hay resultados, selecciona automáticamente el primer cliente  
  - Actualiza DataService con el cliente seleccionado

#### 4. buscarFacturasFELCompras()
**Propósito:**  Obtiene facturas FEL de compras desde el servicio según el NIT ingresado
##### **Validaciones:**
- Verifica que campoNit no esté vacío
##### **Flujo:**
- Prepara modelo con NIT, empresa y nombre de emisor
- Activa indicador de carga cargandoBusquedaNit
- Limpia datos anteriores
- Llama a comprasSATService.getFacturasFelCompras()
- Al recibir respuesta:
  - Mapea resultados agregando propiedad seleccionado: true
  - Extrae keys para columnas de tabla
  - Filtra registros seleccionados automáticamente
  - Ejecuta buscarCliente() para cargar datos del cliente
  - Actualiza estado de sección de cuenta contable
  
#### 5. actualizarCuentaFacturaFELCompras()
**Propósito:** Actualiza la cuenta contable asignada a las facturas FEL en el backend
**Flujo:**
- Ejecuta insCtaContableCtaCtista() para crear relación
- Prepara modelo con NIT, empresa, cuenta contable e ID de cuenta
- Activa indicador de carga cargandoActualizacion
- Llama a comprasSATService.postCuentaFacturaFELCompras()
- Al recibir respuesta:
  - Muestra mensaje de éxito con nombre de cuenta asignada
  - Cierra modal de confirmación
  - Vuelve a buscar facturas para actualizar vista
  
#### 6. insCtaContableCtaCtista()
**Propósito:** Inserta relación entre cuenta contable y cuenta corriente del cliente
**Flujo:**
- Prepara modelo con datos del cliente y cuenta contable seleccionada
- Llama a comprasSATService.insertarCtaContableCtaCtista()
- Maneja errores mostrando modal de error

#### 7. seleccionarPorNit(nit: string)
**Propósito:** Permite seleccionar o deseleccionar registros por NIT dentro de la tabla
**Lógica:**
- Verifica si todos los registros con ese NIT ya están seleccionados
- Alterna el estado de selección para todos los registros con el mismo NIT
- Deselecciona registros con NIT diferente
- Actualiza lista de registros seleccionados
- Reinicia cuenta contable seleccionada
- Actualiza estado de visibilidad de sección de cuenta contable

## Servicios
### ComprasSatService

**Descripción:**  
El servicio ComprasSatService centraliza toda la comunicación entre el módulo Facturas FEL Compras y las APIs del backend. Gestiona el envío, consulta y actualización de datos relacionados con las facturas FEL, utilizando peticiones HTTP seguras con autenticación JWT.
Incluye métodos para trasladar archivos Excel al sistema (trasladarDatosComprasSAT), consultar registros procesados (getFacturasFelCompras y getRegistrosFacturasFelCompras), y actualizar o insertar cuentas contables (postCuentaFacturaFELCompras e insertarCtaContableCtaCtista).
Además, maneja dinámicamente la URL base del API y aplica un control uniforme de errores, asegurando trazabilidad y estabilidad en el intercambio de información entre Angular y el backend.
