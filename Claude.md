## Proposed Project for Claude-Science

1. given the project: https://github.com/rmauron/HDCA_heart_dev
2. data found at: https://zenodo.org/records/15912657
3. paper attached
4. using Scanpy, Squidpy, Visium, and python tools
5. dowload the processed data
6. verify quality
7. start with a single analysis, like ventricles or atrioventricular plane, or other (the simplest)
8. show the UMAP-cell type analysis
9. show the spatial plot (image + single-cell transcriptomics) mapping different highly variable genes or DEGs


## Data

### Mendeley

curl -s -m 60 "https://data.mendeley.com/datasets/bundle.js?342865b28b070babe69b" -o bundle.js -w "code=%{http_code} sz=%{size_download}\n"; echo "--- datasets-v2 route strings:"; grep -oE "datasets-v2[A-Za-z0-9_/{}\\\$\.\?=&:'\"+,()-]{0,90}" bundle.js | sort -u | head -30


- testing:
cd repo   
Z="https://prod-dcd-datasets-cache-zipfiles.s3.eu-west-1.amazonaws.com/fhtb99mdzd-1.zip"  
echo "--- plain GET first bytes:"; curl -s -m 40 -o first.bin -w "code=%{http_code} sz=%{size_download}\n" --max-filesize 100000 "$Z"; ls -la first.bin 2>/dev/null; xxd first.bin 2>/dev/null | head -3
echo "--- HEAD:"; curl -s -m 40 -I "$Z" | head -12


cd repo 
curl -s -m 40 "https://data.mendeley.com/api/datasets-v2/datasets/w65jtfsvpr/zip?version=1"; echo
Z="https://prod-dcd-datasets-cache-zipfiles.s3.eu-west-1.amazonaws.com/fhtb99mdzd-1.zip"
echo "=== range support ==="; curl -s -m 40 -o /dev/null -D - -r 0-99 "$Z" -w "code=%{http_code} got=%{size_download}\n" | grep -iE "^(accept-ranges|content-range|content-length|code)"
￼
### Claude running

The full dataset structure is resolved: 34 Visium Space Ranger sections, plus Cell Ranger scRNA-seq, ISS, and metadata. Let me inspect the metadata and one section's contents to choose the simplest analysis target.


#### metadata

import os, subprocess
os.makedirs("data/meta", exist_ok=True)
want = {
 "HDCA_heart_ST_annotations.csv":"fba18f5b-3fe5-43f7-999b-d0b60d075bc8",
 "HDCA_heart_ST_sections_overview_chambers_present.xlsx":"b39a1db6-b213-46ca-8d8a-120f0837cd7c",
 "HDCA_heart_SC_annotations_HL_240115.csv":"18f67352-f5f4-49dc-8e5d-c2c4af4142e7",
}
for fn, fid in want.items():
    url=f"https://data.mendeley.com/public-files/datasets/fhtb99mdzd/files/{fid}/file_downloaded"
    dst=f"data/meta/{fn}"
    if not os.path.exists(dst):
        subprocess.run(["curl","-sL","-m","120",url,"-o",dst],check=True)
    print(f"{os.path.getsize(dst)/1e3:9.1f} KB  {fn}")


#### annotation tables

import pandas as pd
ann = pd.read_csv("data/meta/HDCA_heart_ST_annotations.csv")
print("=== ST annotations ==="); print(ann.to_string()[:1500])
hl = pd.read_csv("data/meta/HDCA_heart_SC_annotations_HL_240115.csv")
print("\n=== SC high-level annotations ==="); print(hl.to_string()[:1200])


#### Visium data

import os, subprocess, json, io

def list_files(fid):
    u=f"https://data.mendeley.com/public-api/datasets/fhtb99mdzd/files?folder_id={fid}&version=1"
    return json.load(io.StringIO(subprocess.run(["curl","-s","-m","60",u],capture_output=True,text=True).stdout))

def fetch(fid, dst):
    if os.path.exists(dst) and os.path.getsize(dst)>0: return os.path.getsize(dst)
    os.makedirs(os.path.dirname(dst), exist_ok=True)
    url=f"https://data.mendeley.com/public-files/datasets/fhtb99mdzd/files/{fid}/file_downloaded"
    subprocess.run(["curl","-sL","--retry","3","-m","900",url,"-o",dst],check=True)
    return os.path.getsize(dst)

SEC="V10A13-157_A1"; ROOT_ID="65f8327c-2c4a-4d33-bc74-65d624b35996"; SPA_ID="1cc034e7-7921-4e02-b7fb-357fc971a8e1"
base="data/"+SEC
for f in list_files(ROOT_ID):
    fn=f["filename"]; print("%9.2f MB  %s" % (fetch(f["id"], base+"/"+fn)/1e6, fn))
for f in list_files(SPA_ID):
    fn=f["filename"]; print("%9.2f MB  spatial/%s" % (fetch(f["id"], base+"/spatial/"+fn)/1e6, fn))

Show output
- tissue_hires_image.png
- tissue_hires_image.png
- The H&E image shows a PCW8 heart with clearly resolved chamber walls, trabeculated myocardium, and a valve/outflow region. Let me check the environment build.



#### Quality

Quality is high: median 12.5k counts and 3,828 genes per spot, only 3 low-count spots. Let me build the QC figure.


#### Image

