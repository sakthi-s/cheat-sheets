# Caffe Tutorial

## Data Pre-processing
The conventional blob dimensions for batches of image data are number N x channel K x height H x width W. 
Blob memory is row-major in layout, so the last / rightmost dimension changes fastest. 
For example, in a 4D blob, the value at index (n, k, h, w) is physically located at index ((n * K + k) * H + h) * W + w.

[`caffe/python/caffe/io.py`](https://github.com/BVLC/caffe/blob/master/python/caffe/io.py): Can refer to this file to understand more about available funcitons for input/output of data (ex. array_to_datum, array_to_blobproto, Transformer etc.)

### LMDB

## Training


## Issues
### SolverState snapshot fails when .caffemodel is big
Store the snapshot in HDF5 format.
```
# Add this to the solver.prototxt file
snapshot_format: HDF5
```
