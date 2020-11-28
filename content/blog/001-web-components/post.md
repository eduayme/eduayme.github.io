---
title: "Introducción a los Web Components"
summary: "Conceptos principales de los Web Components con ejemplos básicos"
date: 2020-11-22
url: introduccion-web-components
feature_image: "/blog/001-web-components/img/web-components.png"
author:
categories:
    - Tech
tags:
    - Web Components
    - HTML
    - Javascript
---

## Qué son

Los Componentes Web son un paquete de diferentes tecnologías que te permiten crear elementos personalizados reutilizables (con funcionalidad encapsulada apartada del resto del código) y utilizarlos en las aplicaciones web.


## Para qué sirven

Principalmente apuntan a resolver los problemas de reutilización de código que existe en HTML (con sus estilos CSS y scripts Javascript asociados), pudiendo encapsular partes de código (elementos web) haciéndolos independientes unos de otros.


## Ejemplo básico

Vamos a crear un ejemplo básico de Web Component para un título que ponga Hello World. Los pasos a seguir son los siguientes:

1. Crear el fichero del componentes con extensión '.js'
2. Definir en ese fichero javascript la nueva clase extendiendo `HTMLElement` (interfaz que representa cualquier elemento HTML)
3. Especificar el comportamiento del componente dentro de las 4 funciones predefinidas (lo veremos en profundidad en el apartado *Conceptos*). En este caso se utilizará `connectedCallback()`.
4. Registrar el nuevo elemento personalizado que hemos creado fuera de la clase con `customElements.define()`. A esta función se le pasa el nombre del componente (siempre en kebab-case y con mínimo un guión '-' entre dos palabras) y el nombre de la clase.
5. Si se quiere adjuntar un Shadow DOM (lo veremos en profundidad en el apartado *Conceptos*) al elemento se le añade con `Element.attachShadow()`.
6. Importar y utilizar el elemento en el fichero HTML con la nueva etiqueta personalizada.


#### **`say-hello.js`**
```javascript
class SayHello extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `<h1>Hello world</h1>`;
  }
}

customElements.define('say-hello', SayHello);
```

#### **`main.html`**
```html
<say-hello></say-hello>

<script src="say-hello.js"></script>
```


## Conceptos

#### Custom Elements

Conjunto de APIs de Javascript que permiten definir nuevos elementos personalizados y su comportamiento.

Un registro de Custom Element está asociado a un objeto `Window`, en vez de a un objeto `Document`, ya que cada constructor de un Custom Element hereda de la interfaz `HTMLElement` y solo existe uno por objeto `Window`.

La definición de un nuevo elemento es el proceso de añadir un Custom Element al registro de elementos personalizados `CustomElementRegistry`. Para conseguirlo se hace con el método `define(nombre, constructor,`*`opciones`*`)`.

