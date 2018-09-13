# COCO数据集的标注格式

COCO的 全称是Common Objects in COntext，是微软团队提供的一个可以用来进行图像识别的数据集。MS COCO数据集中的图像分为训练、验证和测试集。COCO通过在Flickr上搜索80个对象类别和各种场景类型来收集图像，其使用了亚马逊的Mechanical Turk（AMT）。

COCO通过大量使用Amazon Mechanical Turk来收集数据。COCO数据集现在有3种标注类型：object instances（目标实例）, object keypoints（目标上的关键点）, and image captions（看图说话），使用JSON文件存储。比如下面就是Gemfield下载的COCO 2017年训练集中的标注文件：

可以看到其中有上面所述的三种类型，每种类型又包含了训练和验证，所以共6个JSON文件。

##基本的JSON结构体类型
这3种类型共享下面所列的基本类型，包括info、image、license，而annotation类型则呈现出了多态：

{
"info": info,
"licenses": [license],
"images": [image],
"annotations": [annotation],
}

info{
"year": int,
"version": str,
"description": str,
"contributor": str,
"url": str,
"date_created": datetime,
}

license{
"id": int,
"name": str,
"url": str,
} 

image{
"id": int,
"width": int,
"height": int,
"file_name": str,
"license": int,
"flickr_url": str,
"coco_url": str,
"date_captured": datetime,
}
1，info类型，比如一个info类型的实例：

"info":{
"description":"This is stable 1.0 version of the 2014 MS COCO dataset.",
"url":"http:\/\/mscoco.org",
"version":"1.0","year":2014,
"contributor":"Microsoft COCO group",
"date_created":"2015-01-27 09:11:52.357475"
},
2，Images是包含多个image实例的数组，对于一个image类型的实例：

{
"license":3,
"file_name":"COCO_val2014_000000391895.jpg",
"coco_url":"http:\/\/mscoco.org\/images\/391895",
"height":360,"width":640,"date_captured":"2013-11-14 11:18:45",
"flickr_url":"http:\/\/farm9.staticflickr.com\/8186\/8119368305_4e622c8349_z.jpg",
"id":391895
},
3，licenses是包含多个license实例的数组，对于一个license类型的实例：

{
"url":"http:\/\/creativecommons.org\/licenses\/by-nc-sa\/2.0\/",
"id":1,
"name":"Attribution-NonCommercial-ShareAlike License"
},
##Object Instance 类型的标注格式
1，整体JSON文件格式

比如上图中的instances_train2017.json、instances_val2017.json这两个文件就是这种格式。

Object Instance这种格式的文件从头至尾按照顺序分为以下段落：

{
"info": info,
"licenses": [license],
"images": [image],
"annotations": [annotation],
"categories": [category]
}
是的，你打开这两个文件，虽然内容很多，但从文件开始到结尾按照顺序就是这5段。其中，info、licenses、images这三个结构体/类型 在上一节中已经说了，在不同的JSON文件中这三个类型是一样的，定义是共享的。不共享的是annotation和category这两种结构体，他们在不同类型的JSON文件中是不一样的。

images数组、annotations数组、categories数组的元素数量是相等的，等于图片的数量。

2，annotations字段

annotations字段是包含多个annotation实例的一个数组，annotation类型本身又包含了一系列的字段，如这个目标的category id和segmentation mask。segmentation格式取决于这个实例是一个单个的对象（即iscrowd=0，将使用polygons格式）还是一组对象（即iscrowd=1，将使用RLE格式）。如下所示：

annotation{
"id": int,
"image_id": int,
"category_id": int,
"segmentation": RLE or [polygon],
"area": float,
"bbox": [x,y,width,height],
"iscrowd": 0 or 1,
}
注意，单个的对象（iscrowd=0)可能需要多个polygon来表示，比如这个对象在图像中被挡住了。而iscrowd=1时（将标注一组对象，比如一群人）的segmentation使用的就是RLE格式。

另外，每个对象（不管是iscrowd=0还是iscrowd=1）都会有一个矩形框bbox ，矩形框左上角的坐标和矩形框的长宽会以数组的形式提供，数组第一个元素就是左上角的横坐标值。

area是area of encoded masks。

最后，annotation结构中的categories字段存储的是当前对象所属的category的id，以及所属的supercategory的name。

下面是从instances_val2017.json文件中摘出的一个annotation的实例：

{
"segmentation": [[510.66,423.01,511.72,420.03,510.45,416.0,510.34,413.02,510.77,410.26,\
510.77,407.5,510.34,405.16,511.51,402.83,511.41,400.49,510.24,398.16,509.39,\
397.31,504.61,399.22,502.17,399.64,500.89,401.66,500.47,402.08,499.09,401.87,\
495.79,401.98,490.59,401.77,488.79,401.77,485.39,398.58,483.9,397.31,481.56,\
396.35,478.48,395.93,476.68,396.03,475.4,396.77,473.92,398.79,473.28,399.96,\
473.49,401.87,474.56,403.47,473.07,405.59,473.39,407.71,476.68,409.41,479.23,\
409.73,481.56,410.69,480.4,411.85,481.35,414.93,479.86,418.65,477.32,420.03,\
476.04,422.58,479.02,422.58,480.29,423.01,483.79,419.93,486.66,416.21,490.06,\
415.57,492.18,416.85,491.65,420.24,492.82,422.9,493.56,424.39,496.43,424.6,\
498.02,423.01,498.13,421.31,497.07,420.03,497.07,415.15,496.33,414.51,501.1,\
411.96,502.06,411.32,503.02,415.04,503.33,418.12,501.1,420.24,498.98,421.63,\
500.47,424.39,505.03,423.32,506.2,421.31,507.69,419.5,506.31,423.32,510.03,\
423.01,510.45,423.01]],
"area": 702.1057499999998,
"iscrowd": 0,
"image_id": 289343,
"bbox": [473.07,395.93,38.65,28.67],
"category_id": 18,
"id": 1768
},
3，categories字段

categories是一个包含多个category实例的数组，而category结构体描述如下：

{
"id": int,
"name": str,
"supercategory": str,
}
从instances_val2017.json文件中摘出的2个category实例如下所示：

{
"supercategory": "person",
"id": 1,
"name": "person"
},
{
"supercategory": "vehicle",
"id": 2,
"name": "bicycle"
},
