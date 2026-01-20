
---

Error handling means whenever error happens our code is compliant with error and will continue to work even if error happens

we use try catch and finally blocks to do so

```python
try:
# code block to be attepted which can give us an error
except <optional type of error>:
# code block will gets executed if error is occured in try
else: 
# runs if no error occurs
finally:
# runs every time regardless of error or not
```

We can chain multiple excepts for handling differnet types of errors.

### Unit testing

There are two library which help us here 

1. pylint: Looks at code and reports back possible issues
2. unittest: build in library to allow to test program and check if getting desired outputs.

Python has set of styling rules called PEP8

```bash 
pip install pylint
```

it is runed as the command so you can run 

```sh
pylint sample.py 
```

Since unittest is inbuild. We create a testing class by importing from unitTest library's TestCase module and define a test as a function. These functions should be methods and have self as variable. After this one can import thier code and run it. after this assert is called.

```python
import unittest
from cap import cap_text

class TestCap(unittest.TestCase):
	def test_one_word(self):
		text = 'python'
		result = cap_text(text)
		self.assertEqual(result,'Python')
		
		
# to run the test 

if __name__=='__main__':
	unittest.main()
```