import shutil
from PIL import Image
Image.MAX_IMAGE_PIXELS=None
# Space Ranger / scanpy expects a lowres image too; derive it from hires using scalefactors
sf=json.load(open(base+"/spatial/scalefactors_json.json"))
hi=Image.open(base+"/spatial/tissue_hires_image.png")
ratio=sf["tissue_lowres_scalef"]/sf["tissue_hires_scalef"]
lo=hi.resize((int(hi.width*ratio), int(hi.height*ratio)), Image.LANCZOS)
lo.save(base+"/spatial/tissue_lowres_image.png")
print("hires",hi.size,"-> lowres",lo.size,"ratio",round(ratio,4))

ad = sc.read_visium(base, count_file="filtered_feature_bc_matrix.h5", library_id=SEC)
ad.var_names_make_unique()
print(ad)


#### Squidy image


import squidpy as sq
fig, axes = plt.subplots(1, 2, figsize=(11, 5.2))
sq.pl.spatial_scatter(ad, color=KEY, size=1.35, img_alpha=.85, ax=axes[0], frameon=False, legend_loc="right margin", title="")
sc.pl.umap(ad, color=KEY, ax=axes[1], show=False, frameon=False, title="", legend_loc="on data")
fig.savefig("_check.png", dpi=130, bbox_inches="tight")
print("ok")


#### Tissue plot

sf_h = sf["tissue_hires_scalef"]
xs, ys = xy[:,0]*sf_h, xy[:,1]*sf_h
pad = 40
XL = (xs.min()-pad, xs.max()+pad); YL = (ys.max()+pad, ys.min()-pad)
print("hires img", ad.uns["spatial"][list(ad.uns["spatial"])[0]]["images"]["hires"].shape)
print("x", np.round(XL,1), "y", np.round(YL,1))

#### Rendering spatial gene expression panels

genes = ["NPPA","MYH7","BMP10","PITX2","MYL2","DCN","POSTN","ELN"]
notes = {"NPPA":"trabecular + atrial CM","MYH7":"compact myocardium","BMP10":"right atrium",
         "PITX2":"left atrium","MYL2":"ventricular CM","DCN":"interstitial fibroblasts",
         "POSTN":"valve mesenchyme","ELN":"great-vessel wall"}
I = {g: float(mor.loc[g,"I"]) for g in genes}

fig, axes = plt.subplots(2, 4, figsize=(13.6, 7.2))
for ax, g in zip(axes.ravel(), genes):
    sq.pl.spatial_scatter(ad, color=g, size=1.35, img_alpha=.55, ax=axes.ravel()[0] if False else ax,
                          frameon=False, title="", cmap="magma", use_raw=True, colorbar=True)
    ax.set_xlim(*XL); ax.set_ylim(*YL)
    ax.set_title(f"$\\it{{{g}}}$ — {notes[g]}", loc="left", fontsize=8)
    ax.annotate(f"Moran's I = {I[g]:.2f}", xy=(.98,.02), xycoords="axes fraction", ha="right",
                fontsize=6, color="white")
axes.ravel()[0].plot([XL[0]+60, XL[0]+60+L],[YL[0]-70]*2, color="black", lw=2, solid_capstyle="butt")
axes.ravel()[0].annotate("500 µm", xy=(XL[0]+60+L/2, YL[0]-24), ha="center", fontsize=6)
fig.suptitle("Spatially variable genes on the H&E section — PCW 8 human heart (V10A13-157_A1)", x=.008, ha="left", fontsize=10)
fig.text(.008,.005,"Log-normalised expression per 55 µm Visium spot overlaid on the haematoxylin & eosin image. "
         "Genes are the highest-ranking spatially autocorrelated HVGs (Moran's I, all FDR < 0.001); "
         "colour scales are independent per panel.", fontsize=6, color=META_GREY)
fig.tight_layout(rect=[0,.03,1,.96])
fig.savefig("spatial_genes.png", dpi=300, bbox_inches="tight")
print("saved")


#### Leiden clustering

Leiden clustering at resolution 0.5 gives six domains. Rather than label them from markers alone, I cross-checked each against its position in the tissue and against the atlas's own annotation vocabulary:

Domain	Spots	Defining markers
Left atrial myocardium	119	PITX2, MYL4, KCNA5, NR2F2
Right atrial myocardium	107	BMP10, ANGPT1, MYL4, NPPA
Compact ventricular myocardium	343	MYH7, MYL2, HEY2, TNNI3
Trabecular ventricular myocardium	265	NPPA, IRX3, IRX5, NMRK2
Fibrous–interstitial myocardium	154	DCN, OGN, ASPN, MGP, COL12A1
Valve mesenchyme & vessel wall	79	POSTN, HAPLN1, SOX9, ELN, ACTA2
Two findings worth flagging:

The left/right atrial split was not obvious from clustering alone — both atrial clusters are NPPA/MYL4-high and adjacent in UMAP space. A direct differential test separated them cleanly: PITX2 (logFC 4.7, the canonical left-atrial identity gene) marks one, BMP10 (logFC 5.6) the other. Their positions on opposite sides of the section confirm it.

Epicardium does not form its own domain, despite being in the atlas vocabulary (V_EP, A_EP). I tested this rather than assume it — scoring epicardial markers (WT1, UPK3B, BNC1, TBX18, MSLN) gave near-zero means in every cluster, and a distance-transform depth measurement showed no cluster confined to the outer surface. A 1–2 cell epicardial layer is thinner than a 55 µm Visium spot, so its signal is diluted into neighbouring myocardium. Detecting it would need higher-resolution technology, not better clustering. The same limit applies to the conduction system (HCN4, SHOX2, TBX3 all flat).

The compact/trabecular ventricular split is the clearest validation: median depth from the outer tissue surface is 4.1 lattice units for compact versus 9.0 for trabecular, matching the known inside-out architecture of the ventricular wall.

Spatial expression
