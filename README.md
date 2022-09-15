# GSoC Final Report

**Project name: Python Interface to HDF5 Asynchronous I/O**

**Organization: Center for Research in Open Source Software (CROSS)**

## ![cross-logo-wide](cross-logo-wide.png)

## Introduction

HDF5 is a well-known library for storing and accessing (known as “Input and Output” or I/O) data on high-performance computing systems. Recently, new technologies, such as asynchronous I/O and caching, have been developed to utilize fast memory and storage devices and to hide the I/O latency. Applications can take advantage of an asynchronous interface by scheduling I/O as early as possible and overlapping computation with I/O operations to improve overall performance. The existing HDF5 asynchronous I/O feature supports the C/C++ interface. This project involves the development and performance evaluation of a Python interface that would allow more Python-based scientific codes to use and benefit from the asynchronous I/O.

## What was done

1. Implemented most of the async HDF5 functions of file, group, dataset and attribute and are encapsulated in h5py and tested.

Here is a simple example of how to use the async function to operate a dataset. If we don't delete the debug message, we can also see what HDF5 async function has been used.

If we want to use the asynchronous functions, an eventset is necessary. And then we can create a file object asynchronously

```python
>>> import h5py
>>> es_id=h5py.Eventset()
>>> f = h5py.File("file.hdf5", "w", es=es_id)
Using h5py with async HDF5 to create a file
```

Then we can use the async function to create a group under file and a dataset under group

```python
>>> grp = f.create_group("grp", es=es_id)
Using H5Gcreate_async
>>> dset = grp.create_dataset("dset", (10, 20), maxshape=(20, 60), es=es_id)
Using H5Dcreate_async
```

now we can use the write_direct function to write data into the dataset we have created

```python
>>> dset.write_direct(data_write.reshape(10, 20), es=es_id)
Using H5Dget_space_async
Using H5Dwrite_async
```

and if we want to change its size asynchronously, we can use the resize function with the eventset parameter.

```python
>>> dset.resize((20, 50), es=es_id)
Using H5Dget_space_async
Using H5Dget_space_async
>>> dset.shape
(20, 50)
```

After we finish all the operation, we can use the close() function to close all the HDF5 object

```python
>>> f.es_id=None
>>> f.close()
>>> es_id.wait(1000000)
>>> es_id.close()
```

2. Deveopped the test code of async h5py

   Here is the test code for dataset operation

   ```python
   @ut.skipIf(version.hdf5_version_tuple < (1, 13, 0), 'Requires HDF5 1.13.0 or later')
   class TestAsync(BaseDataset):
       from h5py import Eventset
       def setUp(self):
           self.es_id = Eventset()
           self.es_id.wait(-1)
           assert self.es_id.num_in_progress==0
           assert self.es_id.op_failed==False
           self.f = File(self.mktemp(), 'w', es_id=self.es_id)
   
       def tearDown(self):
           if self.f:
               self.f.close()
               self.es_id.wait(-1)
               assert self.es_id.num_in_progress==0
               assert self.es_id.op_failed==False
           if self.es_id:
               self.es_id.close()
       
       def test_create_dataset(self):
           dset = self.f.create_dataset_async("dset", (20, 30), es_id=self.es_id)
   
       def test_write_direct(self):
           data_write = np.arange(100, dtype=int)
           dset = self.f.create_dataset_async("dset", (10, 10), es_id=self.es_id)
           dset.write_direct_async(data_write, es_id=self.es_id)
           out = dset[...]
           self.assertArrayEqual(out, data_write)
   
       def test_data_change(self):
           es_id0 = Eventset()
           es_id1 = Eventset()
           data0_write = np.arange(20 * 30, dtype=int)
           data1_write = np.arange(20 * 30, dtype=int)
           data1_write *= 2
   
           data0_read = np.empty(20 * 30, dtype=int)
           data1_read = np.empty(20 * 30, dtype=int)
           dset0 = self.f.create_dataset_async("dset0", (20, 30), dtype=int, es_id=es_id0)
           dset1 = self.f.create_dataset_async("dset1", (20, 30), dtype=int, es_id=es_id1)
           # W0, R0, W1, R1, W1', W0', R0', R1'
           dset0.write_direct_async(data0_write, es_id=es_id0)
           dset0.read_direct_async(data0_read, es_id=es_id0)
           #Verify data
           self.ass
   ```

   

Tested the performance of async h5py
<img src="https://raw.githubusercontent.com/xxLovy/async-h5py-documentation/main/chart.png" alt="chart" style="zoom: 67%;" />

When the size of dataset comes to 2.8 GB We can see that the asynchronous function is over 4000 times faster than the synchronous function! Imagine how much performance can be improved for larger data!

<img src="https://raw.githubusercontent.com/xxLovy/async-h5py-documentation/main/performance.png" alt="performance"  />

## What I have gained from this project

+ Improved my python development
+ Improved my problem-solving skills
+ Learned the actual process while developing a software
+ Learned the method of embedding C language library in python and the use of some glue language

## Team

- [Xuan Xu](https://github.com/xxLovy) - Mentee
- [Houjun Tang](https://github.com/houjun) - Mentor
- [Suren Byna](https://github.com/sbyna) - Mentor

## Link to work

- [Link to documentation](https://xxlovy.github.io/async-h5py-documentation/)
- [Link to full repository](https://github.com/hpc-io/h5py/tree/async)

## Future Work

1. There are some bugs in my work that have not been fixed yet. I will try to fix these bugs as soon as I can.

   + The read operations of dataset and attribute cannot be performed asynchronously correctly

   + There is some problem with async file close function under some circumstances some error will occur.  So we cannot close HDF5 object async

   + Currently the async function only works with the first read/write operation. From the second read/write operation due to the H5Dget() called by h5py they are effectively synchronous.

2. There are many documents about h5py (including my async h5py documentation) in English, so if possible, I can translate it into Chinese and contribute to the open source community

## Thank you

The past few months have been no less than fulfilling! I had a lot of fun and learned a lot duration this time because of the endless support of my awesome mentors. Since I'm not a naive speaker, my mentors were very patient with me when communicating with my tutor. This journey is also a very great opportunity for me to improve my English. This journey will surely help me in becoming a better developer.
