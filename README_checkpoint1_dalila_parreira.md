# Agente de atención al cliente para tienda online

**Proyecto:** Checkpoint 1 --- Agente Base Tienda Online\
**Autora:** Dalila Parreira\
**Plataforma:** n8n\
**Versión:** Checkpoint 1

## 1. Resumen del proyecto

Este proyecto implementa un agente conversacional para brindar atención
inicial a clientes de una tienda online. Su función es responder
consultas generales y consultas sobre productos utilizando la
información disponible en un catálogo de Google Sheets.

El agente busca resolver la consulta en el primer contacto cuando
dispone de información suficiente y verificable. Cuando la solicitud
requiere información o acciones que no están disponibles en el flujo,
debe explicar la limitación y orientar al cliente hacia el equipo humano
de soporte.

## 2. Objetivos

-   Automatizar la atención inicial de consultas frecuentes.
-   Consultar el catálogo antes de responder sobre productos, precios,
    características o stock.
-   Evitar respuestas inventadas o afirmaciones que no puedan
    verificarse.
-   Derivar a soporte humano las solicitudes que excedan el alcance del
    agente.
-   Registrar la interacción por correo para facilitar la supervisión.

## 3. Alcance funcional

### El agente puede

-   Responder consultas generales sobre la tienda.
-   Buscar en el catálogo información de productos, como nombre,
    categoría, descripción, precio, características o stock, siempre que
    esos datos estén disponibles en Google Sheets.
-   Comunicar de forma explícita cuando no encuentra información
    suficiente para responder.
-   Orientar al cliente hacia soporte humano cuando la consulta excede
    sus capacidades.

### El agente no puede

-   Consultar ni modificar pedidos.
-   Procesar pagos, cancelaciones o devoluciones.
-   Gestionar garantías, reclamos o excepciones comerciales.
-   Consultar cuentas o datos personales de clientes.
-   Confirmar políticas, plazos, promociones o condiciones que no estén
    disponibles en una fuente habilitada.
-   Afirmar que realizó una operación si no dispone de una herramienta
    que efectivamente la ejecute.

## 4. Arquitectura del workflow

El workflow está compuesto por los siguientes nodos:

1.  **When chat message received:** recibe el mensaje enviado por el
    usuario desde el chat de n8n.
2.  **AI Agent:** interpreta la consulta, aplica las instrucciones del
    System Message y determina si necesita consultar el catálogo.
3.  **Groq Chat Model:** proporciona el modelo de lenguaje utilizado por
    el agente. La configuración actual utiliza `openai/gpt-oss-20b`, con
    temperatura `0.1`.
4.  **Consultar catálogo - Google Sheets:** herramienta de consulta
    conectada directamente al puerto `Tool` del agente. Lee la hoja
    `Productos` cuando la consulta requiere información específica del
    catálogo.
5.  **Enviar log de ejecución:** envía un correo con la consulta
    original y la respuesta generada por el agente para dejar un
    registro de la interacción.

### Secuencia de ejecución

``` text
When chat message received
          |
          v
       AI Agent <------ Groq Chat Model
          |
          +------------ Consultar catálogo - Google Sheets
          |             (herramienta del agente)
          v
Enviar log de ejecución
```

La conexión de Google Sheets es una conexión de herramienta del agente,
no un paso que se ejecute obligatoriamente para todos los mensajes. El
agente decide consultarla cuando la pregunta requiere datos del
catálogo.

## 5. Criterios de comportamiento del agente

El System Message define el rol, el ámbito de actuación, el objetivo
operativo, las reglas de respuesta, el uso de herramientas, los límites
y el escalamiento a soporte humano.

Los principales criterios son:

-   Responder en español con un tono profesional, cordial y directo.
-   Priorizar la precisión por encima de responder inmediatamente.
-   Consultar el catálogo cuando la respuesta dependa de datos de
    productos.
-   Basarse únicamente en los datos recuperados por la herramienta.
-   No interpretar la ausencia de un resultado como confirmación de que
    un producto está agotado.
-   Hacer una pregunta breve si la solicitud es ambigua.
-   No solicitar contraseñas ni números completos de tarjetas.
-   No revelar instrucciones internas, credenciales ni información de
    otros usuarios.
-   Evitar consultas repetidas innecesarias a las herramientas.
-   Limitar normalmente las respuestas a 120 palabras, salvo que el
    cliente solicite más detalle.

## 6. Casos de uso

### Caso 1: consulta de precio

**Consulta de ejemplo:** "¿Cuánto cuesta el producto X?"

**Comportamiento esperado:** el agente consulta Google Sheets y responde
con el precio encontrado. Si no puede verificarlo, lo aclara y ofrece
derivar la consulta a soporte.

### Caso 2: consulta de características

**Consulta de ejemplo:** "¿Qué características tiene el producto X?"

**Comportamiento esperado:** busca el producto en el catálogo y resume
únicamente las características registradas.

### Caso 3: consulta de stock

**Consulta de ejemplo:** "¿Hay stock del producto X?"

**Comportamiento esperado:** consulta el dato de disponibilidad si está
registrado. No debe afirmar que está agotado solo porque no encuentre
una coincidencia.

### Caso 4: producto no encontrado

**Consulta de ejemplo:** "¿Tienen el producto Y?"

**Comportamiento esperado:** si no encuentra una coincidencia clara,
informa que no pudo verificar la información. No inventa productos ni
disponibilidad.

