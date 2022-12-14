# Get better template matching results!

[Template matching]() is a wonderful method of creating automation bots in a very short period of time. 

But unfortunately, template matching usually fails when the screen size changes or when images are rotated. 

Rotated images are much less common than resized windows in desktop automation, this is why this module aims to solve the problem of different sizes rather than rotated images. It supports multiprocessing for faster results. WINDOWS ONLY

```python
# Let's say to want to automate Bluestacks, and
# therefore, need to detect different icons (needles) on your screen (haystack).
```

![_](https://github.com/hansalemaos/screenshots/raw/main/templatematching1.png "_").


```python
$pip install better-template-matching
from better_template_matching import PatternMatchingOnScreen,start_annotation_tool,create_needle_images_from_annotations
# Create an instance
template_matching = PatternMatchingOnScreen(
    scale_percent=25, needle_folder=None, debug_folder=None,
)
```


```python
# Choose a screenshot method to take the screenshot

# configure_window gets screenshots from a specific window, works also for background windows
template_matching.configure_window(regular_expression=r"[bB]lue[sS]tacks.*", hwnd=None)

# configure_monitor takes screenshots of the whole screen
template_matching.configure_monitor(monitor=1)

# If you are using bluestacks or an Android Phone, you can also connect over adb
template_matching.configure_adb(adb_path=r"C:\ProgramData\adb\adb.exe", adb_serial="localhost:5735")

```

```python
# Use save_screenshots_for_creating_needle_images to save screenshots on your HDD each time you press the hotkey
# (This step can also be done with any other screenshot tool)
template_matching.save_screenshots_for_creating_needle_images(
    folder="c:\\templatemat", hotkey="ctrl+alt+p"
)
```



```python

# After you are done, start the annotation tool (https://github.com/drainingsun/ybat) to crop the icons quickly.
# the requested class file (ybat) should look like this. It can be saved as txt
"""
playstore_icon
gamecenter_icon
systemapps_icon
roblox_icon
bluestacks_x_icon
spiele_und_gewinne_icon
kamera_icon
einstellungen_icon
chrome_icon
media_manager_icon
"""
# The files of the tool are included in this module.
# If you encounter any problem, download the ybat files and put them in the folder of this module
# After you are done, click on "Save COCO", and copy the link of the zipfile
# (This step can also be done with any other tool, e.g., Photoshop)

start_annotation_tool()
```

![_](https://github.com/hansalemaos/screenshots/raw/main/templatematching2.png "_").
![_](https://github.com/hansalemaos/screenshots/raw/main/templatematching3.png "_").



```python
# After you are done, use this function to format the screenshots
create_needle_images_from_annotations(
    cocojson=r"C:\Users\Gamer\Documents\Downloads\bboxes_coco.zip",  # generated file by ybat
    savefolder=r"C:\screenshots_for_detection",
    # The folder where the screenshots you took are located, In my case:"c:\\templatemat"
    outputfolder=r"C:\detectiontest",  # Folder to save the results, that means the needle images you are going to use.
    expand_x_negative=200,  # you can limit the search region on the screen - saves time and false positives
    expand_x_positive=200,
    expand_y_negative=200,
    expand_y_positive=200,
    
# After completing this step,
# you should have something like this in your output folder:
)
```
![_](https://github.com/hansalemaos/screenshots/raw/main/templatematching4.png "_").

```python
# (The 2 additional folders in the output folder can be deleted, they are for debugging)

# If you want to change the search region for a picture, you only have to change the file name:

#Here is one example:
# C:\detectiontest\playstore_icon--0x0--200x300.png
# The region (0,0), (200,300) will be checked for the image C:\detectiontest\playstore_icon--0x0--200x300.png

# if we rename the file to
# C:\detectiontest\playstore_icon--0x0--500x600.png
# the region will change to (0,0), (500,600)
# You can rename the file, but don't change the format (NAME)--(XCOORD)x(YCOORD)--(XCOORD)x(YCOORD).png
#

r"""
Some examples of file names
C:\detectiontest\playstore_icon--0x0--478x451.png
C:\detectiontest\gamecenter_icon--244x0--781x451.png
C:\detectiontest\systemapps_icon--528x0--1067x448.png
C:\detectiontest\roblox_icon--833x0--1342x448.png
C:\detectiontest\bluestacks_x_icon--1101x0--1643x449.png
C:\detectiontest\spiele_und_gewinne_icon--1347x0--1920x452.png
C:\detectiontest\kamera_icon--426x203--931x738.png
C:\detectiontest\einstellungen_icon--537x200--1038x735.png
C:\detectiontest\chrome_icon--643x199--1140x734.png
C:\detectiontest\media_manager_icon--744x194--1250x738.png
"""
```


```python
# The needle images have been created, so let's start from the beginning
# scale_percent is not important when creating the needle images,
# but it is for detection. The smaller the picture is, the faster
# is the detection, but the detection is usually not as good as with high resolution.
#
# You don't have to worry about resizing the output results when using scale_percent,
# The correct coordinates will be automatically calculated
# after the detection is done.

template_matching = (
    PatternMatchingOnScreen(
        scale_percent=25, needle_folder=r"C:\detectiontest", debug_folder=None,
    )
        # .configure_window(regular_expression=rr"[bB]lue[sS]tacks.*", hwnd=None)
        # .configure_adb(adb_path=r"C:\ProgramData\adb\adb.exe", adb_serial="localhost:5735")
        .configure_monitor(monitor=1)

)
```

```python
# Let's read the needle images that we have just created
# substract_zoom_percent and add_zoom_percent are important to detect different sizes.
# If you have a needle image with a size 100x100
# and pass substract_zoom_percent=3, add_zoom_percent=3
# the module will try to detect images with the size of:
# 97x97, 98x98, 99x99, 100x100, 101x101, 102x102, 103x103
# if you want all of needle images smaller than the original images, you could pass:
# get_needle_images(substract_zoom_percent=6, add_zoom_percent=-3)
# which will check for needle images with the 94-97% of the size of
# the original image
template_matching.get_needle_images(substract_zoom_percent=3, add_zoom_percent=3)
```


```python
# Lets get the results - grayscale=True is a lot faster, use it whenever you can.
# You can use multiple processors (workers) for the detection
# If you get an error (e.g., freeze_support), make sure you follow the multiprocessing guidelines:
# https://docs.python.org/3/library/multiprocessing.html
# if __name__ == '__main__': must be in your code when using multiprocessing (Windows)

df = template_matching.get_screenshot_and_start_detection(
    grayscale=True,
    interpolation=cv2.INTER_AREA,
    thresh=0.8,
    save_screenshot_in_folder=None,
    workers=3,
    show_results=False,
    sleep_time_for_results=0.1,
    quit_key_for_results="q",
).get_detection_results_as_df()
```


```python
r"""
#The coords/height/width have been automatically ajusted to the original size of the screenshot 

                                                          aa_filepath  aa_zoomfactor  aa_crop_x0  aa_crop_y0  aa_crop_x1  aa_crop_y1  aa_cropped_haystack_x  aa_cropped_haystack_y  aa_cropped_needle_x  aa_cropped_needle_y  aa_x  aa_y   aa_conf  aa_real_x_start  aa_real_y_start  aa_width  aa_height  aa_real_y_end  aa_real_x_end  aa_intersects                               aa_pure_filename  aa_same_zoom_factor
0               C:\detectiontest\roblox_icon--833x0--1342x448.png             22         183           0         295          98                    112                     98                   23                   32    33    23  0.990465              864               92        92        128            220            956           True               roblox_icon--833x0--1342x448.png                   10
1        C:\detectiontest\bluestacks_x_icon--1101x0--1643x449.png             22         242           0         361          98                    119                     98                   31                   36    32    19  0.981858             1096               76       124        144            220           1220           True        bluestacks_x_icon--1101x0--1643x449.png                   10
2  C:\detectiontest\spiele_und_gewinne_icon--1347x0--1920x452.png             22         296           0         422          99                    105                     99                   49                   36    32    19  0.975440             1312               76       196        144            220           1508           True  spiele_und_gewinne_icon--1347x0--1920x452.png                   10
3             C:\detectiontest\chrome_icon--643x199--1140x734.png             22         141          43         250         161                    109                    118                   21                   29    34    44  0.967109              700              348        84        116            464            784           True             chrome_icon--643x199--1140x734.png                   10
4      C:\detectiontest\media_manager_icon--744x194--1250x738.png             22         163          42         275         162                    112                    120                   23                   31    34    44  0.966091              788              344        92        124            468            880           True      media_manager_icon--744x194--1250x738.png                   10
5               C:\detectiontest\playstore_icon--0x0--478x451.png             22           0           0         105          99                    105                     99                   23                   32    29    23  0.963209              116               92        92        128            220            208           True               playstore_icon--0x0--478x451.png                   10
6      C:\detectiontest\einstellungen_icon--537x200--1038x735.png             22         118          44         228         161                    110                    117                   22                   29    34    44  0.952688              608              352        88        116            468            696           True      einstellungen_icon--537x200--1038x735.png                   10
7              C:\detectiontest\kamera_icon--426x203--931x738.png             22          93          44         204         162                    111                    118                   23                   29    34    45  0.952173              508              356        92        116            472            600           True              kamera_icon--426x203--931x738.png                   10
8            C:\detectiontest\gamecenter_icon--244x0--781x451.png             22          53           0         171          99                    118                     99                   30                   34    35    21  0.941764              352               84       120        136            220            472           True            gamecenter_icon--244x0--781x451.png                   10
9           C:\detectiontest\systemapps_icon--528x0--1067x448.png             22         116           0         234          98                    118                     98                   30                   34    34    21  0.938213              600               84       120        136            220            720           True           systemapps_icon--528x0--1067x448.png                   10
 """
```

```python
# if you want to see the results as a video
#
template_matching.show_results_as_video()
# https://github.com/hansalemaos/screenshots/raw/main/templatematching5.png
# https://github.com/hansalemaos/screenshots/blob/main/templatematching6.png
# The thicker the outline is, the more images with the same aspect ratio have been found,
# that usually means that the chance is lower of them being false positives
```

![_](https://github.com/hansalemaos/screenshots/raw/main/templatematching5.png "_").
![_](https://github.com/hansalemaos/screenshots/raw/main/templatematching6.png "_").


