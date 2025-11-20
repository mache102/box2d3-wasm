# See [box2d3-wasm](box2d3-wasm).

*This outer layer is just being used to host a VSCode multi-root workspace, so that we can scope `.vscode` directories to each project.*

## Rebuilding (compat) with our box2d changes

- Modified box2d to keep dynamic-dynamic speculative collisions, and disable dynamic-static speculative collisions.


```bash
cd box2d3-wasm  
rm -rf cmake-build-compat

FLAVOUR=compat TARGET_TYPE=Release ./shell/0_build_makefile.sh

emmake make -j8 -C cmake-build-compat

FLAVOUR=compat TARGET_TYPE=Release ./shell/1_build_wasm.sh
```

Files are produced in `box2d3-wasm/build/dist/es/compat/`.

```
ls build/dist/es/compat/

Box2D.compat.d.ts  
Box2D.compat.mjs  
Box2D.compat.orig.d.ts  
Box2D.compat.orig.mjs  
Box2D.compat.wasm
```