# Setup

## 1. Pull Submodules

The following code will pull external git submodules. It needs to be run only once (after `git clone` of the sam3d codebase).
```bash
git submodule update --init --recursive
```

## 2. Setup Python Environment

The following will install the default environment. If you use `conda` instead of `mamba`, replace its name in the first two lines. Note that you may have to build the environment on a compute note with GPU (e.g., you may get a `RuntimeError: Not compiled with GPU support` error when running certain parts of the code that use Pytorch3D).

```bash
# create sam3d-imag environment
mamba env create -f environments/default.yml
mamba activate sam3d-objects

# for pytorch/cuda dependencies
export PIP_EXTRA_INDEX_URL="https://pypi.ngc.nvidia.com https://download.pytorch.org/whl/cu121"

# install sam3d-objects and core dependencies
pip install -e '.[dev]'
pip install -e '.[p3d]' # pytorch3d dependency on pytorch is broken, this 2-step approach solves it

# for inference
export PIP_FIND_LINKS="https://nvidia-kaolin.s3.us-east-2.amazonaws.com/torch-2.5.1_cu121.html"
pip install -e '.[inference]'

# patch things that aren't yet in official pip packages
./patching/hydra # https://github.com/facebookresearch/hydra/pull/2863
```

## 3. Getting Checkpoints

### From HuggingFace

```bash
pip install -U "huggingface_hub[cli]"

TAG=hf
hf download \
  --repo-type model \
  --local-dir checkpoints/${TAG}-download \
  --max-workers 1 \
  facebook/sam-3d-objects
mv checkpoints/${TAG}-download/checkpoints checkpoints/${TAG}
rm -rf checkpoints/${TAG}-download
```

### From Meta

```bash
TAG=public_v1
CHECKPOINT_PATH=checkpoints/${TAG}
FILES=(
    pipeline.yaml
    slat_decoder_mesh.pt
    slat_decoder_mesh.yaml
    slat_generator.ckpt
    slat_generator.yaml
    ss_decoder.ckpt
    ss_decoder.yaml
    ss_encoder.safetensors
    ss_encoder.yaml
    ss_generator.ckpt
    ss_generator.yaml
    slat_decoder_gs.ckpt
    slat_decoder_gs.yaml
    slat_decoder_gs_4.ckpt
    slat_decoder_gs_4.yaml
)

mkdir -p ${CHECKPOINT_PATH}
for file in "${FILES[@]}"; do
  wget -xv http://dl.fbaipublicfiles.com/sam-3d-objects/checkpoints/${TAG}/${file} -O ${CHECKPOINT_PATH}/${file}
done
```
