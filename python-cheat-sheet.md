# Python Cheat Sheet
References: 
  - [Python Crash Course](https://ehmatthes.github.io/pcc/cheatsheets/README.html)
  - [Numpy Cheat Seet](https://s3.amazonaws.com/assets.datacamp.com/blog_assets/Numpy_Python_Cheat_Sheet.pdf)

## Numpy
### Reshape
  1. __np.expand_dims(a, axis)__ : Insert a new axis that will appear at the axis position in the expanded array shape.
  2. __np.rollaxis(a, axis, start=0)__: Roll the specified axis backwards, until it lies in a given position. (Useful in BGR --> RGB sort of conversions)
  3. __numpy.squeeze(a, axis=None)__: Remove single-dimensional entries from the shape of an array.
  4. 
