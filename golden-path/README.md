# Fase 2: Construcción del Golden Path

El objetivo es investigar la anatomía de los Software Templates de Backstage y construir tu propio Golden Path para crear microservicios NodeJS de forma automatizada.

## ¿Qué es un Golden Path?

En el contexto de Plataforma de Ingeniería, un **Golden Path** es la ruta pre-aprobada que la plataforma ofrece a los equipos de desarrollo para realizar una tarea de forma segura, eficiente y consistente.

Un desarrollador no necesita saber cómo configurar un repositorio, un pipeline de CI/CD, o un manifiesto de Kubernetes: simplemente llena un formulario y obtiene un servicio listo para producción.

En Backstage, los Golden Paths se implementan mediante **Software Templates**, definidas en archivos `template.yaml`.

---

## Investigación

Antes de escribir código, debes investigar la documentación oficial de Backstage. Esta sección contiene **preguntas guía** que debes responder como parte de esta actividad.

### Recursos clave para investigar

| Recurso | URL |
|---|---|
| Documentación de Software Templates | https://backstage.io/docs/features/software-templates/ |
| Referencia de `template.yaml` (fields) | https://backstage.io/docs/features/software-templates/writing-templates |
| Built-in Actions del Scaffolder | https://backstage.io/docs/features/software-templates/builtin-actions |
| Referencia de Nunjucks (motor de plantillas) | https://mozilla.github.io/nunjucks/templating.html |
| Publicar en GitHub con Backstage | https://backstage.io/docs/features/software-templates/builtin-actions#publish-github |

### 1. Anatomía de un `template.yaml`

Un `template.yaml` es un archivo YAML que Backstage interpreta para renderizar un formulario y orquestar acciones.

#### 1.1 Estructura general

Investiga y responde: ¿Cuáles son las **4 secciones principales** de un `template.yaml`? Describe brevemente el propósito de cada una.

```
TODO: Completa esta tabla con tus hallazgos

| Sección  | Propósito |
|apiVersion|Define la versión de la API de andamiaje de Backstage que se va a utilizar (por ejemplo, scaffolder.backstage.io/v1beta3).|
|   kind   |Especifica el tipo de entidad que se está declarando, que para este caso siempre debe ser Template.|
| metadata |Contiene la información de identificación de la plantilla, como su nombre único (name), título legible por humanos (title), descripción y etiquetas (tags).|
| spec     |Es el núcleo de la plantilla; define los parámetros del formulario que llenará el usuario (parameters) y la lista secuencial de pasos/acciones que se ejecutarán (steps).|
```

#### 1.2 Metadata del Template

Investiga los campos del bloque `metadata`. ¿Qué es el campo `annotations` y para qué sirve en el contexto de los templates?

```
Annotations sirve para agregar información adicional y configuraciones avanzadas a la plantilla mediante pares de clave-valor.  En las plantillas (Templates), sirve principalmente para:Vincular documentación.Integrar plugins.Organizar la interfaz.
```

### 2. Parámetros de Entrada (`parameters`)

La sección `parameters` define el formulario que verá el desarrollador. Es el contrato de tu API de Plataforma.

#### 2.1 Tipos de campos (`ui:widget`)

Backstage usa JSON Schema para definir los campos del formulario. Investiga y lista al menos **5 tipos de campos** disponibles y cuándo usarías cada uno.

```
TODO: Completa esta tabla

| Widget (`ui:widget`) | Tipo de dato | Caso de uso |
| text | string | nombre del servicio |
| textarea | string | descripcion del repositorio |
| select | string | elige el programa de programacion
| checkboxes | array |selecciona herramientas extras|
| password | string | ingresa token seguro  |
```

#### 2.2 Validaciones

¿Cómo puedes hacer que un campo de tipo `string` solo acepte valores en minúsculas sin espacios (útil para nombres de servicios)? Menciona las propiedades de JSON Schema relevantes.

```
pattern: define la expresión regular que el texto debe cumplir obligatoriamente. Para minúsculas sin espacios, se usa el Regex ^[a-z0-7-_]+$ (solo letras minúsculas, números, guiones y guiones bajos).
```

### 3. Pasos de Orquestación (`steps`)

La sección `steps` define la secuencia de acciones que el scaffolder ejecutará automáticamente.

