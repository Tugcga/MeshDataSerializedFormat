# Mesh Data Serialized Format

This format developed specialy for saving and reading data of 3d-meshes and other similar objects.

## Format description

* File starts from 4 bytes: 2-byte format identifier (actual 31250 number) and 2-byte version identifier
* Next file contains ordered sequence of so-called "frame containers" (or simply containers). Each container contains data for the given set of frames. 
 * Container starts from header of 18 bytes: 2 bytes for the section key, 4 bytes for the start frame, 4 bytes for the end frame, 8 bytes for the size of complete container without header. So, you can jump over this size and start read the header of the next container.
 * Next 6 bytes contains zero-section of the data: 2 bytes for the section key, 2 bytes for the length of data signature string, 2 bytes for the number of properties of this data section.
 * Next is header-section for the data. The size of this section is 2 + (signature length) + 10 * (properties count). This section contains 2-bytes header key, the string with actual data signature, the sequaence of pairs: 2-bytes parameter key and 8-bytes parameter value.
 * Parameters always contain the following data: the count of data records, the size of one data record, total size of all data recotds, identifier of the data type, identifier of the data context, the size of data name string.
 * If the name is not empty, then the next bytes store this name.
 * Next container contains the sequence of data records. All these should be in the same format, and can contains int, float and bool values.

## How to use

1. Create the main object: 
```
doc = MeshSerializer()
```
2. Add frame container to it with specific frame:
```
container = doc.create_container(1)
```
3. Add data to the container. This data should be array of tuples. Each tuple sohuld be the same size and contains elements of the same type. You can use elements of different type in one tuple, but on different tuples in one place should be elements of the same type:
```
container.add_data([(0, 0), (1, 1), (2, 2)])
```
4. Add anothe frame container:
```
next_container = doc.create_container(2)
```
5. Add data to it:
```
next_container.add_data([(3, 3, 3), (4, 4, 4), (5, 5, 5)])
```
6. Finally save data to the disk:
```
doc.save_to_file(file_path)
```

## Using data type and context

When adding data toe the container you can specify the type of this data and it context. By default type is KEY_TYPE_UNKNOWN, but it is convinent to set KEY_TYPE_POSITION when data contains some positions of object, or KEY_TYPE_VERTEX_NORMAL if this data contains coordinates of normals and so on. But the data type does not describe actual data. For example: different parts of object has normals - vertices, polygons, polygon corners. The context key used to distinguish these cases. So, for adding positions os the mesh vertices tou should use:
```
container.add_data(data_array, data_context=KEY_CONTEXT_VERTEX, data_type=KEY_TYPE_POSITION)
```
If the data contains position of the object, then you shoud use:
```
container.add_data(data_array, data_context=KEY_CONTEXT_OBJECT, data_type=KEY_TYPE_POSITION)
```

To get data of some type from container you can call:
```
container.get_data_by_type(type)
```
This command return the array of tuples for the data of given type in container. There are other methods to obtain data for the given type and context.