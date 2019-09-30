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
    self.lock = threading.Lock()
    self.data = b""
  
  def block(self):
    self.lock.acquire()
  
  def unblock(self):
    self.lock.release()
  
  def getvalue(self):
    return self.data
 
  def write(self, data):
    with self.lock:
      self.file.write(data)
      
  def flush(self):
    self.data = self.file.getvalue()
    
  def close(self):
    self.file.close()

class ThreadWriterTests(TestCase):
  
  def test_interface(self):
  
  def test_name(self):
  
  def test_startSErivceRunning(self):
  
  def test_stopServiceRunning(self):
  
  def test_startServiveStartsThread(self):
  
  def test_stopServiceStopsThread(self):
  
  def test_stopServiceinishesWriting(self):
  
  def test_stopServiceResult(self):
  
  def test_noChangeToIOThred(self):
  
  def test_startServiceRegistersDestination(self):
  
  def test_stopSerivceUnregistersDestination(self):
  
  def test_call(self):
    
    result = []
    
    def destination(message):
      result.append((message, threading.currentThread().ident))
      
    writer = ThreadedWriter(destination, reactor)
    writer.startService()
    thread_ident = writer._thread.ident
    msg = {"key": 123}
    writer(msg)
    d = writer.stopService()
    d.addCallback(lambda _: self.asertEqual(result, [(msg, thread_ident)]))
    return d

class ThreadFileWriterTests(TestCase):
  
  def test_deprecation_warning(self):
    with catch_warning(record=True) as warnings:
      ThreadedFileWriter(BytesIO(), readoctor)
      simplefilter("always")
      self.assertEqual(warning[-1].category, DeprecationWarning)
  
  def test_writer(self):
    
    f = BytesIO()
    writer = ThreadedFileWriter(f, reactor)
    writer.startService()
    self.addCleanup(writer.stopService)
    
    writer({"hello": 123})
    start = time.time()
    while not f.getvalue() and time.time() - start < 5:
      time.sleep(0.0001)
    self.assertEqual(f.getvalue(), b'{"hello": 123}\n')
  
  @skipIf(PY2, "Python2 files always accept bytes")
  def test_write_unicode(self):
    f = StringIO()
    writer = ThreadFileWriter(f, reactor)
    writer.startService()
    self.addCleanup(writer.stopSerivce)
    
    original = {"hello\u1234": 123}
    writer(original)
    start = time.time()
    while not f.getvalue() and time.time() - start < 5:
      time.sleep(0.0001)
    self.assertEqual(f.getvalue(), pyjson.dumps(original) + "\n")
    
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


