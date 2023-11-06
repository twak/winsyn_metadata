
# WinSyn Dataset

![Teaser figure showing real photos, crops, and synthetic images of building windows](https://github.com/twak/winsyn_metadata/blob/docs/images/overview.jpg?raw=true)

WinSyn is a dataset of photographs of building windows from around the world. It is described in the publication [A High Resolution Testbed for Synthetic Data](https://arxiv.org/abs/2310.08471) by Tom Kelly (Kaust), John Femiani (MiamiOh), and Peter Wonka (Kaust).

This document describes the _full_ dataset, it contains the high resolution photots - with jpg and raw files as output from the camera. There are also _pre-rendered_ datasets which make various assumptions. These are (probably easier to use and) available from the Kaust datastore:

* [9k labeled real photos at 1024px resolution](http://dx.doi.org/10.25781/KAUST-8YL8A)
* [89k 512px crops or 97k 1024px crops](http://dx.doi.org/10.25781/KAUST-8YL8A)  
* [72k 2024px photos of windows](http://dx.doi.org/10.25781/KAUST-LWC2Z)

Our synthetic renders are also available:

* [synthetic renders of windows and variations (alpha release)](https://vision.csi.miamioh.edu/data/archinet_public/winsynthetic_jpg_v0.zip)
* [synthetic renders of windows: render-time "ms" dataset (alpha release)](https://vision.csi.miamioh.edu/data/archinet_public/winsynthetic_jpg_ms_v0.zip)

The rest of this document describes the organisation of the full dataset, and the tools available to process it. As well as the original jpg and raw photos, you can find labels polygons, crop information, location information, and a simple website to view the data. There are tools to generate the above pre-rendered datasets and crop images.

### file organisation

The _data_ directory is the "root" of the project. It contains various folders (_photos_, _metadata_single_elments_, _metadata_website_, etc…) which each contain a different type of data.
 
The contents of these folders are divided into _batches_ (subfolders) such as _tom_london_20220418_ for easier processing. They are named for the first name of the photographer, principal location, and date. For example, metadata relating to the image:

 _data/photos/tom_london_20220418/IMG_0206.JPG_ 

can be found in:

_data/**/tom_london_20220418/IMG_0206.*_

where ** is a metadata folder (metadata_single_elements, metadata_window_labels, ...) and * is an extension dependent on the data type (usually _.json_). You can see a summary of available information for each photograph at the bottom of the photo webpage: 

_data/metadata_website/tom_london_20220418/IMG_0206.html_

### where to download the data
 
![example windows and crop size distributions](https://github.com/twak/winsyn_metadata/blob/docs/images/morewindowsthanu.jpg?raw=true)

The project's data and code is split between different sources. Images files are available from the Kaust data repository, while metadata and source code is available in this github repo. 

<!-- [doi of datastore](http://dx.doi.org/10.25781/KAUST-6DZ5P) -->
Jpg and raw files are available from the [Globus datastore](https://app.globus.org/file-manager?origin_id=ba993270-4516-48b9-af7c-77e890865733):

* _photos_
    * the photos in jpg format and raw format
    * the total file size is around 4Tb
    * my notes on [using Globus](https://github.com/twak/winsyn_metadata/blob/docs/globus.md)
    * the photographers were provided with this [guidance document](https://github.com/twak/winsyn_metadata/raw/docs/pdfs/labelling_instructions.pdf) 

Other metadata available from this [repository](https://github.com/twak/winsyn_metadata):

* _metadata_single_elements_
    * The crop information to identify single rectangular samples of windows.
    * These were created manually with the _crop_tool.py_ script by Tom, Michaella, and Prem, with the goals of identifying single windows that we could send to be per-pixel labelled and regularising the different equipment and styles of the many different photographers.
        * Clusters of windows were sometimes annotated as a single window if they had shared frames/interconnected patterns.
        * With glass-façades, we tried to take 2-3 repetitions in each direction.
        * We tried to take no more than 4 similar windows from one façade - in one or multiple images.
        * We tried to keep the crops square where possible.
    * The tags `window`, `door`, `glass_facade`, `shop`, `church` and `abnormal` are treated as windows. Some other tags (`materials`, `facades`) are available but are not consistently applied.
    * This file also contains per-photo tags such as `deleted` (file should not included in dataset) and rotation: `rot90`, `rot180`, `rot270` (where exif rotation data is incorrect).
* _metadata_window_labels_
    * Per-pixel labels created by [LYD](https://labelyourdata.com/) for the first 3002 images. Annotated for the first 11 (12 including _none_) classes. The instructions given to the labelers were collated in this [document](https://github.com/twak/winsyn_metadata/raw/docs/pdfs/labelling_instructions.pdf).
    * Described as polygons; per-pixel bitmap datasets can be created with [process_labels](https://github.com/twak/fast_crop/blob/master/process_labels.py).py.
    * These labels mostly should not overlap.
* _metadata_window_labels_2_
    * Per-pixel Labels created by LYD for the next 6000 images. Annotated for 13 (14 including _none_) classes. 
    * These labels are defined with a z-order and overlap - they are rendered using this order to create the per-pixel labels.
* _metadata_location_
    * Describes location of photographs
    * _locations_data.json_ contains the per-batch location information used for creating per-image location description files.
    * The locations are of varying degrees of accuracy. The image source can be `coarse` (located only by a named city), `camera` (a GPS location captured by the camera), or `track` (a location computed from the photograph time and a separately captured GPS track). Several batches have no geolocation information available.
    * We were unable to verify the location accuracy of the batches `mussa_dar_es_salaam_20221212`, `jan_cebu_20230120`, and `sarabjot_newdehli_20230315`.


The following can be generated once you have the raw data and meta repository :

* _metadata_website_
    * The very simple website created by the [build_website.py](https://github.com/twak/fast_crop/blob/master/build_website.py) script.
    * An example of the site is currently hosted by [miamiOh](https://vision.csi.miamioh.edu/data/archinet/data/metadata_website/index.html).
    * It should be hosted on a webserver (it fetches html content for the batches).
    * metadata_website/index.html shows all photographs. Select the radio button next to a batch to view all the photos for that batch. An icon with a red cross has been deleted. An icon with labels has been labelled.
    * Clicking on an icon will take you to the webpage summarising all available information for each photo.
    * metadata_website/crops.html similarly shows an icon for each crop.
    * metadata_website/map shows the locations of the photos. Zooming in and clicking on a blue marker will show an icon and link to the photo summary page.
* metadata_cook/dataset_cook_crops_{resolution}px_{time.time()}.zip
    * Datasets of images cropped to particular sizes with or without accompanying labels by the [render_crops.py](https://github.com/twak/fast_crop/blob/master/render_crops.py)  script. Simiarly for [render_crops_and_labels.py](https://github.com/twak/fast_crop/blob/master/render_crops_and_labels.py).
<!-- * _metadata_location_ NOW IN REPO!
    * Computed by the build_locations.py script using information from _locations_data.json_ and the photos info/tracks themselves. -->

### data processing scripts from the [code repository](https://github.com/twak/fast_crop)

A collection of scripts to process the data are available from the fast_crop repo. They include scripts to render rops, labels, build the website, crop photos, summarize and validate the dataset, and create some of the published figures.

<img src="https://github.com/twak/winsyn_metadata/blob/docs/images/fast_crop_eg.jpg?raw=true" width="50%" height="50%">

### citing

Please site the below paper when using our work.

```
@misc{kelly2023winsyn,
      title={WinSyn: A High Resolution Testbed for Synthetic Data}, 
      author={Tom Kelly and John Femiani and Peter Wonka},
      year={2023},
      eprint={2310.08471},
      archivePrefix={arXiv},
      primaryClass={cs.CV}
}
```

### other notes

* The dataset was grown organically as resources and applications were added to the project. Therefore early image metadata may be of lower quality than later. 
* The batches started as a day of photography (for tom and michaella) or a contract (1 or 2 thousand images) for the upwork freelancers. 
* The "tom_archive_xxx" folders are images taken before the project started and are from a variety of hardware and locations. Mostly we don't have location information for these.
* The exploratory datasets were created by tom (on holiday) in UK/Denmark, later Michaella contributed images from Austria and Germany. In the third phase we contracted freelancers on the upwork platform to collect images from other locations around the world. Adherance to the provided guidance document was generally good, but a minority of the freelance photographers did not follow this document - these were largely deleted during cropping.
* The guidance documents provided to the photographers and labellers evolved as edge-cases and issues with data collection were identified. For example, specific instructions for labelling unusual configurations of sunshades and Egyptian windows were added. The dates of major changes are noted at the top of these documents.

### acknowledgements

We would like to thank our lead photographers Michaela Nömayr and Florian Rist, and engineer Prem Chedella, as well as our contributing photographers: Aleksandr Aleshkin, Angela Markoska, Artur Oliveira, Brian Benton, Chris West, Christopher Byrne, Elsayed Saafan, Florian Rist, George Iliin, Ignacio De Barrio, Jan Cuales, Kaitlyn Jackson, Kalina Mondzholovska, Kubra Ayse Guzel, Lukas Bornheim, Maria Jose Balestena, Michaela Nömayr, Mihai-Alexandru Filoneanu, Mokhtari Sid Ahmed Salim, Mussa Ubapa, Nestor Angulo Caballero, Nicklaus Suarez, Peter Fountain, Prem Chedella, Samantha Martucci, Sarabjot Singh, Scarlette Li, Serhii Malov, Simon R. A. Kelly, Stephanie Foden, Surafel Sunara, Tadiyos Shala, Susana Gomez, Vasileios Notis, Yuan Yuan, and finally [LYD](https://labelyourdata.com/) for the labeling.

We also thank the blender procedural artists Gabriel de Laubier for the [UCP Wood material](https://www.blendernation.com/2017/10/20/procedural-cc-0-pbr-wood-shader/) and Simon Thommes for the fantastic [Br'cks material](https://simonthommes.gumroad.com/l/Brcks). Both were modified and used in our procedural model.
