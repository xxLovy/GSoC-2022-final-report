# GSoC Final Report

**Project name: Python Interface to HDF5 Asynchronous I/O**

**Organization: Center for Research in Open Source Software (CROSS)**

## ![cross-logo-wide](cross-logo-wide.png)

## Introduction

HDF5 is a well-known library for storing and accessing (known as “Input and Output” or I/O) data on high-performance computing systems. Recently, new technologies, such as asynchronous I/O and caching, have been developed to utilize fast memory and storage devices and to hide the I/O latency. Applications can take advantage of an asynchronous interface by scheduling I/O as early as possible and overlapping computation with I/O operations to improve overall performance. The existing HDF5 asynchronous I/O feature supports the C/C++ interface. This project involves the development and performance evaluation of a Python interface that would allow more Python-based scientific codes to use and benefit from the asynchronous I/O.

## What was done

I have implemented most of the async HDF5 functions of file, group, dataset and attribute and are encapsulated in h5py and tested.

Here is an example about how we use async h5py

```python
import h5py
#create an Eventset and a file
es_id=h5py.Eventset()
f = h5py.File("file.hdf5", "w", es=es_id)
#create a group under the file we create and then create a dataset under that
grp = f.create_group("grp", es=es_id)
dset = grp.create_dataset("dset", (10, 20), maxshape=(20, 60), es=es_id)

import numpy as np
data_read = np.empty((10, 20), dtype=int)
data_write = np.arange(200, dtype=int)

#now we can use write_direct
dset.write_direct(data_write.reshape(10, 20), es=es_id)
dset.read_direct(data_read.reshape(10, 20), es=es_id)

dset.resize((20, 50), es=es_id)
f.attrs["file"] = 1
grp.attrs["grp"] = 2
dset.attrs["dset"] = 3

# now we can verify the data we have
es_id.wait(1000000)
print("the attribute value of file: " + str(f.attrs["file"]))
print("the attribute value of group: " + str(grp.attrs["grp"]))
print("the attribute value of dataset: " + str(dset.attrs["dset"]))
f.attrs.modify("file", 4)
es_id.wait(1000000)
print("the attribute value of file(modified): " + str(f.attrs["file"]))
print("dset.shape(resized) = " + str(dset.shape))

f.es_id=None
f.close()
es_id.wait(1000000)
es_id.close()
```

## What I have gained

+ Improved my python development
+ Improved my problem-solving skills
+ Learned the actual development process
+ Learned the method of embedding C language library in python and the use of some glue language

## Team

- [Xuan Xu](https://github.com/xxLovy) - Mentee
- [Houjun Tang](https://github.com/houjun) - Mentor
- [Suren Byna](https://github.com/sbyna) - Mentor

## Link to work

- [Link to documentation](https://xxlovy.github.io/async-h5py-documentation/)
- [Link to full repository](https://github.com/hpc-io/h5py/tree/async)

## Future Work

1. There are a lot of bugs in my work that have not been fixed yet. I will try to fix these bugs as soon as I can.

   + The read operations of dataset and attribute cannot be performed asynchronously correctly

   + There is some problem with async file close function under some circumstances some error will occur.  So we cannot close HDF5 object async

   + Currently the async function only works with the first read/write operation. From the second read/write operation due to the H5Dget() called by h5py they are effectively synchronous.

2. There are many documents about h5py (including my async h5py documentation) in English, so if possible, I can translate it into Chinese and contribute to the open source community

## Thank you

The past few months have been no less than fulfilling! I had a lot of fun and learned a lot duration this time because of the endless support of my awesome mentors. Since I'm not a naive speaker, my mentors were very patient with me when communicating with my tutor. This journey is also a very great opportunity for me to improve my English. The guidance provided for setting up my development enviroment helping me solve complicated problems will surely help me in becoming a better developer.

