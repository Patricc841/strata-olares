# strata-olares

Builds the [niko1221/strata](https://github.com/niko1221/strata) Docker image (CUDA arch 120) for the Olares One
and holds the Olares chart. Run the `build-image` workflow manually; needs secrets `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`.

## Layout
- `.github/workflows/build.yml` - builds the image (engine compiled with `STRATA_PORTABLE=ON`, CUDA arch 120)
- `chart/strata/` - the Olares chart (Qwen3.8-Flash-Next IQ3_XXS, 256k context, vision on)

Deploy: `olares-cli chart package chart/strata`, then `market upload` and `market install` (or `upgrade`) with `-s upload`.
On a GPU resume, bind the device: `market resume strata --compute-binding <node>:<gpu-id>`.
The entrance host names must be listed in `strata.allowedHosts` (values.yaml), otherwise Strata answers 403.
