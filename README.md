# Práctica XML y DTD

**Estudiante:** Jose Lam (Charlie Lam)  
**Expediente:** 225213509  
**Correo:** a225213509@unison.mx  
**Repositorio:** [xml-dtd-practica](https://github.com/a225213509/xml-dtd-practica)

---

## Objetivo

Diseñar documentos XML bien formados a partir de información no estructurada, definir esquemas de validación mediante DTD (internos y externos), aplicar cardinalidades y restricciones sobre atributos, realizar pruebas de validación positivas y negativas, y gestionar el ciclo de desarrollo de artefactos de forma incremental utilizando Git.

---

## Ejercicio 1: Pedido

### Modelo propuesto

A partir del texto no estructurado del pedido, se identificaron y desglosaron los elementos clave:

| Información | Valor identificado | Elemento / Atributo propuesto |
|---|---|---|
| Identificador del pedido | `PED-2026-001` | Atributo `codigo` en `<pedido>` |
| Destinatario | Juan Pérez López | Elemento `<destinatario>` / `<nombre>` |
| Artículo | Portátil Gamer Modelo X | Elemento `<articulo>` con atributo `codigo="ART-404"` |
| Cantidad | 1 | Elemento `<cantidad>` |
| Precio | 18500.00 | Elemento `<precio moneda="MXN">` |
| Dirección | Calle Mayor 14, 3º B | Elemento `<direccion>` descompuesto |
| Fecha de entrega | 2026-11-15 | Elemento `<fecha_entrega>` |

Estructura XML final implementada en `ejercicio1/pedido.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<pedido codigo="PED-2026-001">
    <destinatario>
        <nombre>Juan Pérez López</nombre>
    </destinatario>
    <articulo codigo="ART-404">
        <descripcion>Portátil Gamer Modelo X</descripcion>
        <cantidad>1</cantidad>
        <precio moneda="MXN">18500.00</precio>
    </articulo>
    <direccion>
        <calle>Calle Mayor</calle>
        <numero>14</numero>
        <piso>3</piso>
        <letra>B</letra>
    </direccion>
    <fecha_entrega>2026-11-15</fecha_entrega>
</pedido>
```

### Decisiones de diseño

1. **¿Conviene almacenar la dirección como un único texto?**  
   No. Almacenar la dirección como una única cadena impide búsquedas granulares, dificulta la validación de campos obligatorios (como el número de portal) e impide la interoperabilidad automática con servicios de paquetería o bases de datos geográficas.
2. **¿Qué ventajas tendría separar sus componentes?**  
   Permite ordenar y filtrar por calle, normalizar códigos postales, validar tipos de datos numéricos y mapear directamente cada campo a columnas de una base de datos relacional.
3. **¿Cómo debería almacenarse una fecha para facilitar su procesamiento?**  
   Debe almacenarse bajo el estándar internacional **ISO 8601** (`AAAA-MM-DD`, por ejemplo `2026-11-15`). Esto facilita la ordenación cronológica natural (lexicográfica), evita confusiones entre formatos regionales (día/mes vs mes/día) y es directamente compatible con motores de bases de datos y parsers.
4. **¿Qué información podría ser atributo y cuál elemento?**  
   - **Elementos:** Contenidos semánticos principales y aquellos que requieren jerarquía o descomposición interna (`destinatario`, `direccion`, `articulo`).
   - **Atributos:** Metadatos o identificadores atómicos que califican a un elemento (`codigo="PED-2026-001"`, `codigo="ART-404"`, `moneda="MXN"`).

---

## Ejercicio 2: Nota

### DTD externo

Se definió la gramática en el archivo independiente `ejercicio2/nota.dtd`:

```dtd
<!ELEMENT nota (para, de, titulo, contenido)>
<!ELEMENT para (#PCDATA)>
<!ELEMENT de (#PCDATA)>
<!ELEMENT titulo (#PCDATA)>
<!ELEMENT contenido (#PCDATA)>
```

El documento `ejercicio2/nota.xml` asocia este DTD mediante la declaración `SYSTEM`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE nota SYSTEM "nota.dtd">
<nota>
    <para>Javier</para>
    <de>María</de>
    <titulo>Recordatorio</titulo>
    <contenido>No te olvides de la reunión mañana a las 10:00.</contenido>
</nota>
```

**Análisis de estructura:**
1. **Elemento raíz:** `<nota>`.
2. **Ocurrencia de `para`:** Exactamente una vez (secuencia obligatoria sin modificador de repetición).
3. **Orden de elementos:** Es significativo y estricto; el DTD exige el orden exacto `(para, de, titulo, contenido)`.
4. **Tipo de contenido:** `<nota>` contiene únicamente elementos hijos, mientras que `para`, `de`, `titulo` y `contenido` contienen únicamente texto analizable (`#PCDATA`).

### DTD interno

En `ejercicio2/nota-interno.xml` se integró la gramática directamente dentro del prólogo del archivo XML:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE nota [
    <!ELEMENT nota (para, de, titulo, contenido)>
    <!ELEMENT para (#PCDATA)>
    <!ELEMENT de (#PCDATA)>
    <!ELEMENT titulo (#PCDATA)>
    <!ELEMENT contenido (#PCDATA)>
]>
<nota>
    <para>Javier</para>
    <de>María</de>
    <titulo>Recordatorio</titulo>
    <contenido>No te olvides de la reunión mañana a las 10:00.</contenido>
</nota>
```

**Comparación entre DTD interno y externo:**

| Característica | DTD interno | DTD externo |
|---|---|---|
| Ubicación | Dentro del propio documento XML (`<!DOCTYPE ... [ ]>`) | En un archivo `.dtd` independiente |
| Reutilizable entre XML | No (debe duplicarse en cada documento) | Sí (múltiples XML apuntan al mismo DTD) |
| Archivo adicional | No requiere archivos adicionales | Requiere archivo `.dtd` externo |
| Conveniente para un único documento | Sí, es autocontenido y autónomo | Innecesario para documentos aislados |
| Conveniente para muchos documentos | No, complica el mantenimiento y la consistencia | Sí, centraliza el esquema para todo el sistema |

### Pruebas realizadas

Se realizaron pruebas negativas sobre `ejercicio2/nota.xml`:

| Modificación probada | ¿Bien formado? | ¿Válido? | Explicación técnica |
|---|---|---|---|
| Sustituir `<para>` por `<destinatario>` | Sí | No | El elemento `destinatario` no está declarado en el DTD y se omite el elemento obligatorio `para`. |
| Intercambiar el orden de `<para>` y `<de>` | Sí | No | El DTD impone una secuencia estricta `(para, de, ...)`. El parser rechaza `<de>` en la primera posición. |
| Agregar `<telefono>5551234567</telefono>` | Sí | No | El elemento `telefono` no está permitido en el modelo de contenido de `nota`. |

Las pruebas se verificaron visualmente mediante `git diff` y se restauró el estado original con `git restore ejercicio2/nota.xml`.

---

## Ejercicio 3: Matrícula

### Modelo

Estructura jerárquica:
```text
matricula
├── personal
│   ├── dni
│   ├── nombre
│   ├── titulacion
│   ├── curso_academico
│   └── domicilios
│       └── domicilio+
└── pago
    └── tipo_matricula
```

### Cardinalidad

El requerimiento exige que exista **al menos un domicilio**.
- Operador `?`: cero o uno.
- Operador `*`: cero o más.
- Operador `+`: **uno o más** (al menos uno).

Se aplicó la regla:
```dtd
<!ELEMENT domicilios (domicilio+)>
```

**Prueba sin domicilios:**
- **¿XML bien formado?** Sí (la sintaxis de etiquetas es correcta).
- **¿XML válido?** No.
- **¿Por qué?** El DTD exige al menos una ocurrencia (`+`) de `<domicilio>` dentro de `<domicilios>`.

### Restricción del atributo tipo

El atributo `tipo` del elemento `domicilio` debe ser obligatorio y restringido a los valores `familiar` o `habitual`:
```dtd
<!ATTLIST domicilio tipo (familiar|habitual) #REQUIRED>
```

**Evaluación de casos:**

| Caso | Predicción | Resultado | Explicación |
|---|---|---|---|
| `tipo="familiar"` | Válido | Válido | Coincide con una de las opciones válidas de la enumeración `(familiar\|habitual)`. |
| `tipo="habitual"` | Válido | Válido | Coincide con la otra opción válida permitida en la enumeración. |
| `tipo="temporal"` | Inválido | Inválido | El valor `"temporal"` no está declarado en la enumeración permitida. |
| Sin atributo `tipo` | Inválido | Inválido | La cláusula `#REQUIRED` hace estrictamente obligatoria la presencia del atributo. |

### DTD externo

Archivo `ejercicio3/matricula.dtd`:
```dtd
<!ELEMENT matricula (personal, pago)>
<!ELEMENT personal (dni, nombre, titulacion, curso_academico, domicilios)>
<!ELEMENT dni (#PCDATA)>
<!ELEMENT nombre (#PCDATA)>
<!ELEMENT titulacion (#PCDATA)>
<!ELEMENT curso_academico (#PCDATA)>
<!ELEMENT domicilios (domicilio+)>
<!ELEMENT domicilio (#PCDATA)>
<!ATTLIST domicilio tipo (familiar|habitual) #REQUIRED>
<!ELEMENT pago (tipo_matricula)>
<!ELEMENT tipo_matricula (#PCDATA)>
```

Archivo `ejercicio3/matricula.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE matricula SYSTEM "matricula.dtd">
<matricula>
    <personal>
        <dni>12345678Z</dni>
        <nombre>Jose Lam</nombre>
        <titulacion>Licenciatura en Ciencias de la Computación</titulacion>
        <curso_academico>2026-2027</curso_academico>
        <domicilios>
            <domicilio tipo="habitual">Calle Rosales 123, Hermosillo, Sonora</domicilio>
            <domicilio tipo="familiar">Av. Universidad 456, Hermosillo, Sonora</domicilio>
        </domicilios>
    </personal>
    <pago>
        <tipo_matricula>Ordinaria</tipo_matricula>
    </pago>
</matricula>
```

### DTD interno

Se implementó en una rama aislada de Git (`dtd-interno-matricula`) y posteriormente se integró a `main` mediante `git merge`.  
El archivo `ejercicio3/matricula-interno.xml` define el esquema completo en el prólogo sin dependencias externas.

### Pruebas realizadas

Se validó el documento `matricula.xml` contra `matricula.dtd` y `matricula-interno.xml` contra su DTD interno empleando el parser validador estándar de Java (`javax.xml.parsers.DocumentBuilderFactory.setValidating(true)`), comprobando:
1. Validación correcta con uno y múltiples domicilios.
2. Rechazo ante omisión de domicilios.
3. Validación de atributos enumerados (`familiar`, `habitual`).
4. Rechazo ante atributos no contemplados o faltantes.

---

## Conclusiones

### Respuestas al cuestionario final

1. **¿Cuál es la diferencia entre XML bien formado y XML válido?**  
   - Un **XML bien formado** es aquel que cumple estrictamente las reglas sintácticas fundamentales de XML (único elemento raíz, etiquetas debidamente cerradas, anidamiento jerárquico correcto, sensibilidad a mayúsculas/minúsculas y atributos entrecomillados).
   - Un **XML válido** es aquel que, además de estar bien formado, se ajusta con exactitud a una gramática o esquema formal predefinido (como un DTD o XSD), respetando el orden, nombres de elementos, cardinalidades y restricciones de atributos.

2. **¿Qué función cumple un DTD?**  
   Define la gramática, estructura y reglas de contenido que deben seguir los documentos XML asociados, permitiendo garantizar la uniformidad, consistencia y validación automática de los datos antes de su procesamiento por aplicaciones o sistemas de persistencia.

3. **¿Qué diferencia existe entre DTD interno y externo?**  
   - El **DTD interno** se escribe dentro del propio archivo XML en el bloque `<!DOCTYPE root [ ... ]>`, siendo ideal para documentos únicos y portátiles.
   - El **DTD externo** reside en un archivo independiente con extensión `.dtd` referenciado mediante `SYSTEM` o `PUBLIC`, permitiendo que múltiples documentos compartan el mismo esquema y facilitando el mantenimiento centralizado.

4. **¿Cómo se expresa cardinalidad en DTD?**  
   Se emplean tres operadores postfijos aplicados a los nombres de elementos o grupos:
   - `?`: Cero o una aparición (opcional).
   - `*`: Cero o más apariciones (opcional y repetible).
   - `+`: Una o más apariciones (obligatorio al menos uno y repetible).
   - Sin operador: Exactamente una aparición (obligatorio y no repetible).

5. **¿Cómo puede restringirse un atributo a determinados valores?**  
   Mediante una declaración `<!ATTLIST ...>` utilizando una **enumeración de valores** separados por barras verticales `|` dentro de paréntesis, seguida del modificador de obligatoriedad (`#REQUIRED`, `#IMPLIED` o un valor por defecto). Por ejemplo: `<!ATTLIST elemento atributo (opcion1|opcion2) #REQUIRED>`.

6. **¿Qué ventaja proporcionó Git durante las pruebas?**  
   Permitió un control estricto de versiones con trazabilidad granular, facilitando realizar pruebas experimentales y destructivas sin riesgo de corrupción del código base, asegurando que cada hito de desarrollo quedara respaldado de manera reproducible.

7. **¿Qué utilidad tuvieron `git diff` y `git restore`?**  
   - `git diff` permitió inspeccionar con exactitud las modificaciones introducidas en los archivos para las pruebas negativas antes de guardarlas.
   - `git restore` permitió revertir de manera instantánea y limpia las modificaciones temporales introducidas durante los ensayos, devolviendo los documentos a su estado válido sin margen de error humano.

8. **¿Qué ventaja proporcionó una rama para desarrollar una solución alternativa?**  
   La creación de la rama `dtd-interno-matricula` permitió desarrollar y probar la variante de DTD interno de forma aislada, sin alterar la rama principal de producción (`main`). Una vez confirmada la validez de la solución, la integración con `git merge` garantizó un historial limpio, trazable y sin conflictos.

---

## Estructura Final del Proyecto

```text
xml-dtd-practica/
├── README.md
├── ejercicio1/
│   └── pedido.xml
├── ejercicio2/
│   ├── nota.xml
│   ├── nota.dtd
│   └── nota-interno.xml
└── ejercicio3/
    ├── matricula.xml
    ├── matricula.dtd
    └── matricula-interno.xml
```
