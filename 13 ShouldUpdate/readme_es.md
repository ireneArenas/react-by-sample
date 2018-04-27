# 13 ShouldUpdate

En este ejemplo vamos a mejorar el rendimiento de 'shouldComponentUpdate'.

Vamos a implementar un widget de satisfacción del cliente, basado en smiles, aceptará un valor de rango (0 a 500) y las caras tendrán un rango de valores 0..100,100..200,200..300,300..400,400..500. Solo activaremos la opción de renderizar siempre que el valor cambie al siguiente o anterior rango de valores.

Vamos a coger el punto de inicio el ejemplo 03 State:

Resumen:
- Eliminar los componentes _hello_ y _nameEdit_ 
- Copiar debajo del directorio _content_ los 4 png que contienen los smiles.
- Crear debajo del directorio _content_ un fichero site.css y definir los estilos de los smiles.
- Crear un componente smily.
- Añadir al estado una entrada currenValue, pasarlo al controls  y añadir un slider para configurarlo.
- Añadiremos una optimización ... compontentshouldupdate.

## Prerequisitos

Instalar [Node.js and npm](https://nodejs.org/en/) (v6.6.0 o más nuevo) si no está ya instalado en tu ordenador.

> Verificar que tienes al menos la versión de node v6.x.x and npm 3.x.x ejecutando `node -v` y `npm -v` en la terminal de windows. Versiones más antiguas puedes producir errores.

## Pasos para construirlo

- Copiar el contenido de  _03 State_ y ejecutar:

```
npm install
```

- Eliminar _

_./src/app.tsx_

```jsx
import * as React from 'react';

interface Props {
}

interface State {
}

export class App extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
  }

  public render() {
    return (
      <div>
      </div>
    );
  }
}
```

- Creamos una carpeta _src/content_ y copiamos las 5 caras de smile ( puedes copiarlas de la implementación de muestra en github).

- Vamos a crear un fichero css: _src/content/site.css_ y añadir los estilos a los smiles:
_./src/content/site.css_

```css
.very-dissatisfied {
  width:100%;
  height:80px;
  background:url('./one.png') no-repeat;;
}

.somewhat-dissatisfied {
  width:100%;
  height:80px;
  background:url('./two.png') no-repeat;
}

.neither {
  width:100%;
  height:80px;
  background:url('./three.png') no-repeat;
}

.somewhat-satisfied {
  width:100%;
  height:80px;
  background:url('./four.png') no-repeat;
}

.very-satisfied {
  width:100%;
  height:80px;
  background:url('./five.png') no-repeat;
}
```

- En _webpack.config.js vamos a añadir el nuevo fichero css como punto de entrada:

_webpack.config.js_

```diff
entry: [
  './main.tsx',
  '../node_modules/bootstrap/dist/css/bootstrap.css',
+  './content/site.css'
],
```
- Necesitamos añadir un cargador para manejar las imágenes en webpackconfig.js (añádelo si aún no está definido en la configuración del paquete) :

_webpack.config.js_

```javascript
{
  test: /\.(png|jpg)$/,
  exclude: /node_modules/,
  loader: 'url-loader?limit=10000'
},
```

- Vamos a crear un simple _faceComponent_ debajo de src, empezaremos añadiendo algo codificado en el fichero src/face.tsx:

_.src/face.tsx_

```jsx
import * as React from 'react';

export const FaceComponent = (props : {level : number}) => {
  return (
    <div className="somewhat-satisfied"/>
  );
}
```

- Vamos a hacer una prueba rápida en _app.tsx_

_./src/app.tsx_

```diff
import * as React from 'react';
+ import {FaceComponent} from './face';

interface Props {
}

interface State {
}

export class App extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
  }

  public render() {
    return (
      <div>
+        <FaceComponent level={100}/>
      </div>
    );
  }
}
```

- Vamos a verificar y ejecutar el ejemplo: verificar que está haciendo lo que se esperaba.

```
npm start
```

- Ahora es el momento de vincular la propiedas con las caras adecuadas. Crearemos una función de estilo en _face.tsx_

```diff
import * as React from 'react';

+ const setSatisfactionClass = (level : number) => {
+  if(level < 100) {
+        return "very-dissatisfied"
+  }
+
+  if(level < 200) {
+        return "somewhat-dissatisfied"
+  }
+
+  if(level < 300) {
+        return "neither"
+  }
+
+  if(level < 400) {
+        return "somewhat-satisfied"
+  }
+
+  return "very-satisfied"
+}


export const FaceComponent = (props : {level : number}) => {
  return (
-    <div className="somewhat-satisfied"/>
+    <div className={setSatisfactionClass(props.level)}/>
  );
}
```

- En _app.tsx_ vamos a añadir una variable de estado manteniendo el nivel de satisfacción en un slider para permitir que el usuario lo actualice.

```jsx
import * as React from 'react';
import {FaceComponent} from './face'

interface Props {
}

interface State {
  satisfactionLevel : number;
}

export class App extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);

    this.state = {satisfactionLevel: 300};
  }

  public render() {
    return (
      <div>
        <input type="range"
                min="0"
                max="500"
                value={this.state.satisfactionLevel}
                onChange={(event : any) => this.setState({satisfactionLevel :event.target.value} as State)}
        />
        <br/>
        <span>{this.state.satisfactionLevel}</span>
        <br/>
        <FaceComponent level={this.state.satisfactionLevel}/>
      </div>
    );
  }
}
```

- Ejecutamos el ejemplo:

```
npm start
```

- Vamos a añadir una optimización de renderizado, debemos renderizar solo cuando el nivel cambia justo de rango de satisfacción, necesitamos mover el componente al estado del componente:

```jsx
import * as React from 'react';

interface Props {
  level : number;
}

export class FaceComponent extends React.Component<Props, {}> {

  setSatisfactionClass(level : number) {
    if(level < 100) {
          return "very-dissatisfied"
    }

    if(level < 200) {
          return "somewhat-dissatisfied"
    }

    if(level < 300) {
          return "neither"
    }

    if(level < 400) {
          return "somewhat-satisfied"
    }

    return "very-satisfied"
  }

  shouldComponentUpdate(nextProps : Props, nextState)
  {
    const rangeChange = [100, 200, 300, 400];

    let index =  0;
    let isRangeChange = false;

    while(!isRangeChange && index < rangeChange.length) {
      isRangeChange = (this.props.level < rangeChange[index] && nextProps.level >= rangeChange[index])
                    ||
                      (this.props.level > rangeChange[index] && nextProps.level <= rangeChange[index])
      ;

      index++;
    }

      return isRangeChange;
  }

  render() {
    return (
      <div className={this.setSatisfactionClass(this.props.level)}/>
    );
  }
}
```

> Hay una forma más fácil de implementar el mismo algoritmo en la actualización del componente.

- Si nosotros hacemos un breakpoint en el método render de faceComponent podemos ver como el render solo se lanza cuando tu cambias de rango de satisfacción (ejemplo: de 99 a 100).

```
npm start
```