[Información de la interfaz CustomElementRegistry](https://html.spec.whatwg.org/multipage/custom-elements.html#custom-elements-api)


#### Shadow DOM

Conjunto de APIs de Javascript para encapsular un DOM "oculto" únicamente para un elemento (renderizado por separado del DOM principal). Utilizando esta característica se pueden mantener en privado todas las características del componente, con su estilo CSS y funcionalidad en el script asociado, sin miedo a colisiones en las otras partes del documento.

Cuando se usa un componente con Shadow DOM asociado únicamente se le puede referenciar como elemento completo, nunca nada interno del componente. En este aspecto un Shadow DOM se comporta como un `iframe`, donde el contenido interno está fuera del alcance del resto del documento.

Con esta definición clara podemos separamos entre dos conceptos: `Light DOM`, donde el contenido es accesible para todo el mundo, y `Shadow DOM`, donde el contenido es local del componente e inaccesible para el resto.

Para asignar un `Shadow DOM` a un componente se hace a través del método `attachShadow(mode:`*` open|closed`*`)`.

#### **`say-hello.js`**
```javascript
const shadowRoot = document.getElementById('say-hello-red').attachShadow({ mode: 'open' });
shadowRoot.innerHTML =
  `<style>
  h1 {
    color: red;
  }
  </style>
  <h1>Hello world</h1>`;
```

#### **`main.html`**
```html
<div id="say-hello-red"></div>
<h1>Hello World not affected</h1>

<script src="say-hello.js"></script>
```

[Más información del Shadow DOM](https://developers.google.com/web/fundamentals/web-components/shadowdom)

[Información de la interfaz ShadowRoot](https://dom.spec.whatwg.org/#shadowroot)


#### ES Modules

Muchas veces en el código importamos ficheros javascript. El problema que existe es que si ya se han cargado previamente no necesitamos perder tiempo en volverlos a cargar. Esto lo resuelve ES Module, un modulo estándard de javascript oficial (ha llevado casi 10 años de estandarización).

A la hora de cargar un archivo JS con la etiqueta `script`, a través de esta estándard, podemos añadirle el tag `type="module"` para que importe ese fichero esté encapsulado en esa ventana y únicamente se ejecute una vez.

```html
<!-- ... -->
<script type="module" src="say-hello.js"></script>
```

Otra función muy importante que añade la estandarización de ES Modules es la de importación de variables. Esto nos permite utilizar las mismas variables entre diferentes fichero. Vamos a ver un ejemplo donde lo que se importa será el propio Web Component, en un caso para meterlo dentro de otro componente y en otro caso para crear uno nuevo extendiendo uno que ya tenemos.

#### **`say-hello.js`**
```javascript
export class SayHello extends HTMLElement {
  connectedCallback() {
    this.innerHTML = `<h1>Hello world</h1>`;
  }
}

customElements.define('say-hello', SayHello);
```

Si queremos meterlo dentro de otro Web Component:

#### **`say-hello-section.js`**
```javascript
import './say-hello'

class SayHelloSection extends HTMLElement {
  constructor() {
    super()
    this.attachShadow({ mode: 'open' })
    const section = document.createElement('section')
    this.shadowRoot.appendChild(section)
    const myCoolDiv = document.createElement('say-hello')
    section.appendChild(myCoolDiv)
  }
}

customElements.define('say-hello-section', SayHelloSection);
```

Si queremos extenderlo al crear un nuevo Web Component:

#### **`say-hello-cooler.js`**
```javascript
import { SayHello } from './say-hello.js'

class SayHelloCooler extends SayHello {
  constructor() {
    super()
    const coolerStyle = document.createElement('style')
    coolerStyle.textContent = `
      h1 {
        background-color: black;
        color: white;
      }
    `

    this.shadowRoot.appendChild(coolerStyle)
  }
}

customElements.define('say-hello-cooler', SayHelloCooler);
```

[Cómo funciona ES Modules](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)


#### HTML templates & slots

El tag HTML `template` permite acabar con código dentro del HTML normal que no será renderizado immediatamente pero puede ser utilizado más tarde.

Este tag de plantilla lo podemos complementar con el tag HTML `slot`, dejando insertar tus propias especificaciones, creando árboles DOM por separado y presentándolos de forma conjunta.

```html
<template>
  <slot name='descripcion'></slot>
</template>
```

Vamos a ver un ejemplo con varios libros y modificar la vista dinámicamente para entender mejor la etiqueta:

#### **`books-info.js`**
```javascript
'use strict';

const books = [
  { title: 'The Great Gatsby', author: 'F. Scott Fitzgerald' },
  { title: 'A Farewell to Arms', author: 'Ernest Hemingway' },
  { title: 'Catch 22', author: 'Joseph Heller' }
];

function appendBooks(templateId) {
  const booksList = document.getElementById('books');
  const fragment = document.getElementById(templateId);

  // Clear out the content from the ul
  booksList.innerHTML = '';

  // Loop over the books and modify the given template
  books.forEach(book => {
    // Create an instance of the template content
    const instance = document.importNode(fragment.content, true);
    // Add relevant content to the template
    instance.querySelector('.title').innerHTML = book.title;
    instance.querySelector('.author').innerHTML = book.author;
    // Append the instance ot the DOM
    booksList.appendChild(instance);
  });  
}

document.getElementById('templates').addEventListener('change', (event) => appendBooks(event.target.value));

appendBooks('book-template');
```

#### **`main.html`**
```html
<template id="book-template">
  <li><span class="title"></span> &mdash; <span class="author"></span></li>
</template>

<template id="book-template-2">
  <li><span class="author"></span>'s classic novel <span class="title"></span></li>
</template>

<ul id="books"></ul>

<fieldset id="templates">
  <legend>Choose template</legend>

  <label>
    <input type="radio" name="template" value="book-template" checked> Template One
  </label>
  <label>
    <input type="radio" name="template" value="book-template-2"> Template Two
  </label>
</fieldset>

<script src="books-info.js"></script>
```


## Compatibilidad

#### Navegadores web

|                 | Firefox | Chrome | Safari | Opera | Edge |
| --------------- |:-------:| :----: | :----: | :---: | :--: |
| Custom Elements | Sí      | Sí     | Sí     | Sí    | v76 o + |
| Shadow DOM      | Sí      | Sí     | Sí     | Sí    | v75 o + |
| ES Modules      | Sí      | Sí     | Sí     | Sí    | Sí  |
| HTML templates  | Sí      | Sí     | Sí     | Sí    | Sí  |


#### Soporte a usuarios según sus navegadores web

|                 | Supported | Partial supported | Not supported |
| --------------- |:---------:| :---------------: | :-----------: |
| Custom Elements | 77.87%    | 16.59%            | 5.53%         |
| Shadow DOM      | 82.63%    | 11.93%            | 5.43%         |
| ES Modules      | 95.02%    | 2.95%             | 2.02%         |
| HTML templates  | 96.57%    | 0.17%             | 3.27%         |

*Datos de noviembre 2020 en la web [Can I use](https://caniuse.com/?search=components)*


### Polyfills

Los Web Components son unos estándares relativamente nuevos para los navegadores web. Para versiones más antiguas existen los Polyfills, que simulan las capabilities faltantes de la forma más realista posible.

[Información de Polyfills](https://www.webcomponents.org/polyfills/)


## Responsive

En el diseño de interfaces gráficas tenemos que asegurar que el diseño se verá bien en cualquier tipo de pantalla y dispositivo. Para los Web Components el problema reside en no poder controlar el estilo basándonos en el elemento padre que lo contiene. Actualmente solo se puede consultar el tamaño del viewport del usuario aunque las propuestas para solucionar esto se empezaron a presentar en 2013 con `Media Queries` o `Element Queries` y más tarde en 2015 con un borrador de propuesta de `Container Queries`.

[Inicio propuesta Media Queries](https://ianstormtaylor.com/media-queries-are-a-hack/)

[Borrador Container Queries](https://wicg.github.io/container-queries/)

Por el momento no existe ningún estándar en los navegadores web con estas posibles soluciones por lo que toca implementar a mano alguna idea para aproximarse a resolverlo.

Tras un tiempo de investigación la mejor solución que se ha podido encontrar por el momento es la de Philip Walton, actual ingeniero de Google, dónde se implementan clases genéricas CSS para los breakpoints y luego se añaden a cada contenedor de forma dinámica para que sus Web Components internos los puedan detectar. Para hacer esto se utiliza la interfaz `ResizeObserver`, actualmente compatible con todos los navegadores excepto IE y para Firefox en Android. La implementación sería a partir de los siguientes pasos:

1. Definir breakpoints para responsive

| Breakpoint name  | Container width    |
| ---------------- |:------------------:|
| SM               | `min-width: 24rem` |
| MD               | `min-width: 36rem` |
| LG               | `min-width: 48rem` |
| XL               | `min-width: 60rem` |


2. Definir breakpoints en el Web Component

```css
MyComponent {
  /* Estilo base para cualquier tamaño de pantalla */
}

.MD > .MyComponent {
  /* Sobrescribir estilo para tamaño mediano de pantalla */
}

.LG > .MyComponent {
  /* Sobrescribir estilo para tamaño grande de pantalla */
}
```


3. Aplicar `ResizeObserver` de forma dinámica

```javascript
/* Si ResizeObserver es soportado */
if ('ResizeObserver' in self) {
  /* Crear instancia ResizeObserver para el contenedor de elementos */
  var ro = new ResizeObserver(function(entries) {
    /* Crear breakpoints por defecto */
    var defaultBreakpoints = {SM: 384, MD: 576, LG: 768, XL: 960};

    entries.forEach(function(entry) {
      /* Si breakpoints están definidos en el elemento observado usar. */
      /* Sino los por defecto */
      var breakpoints = entry.target.dataset.breakpoints ?
          JSON.parse(entry.target.dataset.breakpoints) :
          defaultBreakpoints;

      /* Actualizar los breakpoints del elemento observado */
      Object.keys(breakpoints).forEach(function(breakpoint) {
        var minWidth = breakpoints[breakpoint];
        if (entry.contentRect.width >= minWidth) {
          entry.target.classList.add(breakpoint);
        } else {
          entry.target.classList.remove(breakpoint);
        }
      });
    });
  });

  /* Buscar todos los elementos con atributo `data-observe-resizes` */
  /* Si lo tienen entonces observarlos */
  var elements = document.querySelectorAll('[data-observe-resizes]');
  for (var element, i = 0; element = elements[i]; i++) {
    ro.observe(element);
  }
}
```

Con estos 3 pasos ya tendríamos una personalización para que los Web Components puedan detectar el tamaño del elemento padre. La transformación que se haría sería este:

*Elemento de 600px original*
```html
<div data-observe-resizes>
  <div class="MyComponent">...</div>
</div>
```

*Se convertiría en esto*
```html
<div class="SM MD" data-observe-resizes>
  <div class="MyComponent">...</div>
</div>
```

Este código de ejemplo solo funciona para elementos contenedor que ya están en el DOM. Para solucionar esto añadiremos un único ResizeObserver que observe a todos los componente de contenedor genérico que se creen. Por ejemplo, si el elemento genérico lo llamásemos `<responsive-container>` el código quedaría de esta manera:

```javascript
/* Crear un único observador para todos los elementos <responsive-container> */
const ro = new ResizeObserver(...);

class ResponsiveContainer extends HTMLElement {
  /* ... */
  connectedCallback() {
    ro.observe(this);
  }
}

self.customElements.define('responsive-container', ResponsiveContainer);
```

[Artículo Responsive Components](https://philipwalton.com/articles/responsive-components-a-solution-to-the-container-queries-problem/)

[Responsive Components demo](https://philipwalton.github.io/responsive-components/)

[Información interfaz ResizeObserver](https://www.w3.org/TR/resize-observer/)
