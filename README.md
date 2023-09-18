
**WinSyn Dataset**

WinSyn is a dataset of photographs of building windows from around the world. It is described in the publication xxx. 

This document describes the _full_ dataset, it contains the high resolution photots - with jpg and raw files as output from the camera. There are also _pre-rendered_ datasets which make various assumptions. These are (probably easier to use and) available from the Kaust datastore:

* [9k labelled real photos at 1024px resolution]()
* [80k 1024px photos of windows]()
* [16k 512px synthetic renders of windows]()

The rest of this document describes the organisation of the full dataset, and the tools available to process it. As well as the original jpg and raw photos, you can find labels polygons, crop information, location information, and a simple website to view the data. There are tools to generate the above pre-rendered datasets and crop images.

**File organisation** 

The data directory is the "root" of the project. This contains various folders (_photos_, _metadata_single_elments_, _metadata_website_, etc…) which each contain a different type of data.  
 
The contents of the folders are divided into _batches_ (subfolders) such as _tom_london_20220418_ for easier processing. They are named for the first name of the photographer, principal location, and date. For example, metadata relating to the image:

 _data/photos/tom_london_20220418/IMG_0206.JPG_ 

can be found in:

_data/**/tom_london_20220418/IMG_0206.*_

where ** is a metadata folder (metadata_single_elements, metadata_window_labels, ...) and * is an extension dependent on the data type (usually ._json_). You can see a summary of available information for each photograph at the bottom of the photo webpage: 

_data/metadata_website/tom_london_20220418/IMG_0206.html_

**Where to get the data** 
 
The project's data and code is split between different sources. Binary formatted JPG images are available from the Kaust data repository, while metadata and source code is available from our Github repos. 

