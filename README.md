# strata-olares

Builds the [niko1221/strata](https://github.com/niko1221/strata) Docker image (CUDA arch 120) for the Olares One
and holds the Olares chart. Run the `build-image` workflow manually; needs secrets `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`.

## Layout
- `.github/workflows/build.yml` - builds the image (engine compiled with `STRATA_PORTABLE=ON`, CUDA arch 120)
- `chart/strata/` - the Olares chart (Qwen3.8-Flash-Next IQ3_XXS, 256k context, vision on)

Deploy: `olares-cli chart package chart/strata`, then `market upload` and `market install` (or `upgrade`) with `-s upload`.
On a GPU resume, bind the device: `market resume strata --compute-binding <node>:<gpu-id>`.
The entrance host names must be listed in `strata.allowedHosts` (values.yaml), otherwise Strata answers 403.

## Shared app / how other apps connect
Strata is installed as a shared app (`options.shared`, `onlyAdmin`, namespace `strata-shared`). The service DNS name
(`strata.strata-shared:8080`) is NOT reachable from other apps (504/503); use the internal entrance instead:
`http://<hash>1.<zone>/v1` (`strata-api`, `authLevel: internal`). Find it with `olares-cli settings apps entrances list strata`
(the hash changes on reinstall). No API key is needed. Uninstalling deletes the node-local cache, so the model is downloaded again.
