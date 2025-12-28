# See [box2d3-wasm](box2d3-wasm).

*This outer layer is just being used to host a VSCode multi-root workspace, so that we can scope `.vscode` directories to each project.*

## Rebuilding (compat) with our box2d changes

- Modified box2d to keep dynamic-dynamic speculative collisions, and disable dynamic-static speculative collisions.
  - To build box2d wihtout this change, modify `shell/0_build_makefile.sh` to remove the line:
    ```
    -DBOX2D_DISABLE_STATIC_DYNAMIC_SPECULATIVE=ON
    ```

```bash
# the box2d3-wasm directory **inside** the repo (also called box2d3-wasm)
cd box2d3-wasm  

# pull updated box2d submodule 
git submodule update --init --recursive --remote

rm -rf cmake-build-compat

FLAVOUR=compat TARGET_TYPE=Release ./shell/0_build_makefile.sh

emmake make -j8 -C cmake-build-compat

FLAVOUR=compat TARGET_TYPE=Release ./shell/1_build_wasm.sh
```

Files are produced in `box2d3-wasm/build/dist/es/compat/` (relative to the base `box2d3-wasm` directory).

```bash
ls build/dist/es/compat/ # assumes you are in ./box2d3-wasm  (relative to repo root)

Box2D.compat.d.ts  
Box2D.compat.mjs  
Box2D.compat.orig.d.ts  
Box2D.compat.orig.mjs  
Box2D.compat.wasm
```


## Using files in a Vite project

You will only need:
```
Box2D.compat.d.ts  
Box2D.compat.mjs  
Box2D.compat.wasm
```

### Js wrapper and types

Copy `.d.ts` and `.mjs` to a folder in your project, i.e. `src/lib/box2d3-custom/`.

Add to `vite.config.js` (if not already done): 
```js
alias: {
  'box2d3_dist': path.resolve(__dirname, 'src/lib/box2d3-custom/Box2D.compat.mjs'),
}
```

And optionally to `tsconfig.json`:
```json
"paths": {
  "box2d3_dist": ["src/lib/box2d3-custom/Box2D.compat"] 
},
```

### Wasm file

Copy the `.wasm` file to your project's `public/` folder, i.e. `public/wasm/Box2D.compat.wasm`.

Ensure that vite serves the wasm file correctly. You may need to add to `vite.config.js`:
```js
server: { 
  configureServer(server) {
    server.middlewares.use((req, res, next) => {
      res.setHeader('Cross-Origin-Opener-Policy', 'same-origin');
      res.setHeader('Cross-Origin-Embedder-Policy', 'require-corp');
      if (req.url?.endsWith('.wasm')) {
        res.setHeader('Content-Type', 'application/wasm');
      }
      next();
    });
  },
  
  // ...
},
```

### Loading the module

```ts

import Box2DFactory from 'box2d3_dist'; 

const box2d = await Box2DFactory({ // locatefile to find the wsam 
  locateFile: (path: string, prefix: string) => {
    if (path.endsWith('.wasm')) {
      return '/wasm/Box2D.compat.wasm'; // relative to public/
    }
    return prefix + path;
  }
});

export const {  
  b2DefaultWorldDef, 
  // ...
} = box2d;
```

## Piston Velocity

A new function `b2Body_SetPistonVelocity` has been exposed:

```cpp
/// Set the piston velocity of a body. Usually in meters per second.
/// The velocity must be axis aligned (x or y).
/// This velocity is applied for one time step and then reset to zero.
/// Dynamic bodies touching the piston will be pushed but will not acquire the velocity.
void b2Body_SetPistonVelocity( b2BodyId bodyId, b2Vec2 velocity );
```

## Adding new functions

To add new functions to the Box2D WASM module, follow these steps:
- Add it in `box2d3-wasm/csrc/glue.cpp` and bind it using `EMSCRIPTEN_BINDINGS`. 
Example; we added it between these two, for SetPistonVelocity:


```cpp
    function("b2Body_SetLinearVelocity", &b2Body_SetLinearVelocity);
    function("b2Body_SetPistonVelocity", &b2Body_SetPistonVelocity);
    function("b2Body_SetAngularVelocity", &b2Body_SetAngularVelocity);
```
Then re-run build scripts (starting from here):

```bash
FLAVOUR=compat TARGET_TYPE=Release ./shell/0_build_makefile.sh
```