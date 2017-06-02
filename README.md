# SOM-ORTHANC

This is a Docker image configured to run [orthanc](http://book.orthanc-server.com/users/docker.html) with a simple postgres database. To use, you will need to install Docker and docker-compose. Then:

```
git clone https://www.github.com/som/som-orthanc.git
cd som-orthanc
docker-compose up -d
```

ports 104 and 80 should connect to Orthanc, so you should be able to open [http://localhost](http://localhost) to see things.

How do you see your images? Like this:

```
docker-compose ps
docker ps
```

You will probably want to shell into the image to look around. I usually do this (note that I got the name from the `ps` commands above:

```
NAME=$(docker ps -aqf "name=somorthanc_orthanc_1")
docker exec -it $NAME bash
```

The `i` means interactive, and the `t` means terminal. So you are asking to run `bash` via an interactive terminal.

# Uploading Dicom
I found I could just drag and drop files on the web interface, even my janky cookie dicoms seemed to work ok :) It should be located at [http://localhost/app/explorer.html#upload](http://localhost/app/explorer.html#upload).

And this is how you would use (for example, storescu) to upload files:

```
storescu -aec SOMORTHANC localhost 4242 *.dcm
```

Now that I have this running, I'm going to give it a test and keep working on my dicom tools :)


# Plugins
The application seems to be focused around plugins, which I found here:

```
ls /usr/share/orthanc/plugins/
libOrthancDicomWeb.so           libOrthancWSI.so
libOrthancPostgreSQLIndex.so    libOrthancWebViewer.so
libOrthancPostgreSQLStorage.so
```

I also found some command line executables:

```
$ cd /usr/local/bin
$ ls
OrthancRecoverCompressedFile  OrthancWSIDicomToTiff  OrthancWSIDicomizer
```

## OrthancRecoverCompressedFile

```
OrthancRecoverCompressedFile       
Maintenance tool to recover a DICOM file that was compressed by Orthanc.

Usage: OrthancRecoverCompressedFile <input> [output]
If "output" is not given, the data will be output to stdout
```

## OrthancWSIDicomToTiff
This is pretty neat, it seems to be a tool to convert dicoms to tiff, likely it's used for the web viewer.

```
OrthancWSIDicomToTiff --help
W0602 23:44:33.496989 FromDcmtkBridge.cpp:142] Loading the external DICOM dictionary "/usr/share/libdcmtk2/dicom.dic"
W0602 23:44:33.509186 ApplicationToolbox.cpp:230] Orthanc WSI version: mainline (20170308T122235)

Usage: OrthancWSIDicomToTiff [OPTION]... [INPUT] [OUTPUT]
Orthanc, lightweight, RESTful DICOM server for healthcare and medical research.

Convert a DICOM image for digital pathology stored in some Orthanc server as a
standard hierarchical TIFF (whose tiles are all encoded using JPEG).

Generic options:
  --help                Display this help and exit
  --version             Output version information and exit
  --verbose             Be verbose in logs

Options for the source DICOM image:
  --orthanc arg (=http://localhost:8042/)
                                        URL to the REST API of the target 
                                        Orthanc server
  --username arg                        Username for the target Orthanc server
  --password arg                        Password for the target Orthanc server

Options for the target TIFF image:
  --color arg           Color of the background for missing tiles (e.g. 
                        "255,0,0")
  --reencode arg        Whether to re-encode each tile in JPEG (no transcoding,
                        much slower) (Boolean)
  --jpeg-quality arg    Set quality level for JPEG (0..100)

```

###  OrthancWSIDicomizer 
This looks to be going the other way, creating dicom from a pathology image.

```
W0602 23:46:24.566482 FromDcmtkBridge.cpp:142] Loading the external DICOM dictionary "/usr/share/libdcmtk2/dicom.dic"
W0602 23:46:24.580463 ApplicationToolbox.cpp:230] Orthanc WSI version: mainline (20170308T122300)
E0602 23:46:24.580654 Dicomizer.cpp:587] No input file was specified

Usage: OrthancWSIDicomizer [OPTION]... [INPUT]
Orthanc, lightweight, RESTful DICOM server for healthcare and medical research.

Create a DICOM file from a digital pathology image.

Generic options:
  --help                Display this help and exit
  --version             Output version information and exit
  --verbose             Be verbose in logs
  --threads arg (=2)    Number of processing threads to be used
  --openslide arg       Path to the shared library of OpenSlide (not necessary 
                        if converting from standard hierarchical TIFF)

Options for the source image:
  --dataset arg         Path to a JSON file containing the DICOM dataset
  --sample-dataset      Display a minimalistic sample DICOM dataset in JSON 
                        format, then exit
  --reencode arg        Whether to re-encode each tile (no transcoding, much 
                        slower) (Boolean)
  --repaint arg         Whether to repaint the background of the image 
                        (Boolean)
  --color arg           Color of the background (e.g. "255,0,0")

Options to construct the pyramid:
  --pyramid arg (=0)    Reconstruct the full pyramid (slow) (Boolean)
  --smooth arg (=0)     Apply smoothing when reconstructing the pyramid 
                        (slower, but higher quality) (Boolean)
  --levels arg          Number of levels in the target pyramid

Options for the target image:
  --tile-width arg                      Width of the tiles in the target image
  --tile-height arg                     Height of the tiles in the target image
  --compression arg                     Compression of the target image 
                                        ("none", "jpeg" or "jpeg2000")
  --jpeg-quality arg                    Set quality level for JPEG (0..100)
  --max-size arg (=10)                  Maximum size per DICOM instance (in MB)
  --folder arg                          Folder where to store the output DICOM 
                                        instances
  --folder-pattern arg (=wsi-%06d.dcm)  Pattern for the files in the output 
                                        folder
  --orthanc arg (=http://localhost:8042/)
                                        URL to the REST API of the target 
                                        Orthanc server
  --username arg                        Username for the target Orthanc server
  --password arg                        Password for the target Orthanc server

Description of the imaged volume:
  --imaged-width arg (=15)  With of the specimen (in mm)
  --imaged-height arg (=15) Height of the specimen (in mm)
  --imaged-depth arg (=1)   Depth of the specimen (in mm)
  --offset-x arg (=20)      X offset the specimen, wrt. slide coordinates 
                            origin (in mm)
  --offset-y arg (=40)      Y offset the specimen, wrt. slide coordinates 
                            origin (in mm)

Advanced options:
  --optical-path arg (=brightfield) Optical path to be automatically added to 
                                    the DICOM dataset ("none" or "brightfield")
  --icc-profile arg                 Path to the ICC profile to be included. If 
                                    empty, a default sRGB profile will be 
                                    added.
  --safety arg (=1)                 Whether to do additional checks to verify 
                                    the source image is supported (might slow 
                                    down) (Boolean)
  --lower-levels arg                Number of pyramid levels up to which 
                                    multithreading should be applied (only for 
                                    performance/memory tuning)
```

A few notes from this [list](https://groups.google.com/forum/#!topic/orthanc-users/RO4XKCxDEAE) on how to scale.

>> Regarding Swarm, you'll have to adapt this to a Docker "stack" file; 
the format is mostly compatible with docker-compose and may eventually 
supplant it. You have to pay attention to persistent volumes however, 
which you don't want spread across nodes mindlessly. Leads: consider 
using a distributed filesystem for Orthanc storage, or a key/value 
store with associated plugin, or even a Postgres database with 
"EnableStorage": "true". For the index, only use one replica and 
consider things like CloudStor or other solutions that will ensure the 
data is always where the container is running (or accessible from 
there). PG also has built-in replication capabilities to explore. 

I suggest starting small, though scaling does get interesting ;). 
Stateless services are usually easy to work with, especially with a 
solution like Swarm or K8s, but state is often a little bit of a 
challenge. 
