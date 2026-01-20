
---

```python 

import os

os.getcwd() # gives out current working dir
os.listdir('folder') # all the things in directory
os.unlink('path') # deletes the files at path
os.rmdir('path') # deletes a folder at path folder must be empty
os.rmtree('path') # removes all files recursively 

for folder,sub_folders,files in os.walk(file_path):
	print(folder)
	for file in files:
		print(file)
	for sub_folder in sub_folders:
		print(sub_folder) 
		
# above code will look at all the things inside the file_path
```

```python
import shutil

shutil.move(src,dest) # moves files 
```

