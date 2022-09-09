
<!-- README.md is generated from README.Rmd. Please edit that file -->

![Image](./im/logo.png)

<!-- badges: start -->
<!-- badges: end -->

The goal of `cecelia` is to simplify image analysis for immunologists
and integrate static and live cell imaging with flow cytometry data. The
package primarily builds upon [`napari`](https://napari.org) and
[`shiny`](https://shiny.rstudio.com/). Our aim was to combine shiny’s
graph plotting engine with napari’s image display.

**This package is pre-alpha**

## Installation

**This package currently only works on Unix systems.** We have a Docker
version to support other systems if necessary, so do open an issue if
that is needed.

You can install the development version of cecelia like so:

``` r
if (!require("remotes", quietly = TRUE))
  install.packages("remotes")
remotes::install_github("schienstockd/cecelia")
```

For first time users, you will need to define base directory where
configuration files, models and the shiny app will be stored. `cecelia`
depends on a python environment which needs to be created. There are
multiple options available depending on how you would like to use the
app:

-   `image` For image analysis on Desktop

-   `image-nogui` For image processing without GUI

-   `flow` For flow cytometry analysis

``` r
library(cecelia)

# install all required R libraries
cciaRequirements()

# setup cecelia directory
cciaSetup("~/path/to/cecelia")

# create conda environment
cciaCondaCreate()
# cciaCondaCreate(envType = "image-nogui") # to use without gui
# cciaCondaCreate(envType = "flow") # for flow based only

# download models for deep-learning segmentation
cciaModels()
```

You have to adjust the parameters in `~/path/to/cecelia/custom.yml` to
your system. You need download/install:

-   [`bftools`](https://downloads.openmicroscopy.org/bio-formats/6.7.0/artifacts/bftools.zip)

-   [`bioformats2raw`](https://github.com/glencoesoftware/bioformats2raw/releases/download/v0.4.0/bioformats2raw-0.4.0.zip)

-   [`ImageJ`](https://imagej.net/imagej-wiki-static/Fiji/Downloads) if
    using Spot segmentation

For `ImageJ`, activate the following update sites:

-   IJPB-plugins

-   3D-ImageJ-Suite

-   Bio-Formats

``` yml
default:
  dirs:
    bftools: "/Applications/BFTools"
    bioformats2raw: "/Applications/glencoe/bioformats2raw-0.3.0"
    projects: "/your/project/directory/"
  volumes:
    SSD: "/your/ssd/directory/"
    home: "~/"
    computer: "/"
  python:
    conda:
      env: "r-cecelia-env"
      source:
        env: "r-cecelia-env"
  imagej:
    path: "/Applications/Fiji.app/Contents/MacOS/ImageJ-macosx"
```

## Image analysis general workflow

``` r
library(cecelia)
cciaUse("~/path/to/cecelia", initJupyter = TRUE)
cciaRunApp(port = 6860)
```

1.  Create a Project for `static` or `live`analysis.

2.  Images have to be imported as `OME-ZARR`. Choose `Import Images` and
    create an `Experimental Set`. It is helpful if all images within
    this set have the same colour combinations. Add Images. Select all
    images you want to import and choose `OME-ZARR`. Select the required
    `pyramid scales` and `run` the task.

3.  Select `Image Metadata` and click `Load Metadata` to load the
    channel information. You can assign channels either one by one by
    selecting a channel and `Specify Value` \> `Assign Value`.
    Alternatively, you can give a list of channels in the box and click
    `Assing channels`. You can add further experimental attributes by
    `Create Attribute` and adding respecive values for the individual
    images.

4.  Select `Cell Segmentation` to segment your images.

5.  Select `Population Gating` to use flow cytometry like gating to
    define populations. Select `Populations Clustering` to use cluster
    algorithms to define populations.

6.  Select `Spatial Analysis` to define spatial neighbourhoods, detect
    clustering cells or detect contact between cells.

## Flow Cytometry general workflow

``` r
library(cecelia)
cciaUse("~/path/to/cecelia", initJupyter = FALSE)
cciaRunApp(port = 6860)
```

1.  Create a Project for `flow`analysis.

2.  `FCS` files can be imported either from raw or other sources such as
    `FlowJo`. They will converted into an
    [`Anndata`](https://anndata.readthedocs.io) to perform clustering
    and a [`GatingSet`](https://github.com/RGLab/flowWorkspace) to
    perform manual gating.

3.  The rest of the pipeline is the same as for image analysis.

## Running workflows from `RMarkdown`

All Processing available in the app can be done from `RMarkdown` as
well. Every image is `ReactivePersistentObject` whose state is saved in
an `RDS` file and is `reactive` in a `shiny` app context. These objects
can be loaded and manipulated as such:

``` r
library(cecelia)

# set test variables
pID <- "pEdOoZ"   # project ID
versionID <- 2    # version ID
uID <- "tPl6da"   # image ID

# init ccia object
cciaObj <- initCciaObject(
  pID = pID, uID = uID, versionID = versionID, initReactivity = FALSE
)

funParams <- list(
  pyramidScale = 4,
  dimOrder = "",
  createMIP = FALSE,
  rescaleImage = FALSE
)

cciaObj$runTask(
  funName = "importImages.omezarr",
  funParams = funParams,
  runInplace = TRUE
)
```

## Creating plots

One aim of `ceelia` is to provide plotting over the GUI with `shiny`. We
are currently in the process of testing processing data, this part of
the app is less developed as we learn which plots and statistics are
relevant. Plotting is currently done in `RMarkdown` files but we are
planning to incorporate these into the app.

``` r
# init experimental set
cciaObj <- initCciaObject(
  pID = pID, uID = "U7LRc9", versionID = versionID, initReactivity = FALSE
)

# get image attributes
exp.info <- cciaObj$summary(withSelf = FALSE, fields = c("Attr"))

# get cluster populations
popDT <- cciaObj$popDT(popType = "clust", includeFiltered = TRUE)

# now you can plot this as a normal data.table
library(ggplot2)

ggplot(popDT, aes(centroid_x, centroid_y)) +
  theme_classic() +
  geom_point(aes(color = as.factor(clusters)), size = 0.5) +
  facet_wrap(.~Treatment, scales = "free")
```
