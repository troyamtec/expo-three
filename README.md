<h1 align="center">Welcome to Expo & Three.JS 👋</h1>

<p>
  <a href="https://github.com/expo/expo-three/actions">
     <img alt="GitHub Actions status" src="https://github.com/expo/expo-three/workflows/Check%20Universal%20Module/badge.svg">
  </a>
  <a href="https://twitter.com/expo" aria-label="follow @expo on Twitter">
    <img alt="Twitter: Expo" src="https://img.shields.io/twitter/follow/expo.svg?style=social" target="_blank" />
  </a>
  <a href="https://github.com/expo/expo-three/blob/master/LICENSE">
    <img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-yellow.svg" target="_blank" />
  </a>
</p>

> Tools for using three.js to build native 3D experiences 💙

AR was moved to `expo-three-ar` in `expo-three@5.0.0`

### Installation

[![NPM](https://nodei.co/npm/expo-three.png)](https://nodei.co/npm/expo-three/)

> In `expo-three@5.0.0` Three.js is a **peer dependency**

```bash
yarn add three expo-three
```

### Usage

Import the library into your JavaScript file:

```js
import ExpoTHREE from 'expo-three';
```

Get a global instance of `three.js` from `expo-three`:

```js
import { THREE } from 'expo-three';
```

Do to some issues with the **Metro bundler** you may need to manually define the global instance of Three.js. This is important because three.js doesn't fully use ECMAScript but rather mutates a single global instance of `THREE` with side-effects.

```js
global.THREE = global.THREE || THREE;
```

## Creating a Renderer

### `ExpoTHREE.Renderer({ gl: WebGLRenderingContext, width: number, height: number, pixelRatio: number, ...extras })`

Given a `gl` from an
[`Expo.GLView`](https://docs.expo.io/versions/latest/sdk/gl-view.html), return a
[`THREE.WebGLRenderer`](https://threejs.org/docs/#api/renderers/WebGLRenderer)
that draws into it.

```js
const renderer = new ExpoTHREE.Renderer(props);
or;
/// A legacy alias for the extended renderer
const renderer = ExpoTHREE.createRenderer(props);
// Now just code some three.js stuff and add it to this! :D
```

### `ExpoTHREE.loadAsync()`

A function that will asynchronously load files based on their extension.

> **Notice**: Remember to update your `app.json` to bundle obscure file types!

`app.json`

```diff
{
  "expo": {
    "packagerOpts": {
+      "config": "metro.config.js"
    },
  }
}
```

`metro.config.js`

```js
module.exports = {
  resolver: {
    assetExts: ['db', 'mp3', 'ttf', 'obj', 'png', 'jpg'],
  },
};
```

#### Props

| Property      |           Type            | Description                                                      |
| ------------- | :-----------------------: | ---------------------------------------------------------------- |
| resource      |       PossibleAsset       | The asset that will be parsed asynchornously                     |
| onProgress    |       (xhr) => void       | A function that is called with an xhr event                      |
| assetProvider | () => Promise<Expo.Asset> | A function that is called whenever an unknown asset is requested |

##### PossibleAsset Format

export type PossibleAsset = Expo.Asset | number | string | AssetFormat;

```js
type PossibleAsset = number | string | Expo.Asset;
```

- `number`: Static file reference `require('./model.*')`
- `Expo.Asset`: [Expo.Asset](https://docs.expo.io/versions/latest/sdk/asset.html)
- `string`: A uri path to an asset

#### Returns

This returns many different things, based on the input file.
For a more predictable return value you should use one of the more specific model loaders.

#### Example

```js
const texture = await ExpoTHREE.loadAsync(
  'https://www.google.com/images/branding/googlelogo/2x/googlelogo_color_272x92dp.png',
);
```

## Loaders

### loadAsync(assetReference, onProgress, onAssetRequested)

A universal loader that can be used to load images, models, scenes, and animations.
Optionally more specific loaders are provided with less complexity.

```js
// A THREE.Texture from a static resource.
const texture = await ExpoTHREE.loadAsync(require('./icon.png'));
const obj = await ExpoTHREE.loadAsync(
  [require('./cartman.obj'), require('./cartman.mtl')],
  null,
  imageName => resources[imageName],
);
const { scene } = await ExpoTHREE.loadAsync(
  resources['./kenny.dae'],
  onProgress,
  resources,
);
```

### loadObjAsync({ asset, mtlAsset, materials, onAssetRequested, onMtlAssetRequested })

#### Props

- `asset`: a `obj` model reference that will be evaluated using `AssetUtils.uriAsync`
- `mtlAsset`: an optional prop that will be loaded using `loadMtlAsync()`
- `onAssetRequested`: A callback that is used to evaluate urls found within the `asset` and optionally the `mtlAsset`. You can also just pass in a dictionary of key values if you know the assets required ahead of time.
- `materials`: Optionally you can provide an array of materials returned from `loadMtlAsync()`
- `onMtlAssetRequested`: If provided this will be used to request assets in `loadMtlAsync()`

This function is used as a more direct method to loading a `.obj` model.
You should use this function to debug when your model has a corrupted format.

```js
const mesh = await loadObjAsync({ asset: 'https://www.members.com/chef.obj' });
```

### loadTextureAsync({ asset })

#### Props

- `asset`: an `Expo.Asset` that could be evaluated using `AssetUtils.resolveAsync` if `localUri` is missing or the asset hasn't been downloaded yet.

This function is used as a more direct method to loading an image into a texture.
You should use this function to debug when your image is using an odd extension like `.bmp`.

```js
const texture = await loadTextureAsync({ asset: require('./image.png') });
```

### loadMtlAsync({ asset, onAssetRequested })

#### Props

- `asset`: a `mtl` material reference that will be evaluated using `AssetUtils.uriAsync`
- `onAssetRequested`: A callback that is used to evaluate urls found within the `asset`, optionally you can just pass in a dictionary of key values if you know the assets required ahead of time.

```js
const materials = await loadMtlAsync({
  asset: require('chef.mtl'),
  onAssetRequested: modelAssets,
});
```

### loadDaeAsync({ asset, onAssetRequested, onProgress })

#### Props

- `asset`: a reference to a `dae` scene that will be evaluated using `AssetUtils.uriAsync`
- `onAssetRequested`: A callback that is used to evaluate urls found within the `asset`, optionally you can just pass in a dictionary of key values if you know the assets required ahead of time.
- `onProgress`: An experimental callback used to track loading progress.

```js
const { scene } = await loadDaeAsync({
  asset: require('chef.dae'),
  onAssetRequested: modelAssets,
  onProgress: () => {},
});
```

---

## `ExpoTHREE.utils`

### `ExpoTHREE.utils.alignMesh()`

#### Props

```js
type Axis = {
  x?: number,
  y?: number,
  z?: number,
};
```

| Property |    Type     | Description                       |
| -------- | :---------: | --------------------------------- |
| mesh     | &THREE.Mesh | The mesh that will be manipulated |
| axis     |    ?Axis    | Set the relative center axis      |

#### Example

```js
ExpoTHREE.utils.alignMesh(mesh, { x: 0.0, y: 0.5 });
```

---

### `ExpoTHREE.utils.scaleLongestSideToSize()`

#### Props

| Property |    Type     | Description                                                  |
| -------- | :---------: | ------------------------------------------------------------ |
| mesh     | &THREE.Mesh | The mesh that will be manipulated                            |
| size     |   number    | The size that the longest side of the mesh will be scaled to |

#### Example

```js
ExpoTHREE.utils.scaleLongestSideToSize(mesh, 3.2);
```

---

### `ExpoTHREE.utils.computeMeshNormals()`

Used for smoothing imported geometry, specifically when imported from `.obj` models.

#### Props

| Property |    Type     | Description                                       |
| -------- | :---------: | ------------------------------------------------- |
| mesh     | &THREE.Mesh | The mutable (inout) mesh that will be manipulated |

#### Example

````js
ExpoTHREE.utils.computeMeshNormals(mesh);
```

---

## THREE Extensions

### `suppressExpoWarnings`

A function that suppresses EXGL compatibility warnings and logs them instead.
You will need to import the `ExpoTHREE.THREE` global instance to use this. By
default this function will be activated on import.

* `shouldSuppress`: boolean

```js
import { THREE } from 'expo-three';
THREE.suppressExpoWarnings();
````

---

## ⛓ Links

Somewhat out of date

- [Loading Text](https://github.com/EvanBacon/expo-three-text)
- [three.js docs](https://threejs.org/docs/)

- [Random Demos](https://github.com/EvanBacon/expo-three-demo)
- [Game: Expo Sunset Cyberspace](https://github.com/EvanBacon/Sunset-Cyberspace)
- [Game: Crossy Road](https://github.com/EvanBacon/Expo-Crossy-Road)
- [Game: Nitro Roll](https://github.com/EvanBacon/Expo-Nitro-Roll)
- [Game: Pillar Valley](https://github.com/EvanBacon/Expo-Pillar-Valley)
- [Voxel Terrain](https://github.com/EvanBacon/Expo-Voxel)

## 🤝 Contributing

Contributions, issues and feature requests are welcome!<br />Feel free to check [issues page](https://github.com/Expo/expo-three/issues).

## Show your support

Give a ⭐️ if this project helped you!

## 📝 License

Copyright © 2019 [650 Industries](https://expo.io/about).<br />
This project is [MIT](https://github.com/Expo/expo-three/blob/master/LICENSE) licensed.
