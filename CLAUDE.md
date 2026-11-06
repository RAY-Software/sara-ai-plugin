# Sara AI plugin

## Reglas de UI

Valen para todo lo que tenga pantalla.

- Títulos, subtítulos y headers de columna de tablas nunca se wrapean en 2 líneas. Sin palabras huérfanas en títulos ni en copy de marketing.
- Nunca usar el guion largo (U+2014).
- Todo control on/off binario va con estilo iPhone (track redondeado y knob que desliza), nunca pills Sí/No ni checkboxes: un solo componente compartido por repo; si todavía no existe, crearlo la primera vez que haga falta. Los selectores de 3 o más estados siguen siendo pills o select.
- Todo modal, diálogo y drawer se cierra con la tecla Escape, en cualquier momento, también con el foco dentro de un campo de texto y sin pedir confirmación. Usar un hook compartido por repo; si todavía no existe, crearlo la primera vez que haga falta, nunca un `keydown` suelto en cada modal. Con capas apiladas (modal sobre modal, o un menú abierto dentro de un modal), Escape cierra solo la de arriba. Un modal nuevo lleva Escape desde el principio.
- Única excepción, decidida por Franco: si cerrar perdería trabajo que no se recupera solo (cambios sin guardar en una página o formulario largo, una subida en curso), Escape abre una confirmación en vez de cerrar. Con la confirmación abierta, Escape la cierra y deja el modal como estaba. No ampliar esta excepción: ante la duda, Escape cierra.
- Un cambio de UI se revisa mirándolo renderizado, en desktop y en mobile, no solo en el diff: jerarquía y espaciado, consistencia con los componentes existentes, títulos sin wrap ni huérfanas, y estados vacío, de carga y de error.
