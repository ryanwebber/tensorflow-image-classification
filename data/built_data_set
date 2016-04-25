#!/usr/bin/env python

import os, sys, subprocess
import tempfile
import shutil
import gzip, zlib
import uuid, math
import json
from wand.image import Image, Color
from pprint import pprint

# Extend the wand image class to get pixel 
class DataImage(Image):
    @property
    def pixels(self):
        pixels = []
        self.depth = 8
        blob = self.make_blob(format='RGB')
        for cursor in range(0, self.width * self.height * 3, 3):
            pixels.append(blob[cursor])
        return bytearray(pixels)

    def convertForModel(self, imgWidth=-1, imgHeight=-1, extension="jpg"):
        if imgWidth == -1: imgWidth = self.width
        if imgHeight == -1: imgHeight = self.height
        self.auto_orient()
        ratio = float(self.width)/self.height
        target_ratio= float(imgWidth)/imgHeight

        old_width = self.width
        old_height = self.height
        if ratio > target_ratio:
            width = imgWidth
            height = int(old_height*imgWidth/old_width)
        else:
            width = int(old_width*imgHeight/old_height)
            height = imgHeight

        width = width+1 if (imgWidth-width)%2 == 1 else width
        height = height+1 if (imgHeight-height)%2 == 1 else height

        self.evaluate(operator='median', value=self.quantum_range*0.3)
        self.equalize()
        self.resize(width, height)
        self.format = extension
        self.strip()
        self.type = "grayscale"
        self.gravity = "center"
        self.border(Color("black"), (imgWidth-width)/2, (imgHeight-height)/2)

def update_progress(progress):
    sys.stdout.write('\r[{0}{1}] {2}%'.format('#'*(progress/5), ' '* int(math.ceil((100.0-progress)/5)), progress))
    sys.stdout.flush()

def bytesFromInt(v):
    return bytearray([v >> i & 0xff for i in (24,16,8,0)])

def write(bytes, file):
    file.write(bytes)

def getDirs(base):
    labels = []
    count = 0

    for d in base:
        directory = d["directory"]
        label = d["label"]
        files = []
        label_count = 0
        for i in os.listdir(directory):
            if i.endswith(".jpg") or i.endswith(".png") or i.endswith(".jpeg"):
                files.append("{}/{}".format(directory, i))
                count += 1
                label_count += 1

        labels.append({
            "count": label_count,
            "files": files,
            "label": label
        })

    return {
        "count": count,
        "labels": labels
    }

def buildTrainingDataSet(options):

    print "Packaging Images and Labels:"

    DUMP=options["dump_dir"]
    IMG_WIDTH=options["width"]
    IMG_HEIGHT=options["height"]
    IMG_EXTENSION=options["image_format"]
    DIRS=options["dirs"]
    IMAGE_OUTPUT=options["image_dest"]
    LABEL_OUTPUT=options["label_dest"]

    MAGIC_IMAGE=2051
    MAGIC_LABELS=2049

    if os.path.isdir(DUMP): shutil.rmtree(DUMP)
    os.makedirs(DUMP)

    if not os.path.exists(os.path.dirname(IMAGE_OUTPUT)):
        os.makedirs(os.path.dirname(IMAGE_OUTPUT))

    if not os.path.exists(os.path.dirname(LABEL_OUTPUT)):
        os.makedirs(os.path.dirname(LABEL_OUTPUT))

    dirs = getDirs(DIRS)

    image_file = open(IMAGE_OUTPUT, 'w')
    label_file = open(LABEL_OUTPUT, 'w')

    write(bytesFromInt(MAGIC_IMAGE), image_file)
    write(bytesFromInt(dirs["count"]), image_file)
    write(bytesFromInt(IMG_WIDTH), image_file)
    write(bytesFromInt(IMG_HEIGHT), image_file)

    write(bytesFromInt(MAGIC_LABELS), label_file)
    write(bytesFromInt(dirs["count"]), label_file)

    progress = 0
    for label in dirs["labels"]:
        for filename in label["files"]:
            uname = uuid.uuid4()
            with DataImage(filename=filename) as img:
                img.convertForModel(imgWidth=IMG_WIDTH, imgHeight=IMG_HEIGHT, extension=IMG_EXTENSION)
                img.save(filename="{}/{}.{}".format(DUMP, uname, IMG_EXTENSION))

                progress += 1
                update_progress(int(100 * progress / dirs["count"]))
                pixels = img.pixels

            write(pixels, image_file)
        write(bytearray([label["label"]] * label["count"]), label_file)

    image_file.close() 
    label_file.close() 

    print

    image_size = os.path.getsize(IMAGE_OUTPUT)
    image_expected_size = dirs["count"] * (IMG_WIDTH * IMG_HEIGHT) + 4*4
    label_size = os.path.getsize(LABEL_OUTPUT)
    label_expected_size = dirs["count"] + 2*4

    if image_size != image_expected_size:
        print "Expected exported image file size to be {} bytes but found {} bytes".format(image_expected_size, image_size)
    elif label_size != label_expected_size: 
        print "Expected exported label file size to be {} bytes but found {} bytes".format(label_expected_size, label_size)
    else:
        subprocess.check_call(["gzip", IMAGE_OUTPUT])
        subprocess.check_call(["gzip", LABEL_OUTPUT])
        print "Successfully built images and labels."


def main():
    with open(sys.argv[1]) as data_file:    
        data = json.load(data_file)

    buildTrainingDataSet(options=data)

if __name__ == '__main__':
    main()

