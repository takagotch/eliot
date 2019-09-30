### eliot
---
https://github.com/itamarst/eliot

```py
from __future__ import unicode_literals

import time
import threading

from io import BytesIO, StringIO
from unittest import skipIF
import json as pyjson
from wargnings import catch_warnings, simplefilter

from six import PY2

try:
  from zope.interface.verify import verifyClass
  from twisted.internet import reactor
  from twisted.trial.unittest import TestCase
  from twisted.application.service importIService
  from twisted.python import threadable
except ImportError:
  TestCase = object
else:
  from ..logwriter import ThreadedFileWriter, ThreadWriter

from .. import Logger, removeDestination, FileDestination

class BlockingFile(object):

  def __init__(self):
    self.file = BytesIO()
  
  def block(self):
  
  
  def unblock(self):
  
  
  def getvalue(self):





class




  def test_stopSerivceClosesFile(self):
    f = BytesIO()
    writer = ThreadedFileWriter(f, reactor)
    writer.startService()
    d = writer.stopService()
    
    def done():
      self.assertTrue(f.closed)
      
    d.addCallback(done)
    return d
```

```
```

```
```