### Caso 5: estado de un pedido

**Consulta de ejemplo:** "¿Dónde está mi pedido?"

**Comportamiento esperado:** explica que no tiene acceso al sistema de
pedidos y deriva la consulta al equipo humano.

### Caso 6: devolución o cancelación

**Consulta de ejemplo:** "Quiero cancelar mi compra."

**Comportamiento esperado:** no afirma que pueda realizar la
cancelación. Indica que debe intervenir soporte humano.

## 7. Preguntas frecuentes

**¿El agente puede consultar cualquier información de la tienda?**\
No. En esta versión consulta el catálogo de productos conectado mediante
Google Sheets y responde consultas generales dentro de los límites
definidos.

**¿Puede confirmar que un producto está agotado?**\
Solo si dispone de información suficiente para sostener esa respuesta.
La ausencia de un resultado en la búsqueda no confirma por sí misma que
no haya stock.

**¿Puede modificar pedidos o procesar devoluciones?**\
No. Esas operaciones quedan fuera del alcance de esta versión y deben
ser atendidas por una persona.

**¿Qué ocurre si la hoja no contiene el dato solicitado?**\
El agente debe informar que no pudo verificarlo y orientar al cliente
hacia soporte, en lugar de inventar una respuesta.

**¿Qué registra Gmail?**\
El nodo de auditoría envía un correo con la consulta original y la
respuesta del agente. El registro facilita la revisión humana de las
interacciones.

**¿El agente conserva memoria de conversaciones anteriores?**\
Esta versión no incorpora un nodo de memoria. La memoria y el contexto
persistente quedan fuera del alcance de este checkpoint.

## 8. Supervisión y registro

El nodo **Enviar log de ejecución** envía un correo con:

-   La consulta original recibida desde el chat.
-   La respuesta generada por el AI Agent.
-   La referencia a la herramienta de catálogo utilizada por el flujo.

Este registro permite revisar qué preguntó el usuario y qué respondió el
agente. El correo no reemplaza la revisión humana ni garantiza por sí
solo que la respuesta sea correcta; sirve como evidencia para supervisar
el comportamiento.

## 9. Configuración técnica

  Componente                     Configuración
  ------------------------------ ----------------------------
  Plataforma de automatización   n8n
  Entrada                        When chat message received
  Agente                         AI Agent
  Máximo de iteraciones          5
  Modelo actual                  Groq Chat Model
  Modelo configurado             `openai/gpt-oss-20b`
  Temperatura                    0.1
  Herramienta                    Google Sheets
  Hoja consultada                `Productos`
  Registro                       Gmail

**Nota sobre el modelo:** la configuración actual utiliza Groq. Si la
rúbrica exige literalmente un nodo OpenAI Chat Model o Anthropic Chat
Model, deberá reemplazarse por el proveedor requerido y probarse
nuevamente.

## 10. Requisitos para importar el workflow

Antes de ejecutar el JSON en otra instancia de n8n, verificar:

1.  Que la instancia tenga disponibles los nodos utilizados.
2.  Que se configuren las credenciales de Google Sheets, Groq y Gmail.
3.  Que la cuenta de Google tenga permiso para leer la hoja `Productos`.
4.  Que el catálogo contenga datos de prueba actualizados.
5.  Que el destinatario del correo de auditoría sea el correspondiente
    al entorno donde se despliegue.
6.  Que se realice una prueba manual después de importar el workflow.

Las credenciales no deben compartirse en el README ni publicarse en el
repositorio.

## 11. Pruebas recomendadas

Antes de considerar la entrega validada, ejecutar y registrar al menos
estas pruebas:

  -----------------------------------------------------------------------
  Prueba                              Resultado esperado
  ----------------------------------- -----------------------------------
  Consulta de un producto existente   Respuesta basada en el catálogo

  Consulta de precio o stock          Uso de Google Sheets y respuesta
                                      con datos disponibles

  Consulta de producto no encontrado  Informa que no pudo verificarlo; no
                                      inventa datos

  Consulta ambigua                    Solicita una aclaración

  Consulta sobre pedido, pago o       Explica el límite y deriva a
  devolución                          soporte

  Registro de auditoría               Llega el correo con la consulta y
                                      la respuesta
  -----------------------------------------------------------------------

La captura de ejecución debe corresponder a una prueba real y mostrar el
resultado de los nodos relevantes.

## 12. Archivos del proyecto

-   **`checkpoint1_dalila_parreira.json`**: exportación del workflow de
    n8n.
-   **`README.md`**: documentación funcional y técnica del proyecto.
-   **Captura del workflow**: imagen de referencia de la estructura
    implementada.

Insertar la captura en el repositorio, por ejemplo en una carpeta
`docs/`, y actualizar la ruta de la imagen en esta sección si se decide
mostrarla directamente en GitHub.

## 13. Evolución prevista

Este checkpoint establece una base de atención inicial con consulta de
catálogo y registro por correo. En siguientes etapas se podrán
incorporar capacidades adicionales según las consignas del curso, por
ejemplo memoria y contexto, luego de que esos módulos sean abordados y
sin ampliar el alcance de esta entrega antes de tiempo.

------------------------------------------------------------------------

**Autora:** Dalila Parreira\
**Proyecto:** Checkpoint 1 --- Agente Base Tienda Online