Please observe our [dataset conditions](http://todo.com) of use at all times. Be aware of situations where data may leak, e.g., due to loose directory permissions, caching, or model memorization. 

Available from the [Kaust datastore]([https://repository.kaust.edu.sa/](https://repository.kaust.edu.sa/handle/10754/694463):

* _photos_
    * The photos in jpg format and raw format.
    * the file size is around 4Tb.
    * The photographers were provided with this [guidance document](https://github.com/twak/winsyn_metadata/raw/docs/pdfs/labelling_instructions.pdf). 

Available from this [metadata repository](https://github.com/twak/winsyn_metadata):

* _metadata_single_elements_
    * The crop information to identify single rectangular samples of windows.
    * These were created manually with the _crop_tool.py_ script by Tom, Michaella, and Prem, with the goals of identifying single windows that we could send to be per-pixel labelled and regularising the different equipment and styles of the many different photographers.
        * Clusters of windows were sometimes annotated as a single window if they had shared frames/interconnected patterns.
        * With glass-façades, we tried to take 2-3 repetitions in each direction.
        * We tried to take no more than 4 similar windows from one façade - in one or multiple images.
        * We tried to keep the crops square where possible.
    * The tags `window`, `door`, `glass_facade`, `shop`, `church` and `abnormal` are treated as windows. Some other tags (`materials`, `facades`) are available but are not consistently applied.
    * This file also contains per-photo tags such as deletion (file should not included in dataset) and rotation (where exif rotation data is incorrect).
* _metadata_window_labels_
    * Per-pixel labels created by LYD for the first 3002 images. Annotated for the first 11 (12 including _none_) classes. The instructions given to the labelers were collated in this [document](https://github.com/twak/winsyn_metadata/raw/docs/pdfs/labelling_instructions.pdf).
    * Described as polygons; per-pixel bitmap datasets can be created with [process_labels](https://github.com/twak/fast_crop/blob/master/process_labels.py).py.
    * These labels mostly should not overlap.
* _metadata_window_labels_2_
    * Per-pixel Labels created by LYD for the next 6000 images. Annotated for 13 (14 including _none_) classes. 
    * These labels are defined with a z-order and overlap - they are rendered using this order to create the per-pixel labels.
* _metadata_location_
    * Describes location of photographs
    * _locations_data.json_ contains the per-batch location information used for creating per-image location description files.
    * The locations are of varying degrees of accuracy. The image source can be _coarse_ (located only by a named city), _camera_ (a GPS location captured by the camera), or _track_ (a location computed from the photograph time and a separately captured GPS track). Several batches have no geolocation information available.


The following can be generated from the above:

* _metadata_website_
    * The very simple website created by the [build_website.py](https://github.com/twak/fast_crop/blob/master/build_website.py) script.
    * It should be hosted on your webserver (it fetches html content for the batches).
    * _metadata_website/_index.html shows all photographs. Select the radio button next to a batch to view all the photos for that batch. An icon with a red cross has been deleted. An icon with labels has been labelled.
    * Clicking on an icon will take you to the webpage summarising all available information for each photo.
    * _metadata_website/_crops.html similarly shows an icon for each crop.
    * _metadata_website/map _shows the locations of the photos. Zooming in and clicking on a blue marker will show an icon and link to the photo summary page.
* _winsyn_cook_{time.time}.zip_
    * Datasets of images cropped to particular sizes with or without accompanying labels by the [render_crops_and_labels.py](https://github.com/twak/fast_crop/blob/master/render_crops_and_labels.py)  script.
* _metadata_location_
    * Computed by the build_locations.py script using information from _locations_data.json_ and the photos info/tracks themselves.

**Scripts from the [code repository](https://github.com/twak/fast_crop)**

These scripts expect to be run from the `data` directory - they tend not to take command line arguments, but rely on the cwd & editing scripts to change the parameters in a way which can be tracked by vcs. 

* [render_crops_and_labels.py](https://github.com/twak/fast_crop/blob/master/render_crops_and_labels.py) creates image + label datasets using the metadata_window_labels(_2) data.
* [render_crops.py](https://github.com/twak/fast_crop/blob/master/render_crops.py) creates cropped rgb images from the metadata_single_elements data.
* [build_website.py](https://github.com/twak/fast_crop/blob/master/build_website.py)
    * creates a website showing photos and crops by batches, and the photo locations. This is output to the metadata_website folder, which can be hosted by a webserver (e.g., Apache).
* [fast_crop.py](https://github.com/twak/fast_crop/blob/master/fast_crop.py)
    * the interactive tool used to create the metadata_single_elements folder. Allows windows (and other things) to be annotated.
    * the _tags.py_ describes the different types of rectangular crops that may be annotated. Only the window classes are reliably applied. The window subclasses  (glass_facade, church, shop, abnormal, windows) might not be so reliable. The other classes (façade, material, private) are quite irregular.
    * the entire photo can also be annotated with 
        * _deleted - _the photo has been deleted from the dataset due to poor quality, containing no windows, or only containing windows repeated elsewhere in the batch
        * _rot90, rot180, rot270 _- the photo has been manually rotated before cropping (after any exif-encoded rotate has been applied).
* build_locations.py
    * creates the metadata_location folder containing different sources.
* [summary.py](https://github.com/twak/fast_crop/blob/master/figure_summary.py), 
    * outputs various statistics for the whole dataset.
* figure_many_xxx.py
    * scripts used to create the figures for the paper.

**Other notes**

* The dataset was grown organically as resources and applications were added to the project. Therefore early image metadata may be of lower quality than later. 
* The batches started as a day of photography (for tom and michaella) or a contract (1 or 2 thousand images) for the upwork freelancers. 
* The "tom_archive_xxx" folders are images taken before the project started and are from a variety of hardware and locations. Mostly we don't have location information for these.
* The exploratory datasets were created by tom (on holiday) in UK/Denmark, later Michaella contributed images from Austria and Germany. In the third phase we contracted freelancers on the upwork platform to collect images from other locations around the world. Adherance to the provided guidance document was generally good, but a minority of the freelance photographers did not follow this document - these were largely deleted during cropping.
* The guidance documents provided to the photographers and labellers evolved as edge-cases and issues with data collection were identified. For example, specific instructions for labelling unusual configurations of sunshades and Egyptian windows were added. The dates of major changes are noted at the top of these documents.