#### 3.1 Actions disponibles

Investiga las **Built-in Actions** del scaffolder de Backstage. Completa la siguiente tabla con las acciones que necesitarás para tu Golden Path:

```
TODO: Completa esta tabla

| Action ID | ¿Qué hace? | Inputs principales |
| fetch:template | Descarga una plantilla base desde tu repo y renderiza variables | URL , targetPath, values |
| publish:github | crea un nuevo repo en github | allowedOwners, repoUrl, description |
| catalog:register | registra automaticamente el nuevo componente generadoen el catalogo del Backstage | cataologInfoUrl, Catalog-info.yaml |
```

#### 3.2 Motor de Plantillas (Nunjucks)

El paso `fetch:template` usa **Nunjucks** para reemplazar variables en los archivos del skeleton. ¿Cómo accedes al valor de un parámetro llamado `serviceName` dentro de un archivo del skeleton?

```
TODO: se utiliza doble llave ({{ ... }}) apuntando al objeto values.
```

#### 3.3 Output del paso `publish:github`

La acción `publish:github` retorna un output que puedes usar en pasos siguientes. ¿Qué propiedad del output contiene la URL del repositorio recién creado? ¿Cómo se referencia en el siguiente paso?

```
TODO: remoteUrl. Se utiliza ${{ steps['<ID_DEL_PASO>'].output.remoteUrl }}
```

### 4. Outputs del Template

La sección `output` del template permite mostrarle al usuario información al final del proceso (links, instrucciones, etc.).

¿Qué tipos de `links` puedes mostrar en el output? Lista al menos 2 ejemplos concretos para nuestro caso de uso (link al repo, link al componente en el catálogo).

```
TODO:

 1. Link externo al repositorio recién creado en GitHub
    title: Ver Repositorio en GitHub
    url: ${{ steps['publish'].output.remoteUrl }}
    icon: github

 2. Link interno al componente registrado en el Catálogo de Backstage
    title: Ver en el Catálogo de Backstage
    entityRef: ${{ steps['register'].output.entityRef }}
    icon: catalog
```

### 5. El Skeleton del Microservicio

El directorio `skeleton/` contiene la estructura base del proyecto que se copiará como punto de partida. Investiga:

#### 5.1 Parametrización del skeleton

El archivo `skeleton/catalog-info.yaml` también debe parametrizarse. Investiga la estructura del `catalog-info.yaml` de Backstage y responde: ¿Qué campos mínimos son obligatorios?

```
TODO: apiVersion, kind, metadata.name, spec.type, spec.lifecycle, spec.owner.
```

---

## Objetivo: Construir tu Golden Path

Con la investigación completa, tienes todo lo necesario para construir tu Software Template. Tu template debe cumplir los siguientes requisitos:

### Contrato de la API (Parámetros)

El formulario que verá el desarrollador debe pedir **exactamente 2 campos**:

| Campo | Tipo | Descripción |
|---|---|---|
| `serviceName` | `string` | Nombre del microservicio (minúsculas, sin espacios) |
| `owner` | `string` | Propietario del componente (user o group del catálogo) |

### Acciones Orquestadas (Steps)

El template debe ejecutar **exactamente 4 pasos** en este orden:

```
Paso 1: fetch:template
        └── Descarga el skeleton de este repositorio e inyecta los parámetros

Paso 2: publish:github
        └── Crea un nuevo repositorio en tu cuenta personal de GitHub
            y hace push del código generado

Paso 3: catalog:register
        └── Registra el nuevo componente en el Software Catalog de Backstage

(Opcional) Paso 4: Output
        └── Muestra al usuario el link al repo y al componente en el catálogo
```

---

## Cómo cargar tu template en Backstage

Una vez que tengas tu `template.yaml` completo y subido a GitHub:

1. En Backstage, ve a **Settings → Catalog → Add Existing Component**
2. Introduce la URL raw de tu `template.yaml` en GitHub, por ejemplo:
   ```
   https://github.com/tu-usuario/tu-repo/blob/main/golden-path/template.yaml
   ```
3. Backstage descargará el template y lo mostrará en la sección **Create** del portal.
4. Prueba tu Golden Path creando un nuevo microservicio de prueba.
